---
layout: post
title:  "GAN interpolations in Tensorflow.js: The Basics"
date:   2022-03-25 19:00:00 +0100
categories: icon-gan tfjs
---

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

So you've trained your model and now it's time to inspect the results. A typical method - or at least what I did a lot - is generating random images. But surely there are better options to explore your creation. One such way is interpolation. We can take two images and observe how the model behaves while transitioning between them.

<p align="center">
<img src="/images/1d-interpolation.png" alt="Image of interpolation on Icon GAN model">
</p>

I'm always getting caught up in off-by-one errors on these, so let's write it down... and hopefully I can refer to it later :)

If you just want to try it out, you can [skip to the bottom](#interpolation-codepen) or check out [Icon GAN](/icon-gan/).

## Background

[Generative adversarial networks](https://en.wikipedia.org/wiki/Generative_adversarial_network) (GAN) are a class of neural networks. One part generates images and another part differentiates between the generated (fake) images and the real images in the training set.

The generator is typically given a randomized input. The distribution of that input is called the **latent space**. Once the model is trained, the latent space becomes an embedding of the images. Images that look alike typically lie close together in the latent space. Depending on the training method, different directions in the latent space can also correspond to certain features - like glasses or beards in face images or generally shapes and colors. [StyleGAN 2](https://github.com/NVlabs/stylegan2) for example explicitly aims to improve the mapping from latent space vectors to the generated images.

In StyleGAN 2, the latent space consists of one `w`x`h` matrix and multiple vectors. These are given to different layers of the network.

<p align="center">
<img src="/images/StyleGAN2-network.svg" alt="Image of StyleGAN 2 generator network with inputs">
</p>

## Loading the model

To generate images, we need to load a model. In tfjs, we can do this using:
```js
const modelUrl = "https://raw.githubusercontent.com/Akatuoro/nn-models/master/icons-64-web/model.json";
const model = await tf.loadGraphModel(modelUrl);
```

The model has to be in the correct format. If you have a Tensorflow SavedModel or a Keras model, you can convert it using [tfjs-converter](https://github.com/tensorflow/tfjs/tree/master/tfjs-converter).

## Generating base images

We first need two images between which we interpolate. Or rather, we need their inputs in the latent space. Let's call them `A` and `B`. They could be preselected - e.g. like in [Icon GAN](/icon-gan/) using drag & drop. But for this example, we generate them randomly:

```js
const inputA = getRandomInput();
const inputB = getRandomInput();
```

So what is `getRandomInput`?

<br>

This specific model has one `64 x 64 x 1` matrix and five vectors of length `512` as input. When entering them as array, the matrix needs to be in the 3rd position. Since we will want to generate batches, we add an additional dimension at the beginning of each tensor.

```js
function getRandomInput() {
    return [
        tf.randomNormal([1, 512]),
        tf.randomNormal([1, 512]),
        tf.randomUniform([1, 64, 64, 1]),
        tf.randomNormal([1, 512]),
        tf.randomNormal([1, 512]),
        tf.randomNormal([1, 512])
    ];
}
```

Testing that image generation works, we generate and render the image A:
```js
const outputA = model.execute(inputA)
const imageA = toImg(outputA);
ctx.putImageData(imageA, 0, 0);
```

But our output is not yet an image - we need to clip any values that exceed the `[0, 1]` range, add an alpha channel and multiply by 255 to put it into an [ImageData](https://developer.mozilla.org/en-US/docs/Web/API/ImageData) object. Also, we transpose our output matrix, so that we can generate a single big image.

```js
function toImg(tensor) {
    const n = tensor.shape[0];

    // clip, alpha channel & multiply:
    let d = tf
        .concat([tensor.clipByValue(0, 1), tf.ones([n, 64, 64, 1])], 3)
        .mul(255);

    d = tf.transpose(d, [1, 0, 2, 3]);

    const im_data = new ImageData(new Uint8ClampedArray(d.dataSync()), 64 * n, 64);

    return im_data;
}
```

If you want multiple single images, you could simply iterate over the batch dimension:
```js
// clip, alpha channel & multiply...
tf.split(tensor, n).forEach(d => {
    const im_data = new ImageData(new Uint8ClampedArray(d.dataSync()), 64, 64);
});
```

## Interpolation

Now that we have two images and tested that basic image generation works, we can start with interpolating.
We take the inputs of our images and do a linear interpolation with $$n$$ inputs $$Z_t$$, including our base inputs:


$$Z_{t=0..n} = A + (B - A) * \frac{t}{n}$$


<div class="image-grid">
	<div>A</div>
	{% for i in (1..9) %}
		<div class="blue">0.{{i}}</div>
	{% endfor %}
	<div>B</div>
</div>
<br>

After calculating the inputs $$Z_t$$, we concatenate them on the first axis for batch processing.
As we have multiple input tensors, we need to perform the operations for each tensor:

```js
// precalculate B - A
const v = inputB.map((t, i) => t.sub(inputA[i]));

const n = 9;
const input = inputA.map((t, i) => {
	const combined = [];
	// calculate all Z_t for the current input tensor index
	for (let j = 0; j <= n; j++) {
		combined.push(t.add(v[i].mul(j / n)));
	}
	// concatenate Z_t for batch processing
	return tf.concat(combined);
});
```

Now simply execute the model and render the resulting image using our predefined method:
```js
const output = model.execute(input);
const imageData = toImg(output);
ctx.putImageData(imageData, 0, 0);
```


<a id="interpolation-codepen"></a>Check out the resulting CodePen:

<p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="YzYXJvR" data-user="akatuoro" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/akatuoro/pen/YzYXJvR">
  GAN Interpolation</a> by Akatuoro (<a href="https://codepen.io/akatuoro">@akatuoro</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

<br>
<br>

> What model are you going to interpolate on?


<style>
	.image-grid {
		display: grid;
		grid-template-columns: repeat(11, 1fr);
		width: 100%;
	}
	.image-grid div {
		aspect-ratio: 1/1;
		display: grid;
		place-items: center;
		text-align: center;
		font-size: 1vh;

		border: solid;
		border-width: 1px;
	}

	.blue {
		background-color: lightblue;
	}
	.green {
		background-color: lightgreen;
	}
	.teal {
		background-color: teal;
	}
</style>
