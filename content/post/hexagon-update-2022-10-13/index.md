---
title: "Hexagon Game Update: The Past"
date: 2022-10-13T12:29:46+13:00
draft: false
tags: ["hexagon_game"]
---

I realised that since I started writing this silly wee game I actually
haven't written down any of the initial thoughts about how this was
goign to work, and since I've only just resurrected writing a blog
after I'd already started writing the game there's no history here
either.

Right, let's see about writing some of those thoughts down.

## Hexagon tiles

I'm implementing a classical RPG and hexagon tiles are pretty common
for maps in that context. Square maps are also an option, but I feel
like they lend themselves better to worlds where you and the party
are in fine-definition worlds, where hexagons are better at
coarse-definition worlds.

So for example, in a fine-definition world a city or a village 
is a large number of tiles, you have the city's streets to actually
navigate around etc. In a coarse-definition world the city is one
tile, possibly with some visual sugar around that, but internal
city views are a bit more menu-like and less navigated using tiles.

This does also make some types of generation of the world a bit 
easier, because we really only looking at high-level detail and 
that has a lot fewer elements to try to model. Cities are just a
single world position and not a complex map of tiles within the world.

That said, there's nothing stopping me from implementing a nested
world approach - where entering a city essentially creates a new
hexagon map at a finer level of detail - so we're not really closing
ourselves off on doing that level of detail. It would always be a
scene switch to do it, however.

Since I'm pretty rubbish at art, I'm leaning into [Kenney's Assets](https://kenney.nl/assets/)
which include a [Hexagon Tile Kit](https://kenney.nl/assets/hexagon-kit).
So we have enough art to do world generation and some idea what the
tile set *should* be able to do, if I can write the supporting code.
If you have an interest in game dev but not much ability in art, then
you really should look at Kenney's work as it's high quality and 
available under very generous license terms.

The tile set is flexible enough to be coarse or fine-definition, for
the most part. There's some gaps here and there but for this game in
this state the gaps are just small workarounds.

## Hexagon math

Having a set of actual 3D tiles is all very well but you have to 
position them. The positioning is
some very well documented math, the resource I used is [Red Blob Games Hexagonal Grids](https://www.redblobgames.com/grids/hexagons/)
which is extremely clearly written and easily implemented in any language
you like. I'm not going to explain any of the math here!

The discussion on co-ordinate systems in Red Blob Games's guide is
very important to read before going to far into the development. 
I settled on the Axial coordinate system described in the guide.

I didn't consider the offset coordinate systems desscribed there for
long because the math is annoying for neighbours, directions, distances
etc. They were easiest to deal with by converting offset to cube or 
axial coordinates and if that's the case I might as well use cube or
axial natively.

Comparing axial to cube, the big advantage of axial is that the
coordinates are always valid, as they are in offset systems. That is,
a random value for each coordinate dimension is always okay, but
with cube coordinates valid coordinates are only where they sum to zero.

Since the vector math for axial is nothing unusual, we can use
Godot's `Vector2i` type for an axial coordinate. I'm using `Vector3i`
for cube coordinates and there's conversion functions between them
documented in the Red Blob Games guide.

The conversion from axial to world (ie, where we place a hex on a flat
plane) is also in the guide and is easy to implement. That makes
positioning in the game's 3D world coordinate space quick to do.

## Camera

Hexagon tile games are commonly seen with both perspective cameras and
orthographic cameras. The difference between the two is something you
are best just firing at a search engine, but the tldr is perspective
is what we see normally (far away objects are smaller), and ortographic
keeps everything the same size regardless of distance. 

The latter camera has the same look as games that use a 2D engine with
tiles that fake 3Dness, the classic "isometric" look. That particular
look is historically very common for an RPG or tile base game. 

Although it's tempting to ram home the somewhat retro ideas in the
game by using an isometric (or approximately isometric) view, I've 
ended up using a perspective camera as I have had endless
problems with Godot's orthographic camera, ranging from clipping 
models to annoying lighting behaviour.

I know that there's a big chunk of this which is just my understanding
of how to correctly configure and drive the camera, but perspective
"Just Works" and it's perfectly valid for a tile RPG (or other hexagonal
grid games, like Civilization), so I'm taking the path of least resistance.

## Hexagon storage

The Red Blob Games guide to Hexagon grids discusses some issues around
storage, but this is actually a pretty easy decision. 

Since Godot provides `Dictionary` type, and we can use axial coordinates
(really `Vector2i`) as keys, it is the fastest and easiest way to store the map data. It 
will tolerate any shape and holes or partially generated state. It has
fast access since it uses a hash to return the content of a specific
key, it enforces unique keys, and walking the whole map is no harder
than any other storage approach. It stores about as efficiently as 
is possible.

As I'm using axial coordinates, there are helper functions to do
things like return the neighours of a coordinate, or the tile in a
specific direction and distance, and so forth. We don't rely
on the data storage structure to provide those constructs, so it
doesn't matter that the storage has no concept of these.

However, there's one issue with storage of a world..

## Infinite vs Finite

I am a fan of infinite prodcedural worlds. But for this game, I 
don't think we need a world we move infinitely in any direction, as
we can provide new generated challenges within a finite world.

Not allowing the world to be infinite has several benefits. For one, world
generation is simplier if the world is finite, as we can simply
iterate over the world multiple times to add features to it. With
infinite worlds, each area or chunk (generally) needs to be generated
independently and unexpectedly.

The other comes with approaching difficulty for the player. Rather than
having the difficulty determined by some value proportionate to 
experience or gear, I would like difficulty to be determined by the
location the player is at. 

That is, difficulty is a function of the distance from a "safe" point
to the edge of the world. For easy implementation, we'll set the safe
point to the literal center of the world (`Vector2i(0,0)`). This means
when the player travels far away from the center the challenges are
more difficult, but they can easily control what challenges they want
to manage just by moving around. 

With an infinite world, you can do this, but I feel like it takes
away from one of the values of an infinite world (that is, you can just "explore"
and you aren't bound to the same place, but you can't if the difficulty
gets worse the further you are out).

At the same time, I can hand-wave away the storage of the world needing
chunks because we don't need chunks straight away for a finite world.
Let's just make the whole world sit in RAM. What could go wrong?

## Wraping up

Thus, the game being written is:
 
 - A finite world, so we can cut corners on world gen
 - Difficulty scales by distance from the centre, in which case we 
   can just use a "circular" (actually big hexagon) shape for the world
 - Perspective because Orthogonal camera disagrees with me
 - Doesn't yet try to do anything to load/save/cache chunks but just
   has everything in memory at once

There's other decisions yet to come, but this'll do for now.