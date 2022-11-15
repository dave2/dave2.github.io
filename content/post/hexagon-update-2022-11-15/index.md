---
title: "Hexagon Update: 15 Nov 2022"
date: 2022-11-15T15:25:24+13:00
draft: false
---

Well hi there folks. Not sure anyone is reading this, but that's just
fine, it's more for my mental well-being than anything else. Let's 
dive into the state of things.

## Inventory

Most of the last week has been on nothing but Inventory and getting
both the basic UI, data, and storage sorted out.

Unexpectedly, building the UI still goes somewhat okay. There's still
a huge problem in my head between where the UI is doing things, and
whether things like validation of what can be placed where live in
the UI (where, actually, they get called a lot) or somewhere else.

At the moment there is some uncomfortable duplication. What should
really be taking place is the Inventory UI is a view of the data,
not populated with it's own set of the items in every slot. However,
from the way the UI is structure, it is *much* easier to poke items
into a slot in the UI and have the UI render that, than tell the
slot in the UI how to find out what it's contents might be.

It's also getting pretty deep in layers of callbacks to deal with
changes. This is where the way it's being implemented is starting
to break down.

The reason is we're using Godot's built-in UI features and these
mostly fire signals at our top-level elements. In the inventory case
there is an invisible button on top of every slot which gets these
events. 

Now that makes it easy to capture events - no math needed to convert
positional information to identify the slot picked - but we have to 
then look more at callbacks to make it come together with state
the individual button isn't holding.

Anyway, some gamedev will be screaming at the page now going "just 
do it &lt;this way&gt;" and they'll be right. But I'm feeling out all of
this from first principles right?

We're already on a major rewrite of how inventory is stored, so 
maybe I should finish that before starting a new rewrite? 

That current rewrite was triggered by a deeply flawed decision to
have separate variables holding equipped from just stored items.
This resulted in doubling all the functions, and I used strings for
equipment slots and ints for normal storage. And then I mixed up
the slots an item may be stashed in with slots we actually have.

That last point is based on items that either span multiple slots
or can be used in multiple slots. So we have a `TWOHAND` equip
slot on items .. but that doesn't exist in the inventory. Building
all the data structures on what items were using as slots was all
sorts of wrong, but I didn't see that until quite late.

So it's all been rewritten, equipment slots are now just low-numbered
inventory slots, and we special case them when testing for accepting
or stashing an item.

The good news is, because I'm just using Godot's internal drag and
drop I have drag and drop working, you can move items around the 
inventory, and drag them in and out of equipment slots, and it 
even tests if this is an acceptable move. Hooray.

After I do another rewrite I need to add support for stacking and
unstacking items. The item class already has stack properties, and
the UI already has support for showing stacked items, but none
of this is actually wired up. And once stacking is done then it's
on to looting UI and shop UI!

This stacking thing shouldn't be too hard, he said having realised
actually it is hard because items don't have a concept of equality. 
Huh.


