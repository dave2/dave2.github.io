---
title: "Hexagon Update: 2 Dec 2022"
date: 2022-12-02T19:24:00+13:00
draft: false
---

Ah it's been a while, what's been going on?

## Android using OpenGL3 compat

So as of Godot 4.0 beta 7, you can re-enable opengl3 as a rendering engine
for Godot builds. Great! This is actually the first time I think I've
had the game running on my phone at all.

The good news is it looks about what I'd expect, except buttons are a 
little on the small side. But it's very very slow (esp refreshing the 
display) so this seems to be a mix of opengl3 being very rough at this
point in the Godot 4 cycle and simply the fact my phone ain't my PC
when it comes to the lifting.

The opengl3 engine isn't really expected to be any good in 4.0 and it's
not expected to be properly good until sometime later in the 4.x cycle
so this isn't unexpected at all. 

The other apsect which didn't work at all is GUI control input from touch.
I have "Emulate mouse from touch" enabled which should be all that's 
required but either there's some other major problem stemming from the
not fully cooked opengl3 renderer, or I have a real problem with
input in Android.

Anyway, hopefully the last remaining Vulkan rendering bug preventing the
game running at all on Vulkan on mobile will be fixed soon, and then 
we shall see what the real causes are!

## Dialog and Quests

Both of these systems are ones which I am not particularly familar with 
in how to approach them. However, the basic dialog frame with player
input works. It doesn't have next-page buttons like it should, but 
that's not too difficult to work on.

At the same time, there is rudimentary support for recognising that places
in the world can be entered and therefore ask about this. This needs
a better indication, as we shouldn't use the classic quest exclamation 
mark to mean "enterable", like shops are not quest points right?

The quest plan itself is roughly bullt in terms of what classes will do
what. There's a ``QuestPlan`` which contains the whole set of steps for
this quest, and where we are up to, and it holds at start point and
rewards and description for the overall quest. This also stashes an
array of ``QuestObjective`` and each of these are our requirements to
meet. Go talk to people, get objects, kill stuff.. those are steps.

The data model looks okay, but it needs a lot of filling out of the 
actual quest generation and objective handling. But baby steps. These
are the third and second most difficult parts of the game I have to
implement. (The most difficult is combat!)

## GUI work

I've also been fixing up the UI here and there, and trying to make it
look a lot more game-like. I'm still using the main Godot GUI components
and it turns out that I can theme them much more than I thought was
actually realistic to do so. Now I have much more richly-themeed 
components I can use instead of layering a bunch of controls and 2D
elements to fake it.

## Wrapping up

Progress is very slow (slower than it's ever been), but I'm chipping
away at the code as best I can, while the beta releases of Godot truck
on. Apart from some colour lookup issues, and ``FileAccess`` API rework
that landed in beta 5 and beta 3 respectively, it's been remarkably
smooth sailing in the beta releases so far, so I'm pretty happy about
how things have been going.

Cherry-O!