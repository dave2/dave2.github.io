---
title: "Hexagon Update: 3 Nov 2022"
date: 2022-11-03T20:38:00+13:00
draft: false
---

This will be a quick update only..

I've started working on allowing a placeholder player to be move around
the world. Actually, this wasn't too hard, it even does a nice tween that
deals with distance. I don't actually need to do that (I expect players
to move hex by hex) but it's nice to know the code works.

By "wasn't too hard", it did open up the can of worms I've been waiting
to have pounce on me.

The first thing was the default node for just holding UI elements
will block all mouse interaction below it in the scene. This is not the 
first time I've scratched my head for a good long while wondering why no
mouse events seem to reach the view of the world. I am not sure why 
essentially a holder for UI nodes defaults to blocking mouse events.
It can be disabled, but by default if you just create this simple 
bag of holding UI, it blocks mouse events.

Anyway, once that was out of the way (literally) mouse events were cleanly
getting down through to the world map and useful events coming from it.
The easy way to get an event for clicking or tapping an object is to 
make it "pickable", which means throwing a collider on it and then 
ensuring "pickable" is ticked. Easy! In fact Godot then takes care of a
ton of problems for you, just connect the signal out of any tile to a
common handler, and it takes care of all the conversion from 
viewport-camera coordinates to "real" world ones and you're off.

Okay, so there's the can of worms. Every tile would need a collider to do
it this way. Or, maybe only the active selectable ones. Maybe the 
collider is built at runtime, or on importing models, .. or it's baked
into a nice prefab that could hold a bunch of other things. The options are
a bit far and wide. That's before you go down the path of just raycasting
yourself out the camera on a low-level mouse click. 

I'm loathe to rebuild all the tiles again shifting from the ready-to-go 
GLTF that Godot handles fairly well. I've already walked every single one
externalising materials so I can apply my own common set to tiles. And
I would have to do something else to use more raw mesh formats and 
override the materials.

Building the collider on import is what I've done in other projects before.
It's not too hard to programatically create one as tile is being loaded.
But it feels like ugly code and another loading step, is marginally harder
to position correctly, but mostly feels "anti-Godot" somehow? It has a 
scene system, so why add complexity fiddling with models in code.

There's also the point that, as I said above, if I'm moving hex by hex
I could actually just add colliders at runtime when we want to move, and
add them only to the valid tiles. But that's also got problems, the 
relationship between the collider at the hex is either disconnected (and
we need to calculate the placement in the world separately), or it's a
child of the tile, but then we're tinkering with the tile's children 
(which means we have to be careful to clean up).

And sitting in the back of my head is that *if* I were to rebuild the 
tiles I could also add some other features to them. If I wanted to have
smoke from a chimney, or moving water wheels, then this is all easier
to do with taking the meshes more directly into a prefab and include all
the other elements (like animation nodes) in there.

In other words, rebuilding them is probably going to happen eventually
and if I do half that job now, then adding animations and particles to
them is "fairly" easy.

In the meantime, the game has a proper pause menu and saving just needs
to be wired to the save button. So it's not a lot of progress, but there
has been some!
