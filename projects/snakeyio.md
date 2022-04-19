---
layout: page
title: Snakey.io
permalink: /snakey.io/
---

A classic snake game with multiplayer using WebRTC.

<a href="https://snakey.io" target="_blank">**Check it out at snakey.io.**</a>

It was inspired by [Proxx](https://github.com/GoogleChromeLabs/proxx) as a classic game built with modern web technologies. Initially designed to be minimal in size with singleplayer, this has extended when bringing in a multiplayer that combines the discrete step nature of the snake game with continous animation and latency compensation.


Snake has always been a fond game of mine since the days of feature phones. My first implementation of Snake (in Java) was during a winter vacation in 2008. It was later broken due to a lack of version control - git was not yet as popular. Since then, I've implemented simple versions multiple times with [snakey.io](https://snakey.io) being the version I can now enjoy playing anywhere, anytime.


![Two snakes moving towards an apple in the center](/images/snake-banner.png)

### Rendering

The game uses spritesheets generated when loading the game. This ensures low download sizes while generating fitting sprites for any screen resolution.

### Controls

- key input
- swipe acceleration based controls when using a touch device

### Multiplayer

Currently multiplayer is supported by exchanging WebRTC ICE candidates ([Signaling](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Connectivity#signaling)) in a lobby managed with [socket.io](https://socket.io/).

Once the game starts, state and actions are sent over WebRTC with one player acting as the host. The game state is simulated / predicted on client side based on the already known player actions - or lack thereof. If the client later gets updated information from the host, these changes are rolled back and the new changes are applied.
