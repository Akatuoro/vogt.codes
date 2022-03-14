---
layout: page
title: Icon GAN
permalink: /icon-gan/
---

A generative adversarial network (GAN) for interactively generating favicons.

**<a href="https://akatuoro.github.io/icon-gan" target="_blank">Check it out at akatuoro.github.io/icon-gan.</a>**

Star it on <a href="https://github.com/Akatuoro/icon-gan" target="_blank">github.com/Akatuoro/icon-gan.</a>


Starting out as a class end project with a conventional GAN and a Wasserstein GAN, this project has evolved into an interactive interface to generate icons and explore their variations in the latent space. Using the different input layers, parts of the style like form and color can be directly influenced. Favored images can be set aside and later used for generating 2- or 3-way interpolations.

<p align="center">
<img src="/images/icon-interpolation-transparent.png" alt="2D interpolation between 3 icons">
</p>

### Models

The [current model](https://github.com/Akatuoro/nn-models/tree/master/icons-64-web) is based on [StyleGAN 2](https://github.com/NVlabs/stylegan2), using a Tensorflow 2 implementation and converting the result into a Tensorflow.js model.
