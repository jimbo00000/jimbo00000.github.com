---
layout: post
title:  "Demo Production Lessons"
date:   2017-6-10 3:14:00 -0500
categories: demo prod programming graphics
---


## Control

@party 2017 was an opportunity to exercise Joule Plating's demo tool capabilities. I was aiming to maximize the bang-per-effort ratio, and succeeded in some ways.

Here are some things I would do differently:

### Copy or Include?

Include is great to keep chunks of code small and manageable. Copy is great to isolate a chunk from the impact of any future changes. In a crunch situation, you're far more likely to make a breaking change. In a refactoring situation, copy often requires you to back-port changes.

|                   | Copy               | Include           |
| ----------------- |:------------------:|:-----------------:|
| **Advantage**     | Isolation          | Small code chunks |
| **Disadvantage**  | Duplicated work    | Dependency hell   |


Perhaps the best of both worlds would be a 'bake' option which would compile all the sources from disparate components of the engine into one monolithic source file for direct import to the demo.


### Adding channels in Rocket

Choreographing events in the demo is perhaps the best "bang" return on effort, but required a lot of manual work. Adding channels is not easy, and with each new one, the burden of keeping them organized grows.

In the future, I would like some sort of abstraction layer which can take just a straight float to indicate place in the beat, possibly calculating it from time and bpm, or possibly sync-tracked. That layer could use concepts like 'downbeat', 'stab', 'flare' (or something) which would map to specific parts of the song(notes, beats, maybe measures) and could in turn be expressed with some new graphical idea just by switching a variable value around.

I think that means the first step would be to dissect the song into intervals or events that have some sort of semantic or maybe emotional meaning. Of course it would be easy to take this too far. How do choreographers do it?


### Getting it working on Android

main_demo.lua does all the demo sync and playing work - maybe it could be folded entirely into a Scene. This might make it really easy to drop into Flickercladding and play it on Android.


### Fake time vs. real time

The timestep function takes `absTime, dt` as a parameter, which works fine when playing monotonically forward. But when connected to a sync tracker, there is a different notion of time - that can be controlled by scrubbing back and forth. `dt` becomes meaningless. Maybe a different timestep function is needed here, or maybe the demo engine should just not call it. I had some functionality left behind in the timestep function which caused inconsistent playback when editing which is a pain. It should still be consistent when playing from the start.


### Baking it all into a 4k

I don't think there's any way that LuaJIT code could be baked into assembly for sizecoding purposes.[^1] I guess the next best thing would be to re-implement the Lua code, transliterating it into C++ or Rust.

[^1]: Is there? It would be pretty cool...
