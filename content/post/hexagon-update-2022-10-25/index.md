---
title: "Hexagon Update: 25 Oct 2022"
date: 2022-10-25T09:25:22+13:00
draft: false
---

Lots since the last update, lets dive into it.

## Refactoring Outcome

Refactoring all the road generation to use tile variants instead of
laying a separate prefab on top was a Huge Success and was definitely
the right approach.

Baking the roads into the tile set took a while but it resulted in
a much nicer implementation internally, as I took the chance to also
drop object pooling.

An optimization I had done very early on was to push 3D tiles into a
pool of pre-instantiated objects and pull them into the world as
needed. This is fairly useful in most game engines, but actually in
Godot it turns out it has no useful performance gain at all.

In Godot, the cost of `.instantiate()` is basically nothing and you
get a much cleaner code path without involving your own object pool.
You can just create objects and then `.queue_free()` them when done.
The name of that function hints at what is going on under the hood,
and something I hadn't really noticed until I did the refactor.

So long as you've already loaded the resource, the cost of spawning
many new ones at once is as low as all the work needed to maintain
the object pool. Having an object pool adds huge complexity (mostly
on the freeing side, you can't just free the object, you need to 
"hand back" the object to the pool). Given all the extra complexity
with no performance gain, it was a good thing to remove it entirely
and write more simple code.

## Stalling

Having refactored all code around the map view and in turn the way
we load tile type information, it was time to revisit world generation,
but specifically about both how slow it ran and the lack of feedback.

Providing feedback to the user is really important. If you're given
a blank screen with no visible activity, the instinct is the game 
has crashed or is otherwise misbehaving, and to kill it. It's
important while doing heavy work to show work being done, and ideally
some indication about the progress of that work.

Okay, no worries, Godot has progress bars and text I can display for
such things. But despite sending updates to the progress bars and
text, nothing rendered until it was all done.

The immediate approach I took was to slice off the world generation
into a thread and then just wait around for that to complete. This
sort of worked and then I changed things and it didn't, and because
I'm not the smartest person I couldn't remember what was actually
changed which made any difference.

Godot supports threads, and all the supporting elements needed to make
threads word (like mutexes and semaphores). Alas, adding all these
in various places didn't make the code behave any better. In one
part of the code, we'd test if the thread was running and it would
always return false, while the output showed the thread *clearly*
still running and spamming the log with messages.

It also didn't help that it started to look like sometimes if you
called functions they actually somehow caused all the visual rendering
to stall even if that function should have been in a thread.. right?

Two days later, I now have a better understanding of both Godot's 
main loop and most of the downsides of threads. Not a fun two days.

One thing I had forgotten is by and large games are single-threaded.
They might offload certain low level functions to a thread, like
audio playing or physics, but many games (and game engines) are 
single threaded because there are less complications about data
access and synchronization than a multi-threaded game or engine.

In Godot's case, rendering and `_ready()` in a script share a thread,
so if you block in `_ready()` then rendering ain't going to work.
But, I had "solved" this with making sure world gen was on a thread.

Well, sort of. Yes, some of it was on a thread, but not all of it,
and it was hard to identify what was actually executing where. A lot
of print-as-debug was added with every line including the output of
`OS.get_thread_caller_id` to show where this was really taking place.

What I found was it looks like singletons in Godot always execute on
the main thread, and the way to avoid this is ensure any functions
you call are on separate objects in your thread. Okay, refactor 
around that a bit (the calls into the world map singleton are fast,
so we'll ignore they run in the main thread, but it meant world gen
had to be separated from the singleton). 

Still, we had problems, threads were being reported to the main
code as finished when the logs were full of signs they were running.
Getting to the bottom of this I think came down to pure luck and
writing a separate set of tests in a different Godot project.

The test case I wrote looked something like the following. Note in 
this example we have a script on a node called `SimpleProgressBar`
which updates different elements, like upper and lower text messages
as well as the bar itself.

```python
var progress_bar : Node
var m : Mutex

func _ready():
    # we'll use a progress bar for visuals
    progress_bar = get_node("SimpleProgressBar")
    # we'll use this to protect our access
    m = Mutex.new()

    # start the long work thread
    var t = Thread.new()
    t.start(Callable(self,"_long_work_thread"))

    for i in range(0,100):
        # update the progress bar too!
        m.lock()
        progress_bar.set_progress(i)
        m.unlock()
        await get_tree().create_timer(1).timeout

    # wait for the thread to exist
    while t.is_alive():
        await get_tree().create_timer(1).timeout
    # clean up thread (normally blocks unless thread is dead)
    t.wait_until_finished()

func _long_work_thread():
    for i in range(0,100):
        # update the progress bar, ensuring no-one else is
        m.lock()
        progress_bar.set_lower_text("Long process step %d" % i)
        m.unlock()
        # simulate a long process
        OS.delay_msec(5000)
```

Okay, so the test case worked, even though I was sure I was doing
the same thing in my game code. The long work thread continues to
update the progress bar and this visually updates, and the code
in `_ready()` waits for the long work thread to complete.
It turns out, I wasn't doing this actually, and the detail is interesting.

