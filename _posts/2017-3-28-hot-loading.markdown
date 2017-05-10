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

## Practice Run - Editable Shader Code

Editing shader code is a good first pass toward the goal of live-edited hot-loaded code. GLSL is passed to the driver as a string for compilation at runtime via `glCompileShader`. None of the calling code needs to be changed for this to be realized, so the flexibility is not total, but it is enough to play with and get some real results. Parts of the implementation can be reused for a more complete self-editing system - the source editor and the terminal.

### Controls

The editor and terminal can be toggled by a key. Which key is a matter of taste, I like to line them up under escape.

#### Key values/layouts

#### Mouse clicks too

### Error layer

### Recompile on each keystroke

### Hide/show and the "terminal"

### Tweakbar?

## Where does it live?
Include in scene
Move out to main? An intermediary?


## Shader types
Single shader string only
VS TCS TES GS FS CS
Feedback


#### Future plans
DrawIndirect?

