---
title: "Hexagon Game: Update for 12 October 2022"
date: 2022-10-12T11:47:55+13:00
draft: false
---

Well there's been a little bit more progress on world generation. I've
been avoiding working on roads (I have a theory) and rivers (lol no
ideas), and doing "city" generation. 

Placing the cities is easy enough. The world is divided into biomes,
and we assign one city to each biome, checking to make sure when
we place it there are no other cities too close. We also check against
a reserved starting area.

The biggest problem is then names. These have to be sourced on demand
from .. something. The best way would be names which were generated
as needed, that were vaguely consistent with a language (ie, "hhhhhhhf"
is not a great city name) and which didn't just come from a fixed list
(because I am not that creative).

One way would be to just use any one of a number of online generators
to produce a "long enough" list for the game, and I did think about
doing this. It wouldn't be as elegant as actual name generation on
the fly, but it's simple to implement.

Although "simple to implement" is often my approach[^1], there were 
some open source name generators based on a very classical approach
called Markov Chains. These produce new "words" by statistical 
analysis of some input word lists and then making new combinations
from the observed patterns in the lists plus some randomness.

There wasn't any good examples I liked in GDscript, so porting one
from another language was the next best thing. Despite how awful
I am usually at writing code, I read it fairly well and have read
enough in a variety of languages to be able to work out how to
translate it to another language.

One of the biggest problems with doing that sort of porting is the
actual algorithm can be opaque enough that the translation is merely
shifting one language's syntax to another syntax and a little bit of
massaging how it uses types and built-in functions.

Usually these things don't come with a nice set of tests to show
the outcome of any function works as expected. This means when it 
doesn't work, if you don't know the algorithm it an be pretty hard
to see where it's going wrong.

After a few days of headscratching, I did finally find the cause of
the code not producing any useful words: substrings. In the original
langauge, there's a function `substring()` which obviously would
map to `substr()` in GDscript: give me some part of the string. 

But it turned out these two functions had different meanings for 
their inputs: In GDscript it's `substr(start,len)` while the original
langauge I was porting from used `substring(start,end)`. It had
*another* function, `substr()` as well which implemented the same
handling as GDscript's `substr()`. But this wasn't obvious when
I did my awful lazy conversion pass over the code.

Never mind, now it works. We have cities with generated names and
a way to generate names for lots of other things.

At some point I'll publish the GDscript code under the same license
as the original if anyone wants it. I wouldn't trust it though!