The use of `await` in `_ready()` is because we don't want to block,
we know that we can't call `OS.delay_msec()` because it would block
and then rendering would stop. In the thread, we *can* just block 
(after all, that's the point of the thread), so we call `OS.delay_msec()`.

Going back over my game code, surely it doesn't matter in a thread
if we block or not, the thread is separate and we can do what we
like. Oh, no, we definitely can't.

Let's use a non-blocking delay instead in the long work thread, change
`OS.delay_msec(5000)` to `await get_tree().create_timer(5).timeout`
which is the same 5 second delay. Suddenly now the main code in
`_ready` doesn't wait for the thread to finish, it just plows through
having been told from the result of `t.is_alive()` that the thread
is dead.

It appears that you *cannot* call `await` in a thread other than the
main thread. For whatever reason, the thread is lost and forgotten
and you can't get back to it. It is not the case that creating a
thread gives you the same properties as the main thread *at all*.

The reason this tripped me up is that I was also doing some fancy
fades using tweens to the progress bar. The common way to write
this (and comes up in all the searches BTW) is:

```python
var tween = create_tween()
tween.tween_property(progress_bar, "modulate:a", 1.0, 1.0)
await t.finished
```

So naturally, in my thread for doing long work, it started with
that exact fragment. And then the thread blew up.

In fact, in the main loop we can do that, but we *must* block if
we want to do that in a thread, and it becomes:

```python
var tween = create_tween()
tween.tween_property(progress_bar, "modulate:a", 1.0, 1.0)
while tween.is_running():
    OS.delay_msec(100)
```

Annoyingly, this also still causes rendering issues, but mostly
it seems to be because `OS.delay_msec()` might just busy-loop and
so it starves something else of cycles, so you get stuttering and
uneven tweens.

The alternative to a thread for long work is our friend `await` and
littering your long work with lots of calls to `await get_tree().process_frame`
which will gracefully leave to allow rendering to continue for a whole
frame, and then come back. This is pretty much how a single-threaded
game should be doing things, but it certainly feels rough to write. 

Anyway, that's two days I won't get back. 

I'm unsure if I'll revert out of threads for things like world gen,
or leave it there. Now it has no problems with unexplained "death"
it works so I'm tempted not to touch that whole space for a while.

Performance was also problematic for generating the world (even in
a nice thread). There were quite a few cases where I was using
a `Dictionary` for performance and then throwing that all away by
forcing GDscript to iterate over an `Array`, which is a lot slower.

For example, here's a few of ways of testing if we have something:

```python
var big_thing : Dictionary()

# ...

if big_thing.has("key"):
    # ...

if "key" in big_thing:
    # ...

if "key" in big_thing.keys()
    # ...
```

The first is obviously using the dictionary correctly. The second and
third I thought were the same, because of the `in` operator. This is
not the case: using `in` with a `Dictionary` is functionally the same
as calling `.has()` on it, while using `in` with the result of `.keys()`
of the `Dictionary` produces an `Array` and then uses the `in` operator
on it which is really really slow.

I've been mixing all three types without much care, and it actually
matters a lot. I still randomly pick between `x.has(key)` and `key in x`
which isn't so good, but for now at least they both don't suck.

This alone was responsible for a large amount of lost time in world
generation.

## Saving and Loading

After the fun with threads, I got on with actually useful progress
(ha ha) and implemented saving and loading the game state. This
is mostly easy, and apart from not clearing out some state when
loading, it just works.

Save games are packed binary files, because serialization to say
JSON is kind of annoying. Lots of types you *have* to munge and
can't just call `JSON.stringify()` on a wodge of data. Instead
you have to pick out and transform some things to strings before
trying to throw them into JSON.

It's not that `stringify()` can't convert some types to strings,
it does that just fine. It's that the inverse function, `parse()`
doesn't understand how to reverse `stringify()` for all cases.

Take a `Vector2`, the string version is something like `"(1.0, 2.0)"`,
which is just fine, it's what `JSON.stringify()` produces, it's valid
JSON so all good. However, `parse()` just sees that as a string and
gives you back .. a string. Fair enough, it **is** a string, but it's a
string you -- Mr JSON class -- put there. You want to give me back
what I put into you? "Hell no" says Mr JSON class.

Thankfully, compared to the threading stuff, that's easy to deal with.
Don't use JSON for save games.

(Aside: Not that binary is free of any quirks either. The binary
store/load functions have `.store_string()` which most definitely
does not "store" a string like every other function with "store"
in the name. There is no `.get_string()` to match it, while every
other "store" function has a matching "get". It wasn't immediately
obvious if you want to store/load a string you really need to 
use `.store_pascal_string()` which has a matching "get" and 
is a great throwback to a long forgotten language.)

## Next up

Because I can't leave well enough alone, I'm exploring a different
tile set to the one currently being used. It has some nice features,
like hexagons are more obviously visible, and a lot of interesting
decorations for tiles.

While playing around with a different tile set, it's also become 
clear I am not handling materials on tiles in a way that makes it
a bit easier for me to tweak the tile colouring and surface. So
we'll also be fixing that for the tile sets as well.