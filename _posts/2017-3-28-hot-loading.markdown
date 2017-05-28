---
layout: post
title:  "Hot Loading Modules"
date:   2017-3-28 3:14:00 -0500
categories: gamedev programming architecture
---


The dream of the all-source engine is to edit any part of it while it's running. In practice, I'm not sure it's possible. But how could we make as much of it editable as we can?

LuaJIT provides much of what we need in its module loading system. We can use `require` and `dofile` to load a module into our runtime, and delete it with `package.loaded[name] = nil` (once the gc comes around).

A module is directly analogous to a library in C. It's a bunch of code with entry points. There may also be some module scope code that gets executed at load time. This code may contain prototype "constructor" functions that create tables of state that may persist after their creating module is unloaded and reloaded.

### Handling Errors

What do we do if the code loaded in has any syntax errors? At the very least we'll need a copy of the last working version of the module(essentially as double-buffer). Maybe a whole list of these with undo support would be nice.

Infinite loops are another class of problem. Since there is no way to determine this by analysis, we have to just try it out. How can we have a watchdog that will kill our process after a half second or so? Maybe we can do it on another thread? What extensions and OS support do we need to pull this off? Context priority? Shader pre-emption? Both?

Slow-running code is yet another class of error that comes up in VR. If a module causes us to drop frame rate, we have to axe it immediately. Could we use the same watchdog thread with a 1/60 second timeout?  What if we had a second thread with its own GL context and run it there? We could share the output render texture with the main VR render thread and clock its FPS independently. This would grant the user a way of vetting incoming code by checking not only its frame rate but its actual output on a preview frame. Could we pull just surplus rendering power from the system for the guest code, ensuring our local compositor runs at maximal QoS? How can we allocate priority with low latency at runtime to GPU tasks?

#### Practice Run - Editable Shader Code

Editing shader code is a good first pass toward the goal of live-edited hot-loaded code. I'm not sure it can ever make the process crash, but you can see what happens on an infinite loop.


## What we ultimately want

We need a compositor that positively cannot crash and must run with consistent quality of service. Its runtime needs to be bounded. It has to share GPU memory with any user processes, which sounds like it could be a huge security hole in a memory-protected OS. In practice, we can impose specific restrictions on the shared surface: the compositor only needs read access to a contiguous block in a known format.

The compositor proces should be the top priority process in the entire system. Everything else can crash but that. The core of the system is a loop that reads a matrix of head pose from a hardware subsystem right before each scanout(or chunk wise before each scanline), performing timewarped compositing and dumping pixels to the HMD. Once done, it can hand off that head pose data to any user programs/scenes which are free to write to the shared compositor surface or crash and burn. A watchdog could indicate if a user process hasn't met its deadline and exclude its layer from the compositor if necessary.

