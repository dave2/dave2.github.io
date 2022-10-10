---
title: "Never Complete Games: Hexagon"
date: 2022-10-10T13:17:56+13:00
draft: false
cover:
    image: "game.jpg"
    alt: "A hexagon tile video game screenshot showing low-polygon tiles, with some places labelled"
    caption: "Hexagon World October 2022"
---

## What's all this then?

From time to time I get this idea I can and should write video games.
Which usually doesn't end well, but hey, it's something which passes
the time.

This round I've decided to go with a classic procedural RPG using
hexagon tiles and a top-down perspective camera. I tried using an
othrographic camera and yet again I ran into issues with how it was
culling shapes close and far from the camera.

Perspective works just fine with a narrow field of view, and everything
is much easier to drive spatially with it. Orthographic camera just 
always ended in tears failing to understand how to move it well and
keep stuff in view.

Anyway, this one seems to be actually holding my attention, so at some
point I will release .. bwhahaha who am I kidding. This may never see
light of day, but we shall see.

At the moment it's only running on PC because Reasons. The main one is
that Godot 4.0 beta has broken Android support for certain GPUs in 
Android phones. It tries to set up fancy Variable Rate Shading which 
my potato of a phone does not like.

I'll post more screenshots when there's more game to show!