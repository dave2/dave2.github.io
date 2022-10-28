---
title: "Hexagon Update: Roads .. again?"
date: 2022-10-28T15:23:10+13:00
draft: true
---

This will be a short update.

Roads are by far and away the most expensive part of world generation.
Creating biomes, setting elevation, and decorating the terrain with
forests is all fairly quick. We largely only going over the tiles
once. In the case of elevation and forests we even do both of those
in the same pass.

But roads are annoying. They are expensive to calculate for a single
path and a nice way of 
Elevation and biome assignment are fast as we mostly only pass over
all the tiles once.
