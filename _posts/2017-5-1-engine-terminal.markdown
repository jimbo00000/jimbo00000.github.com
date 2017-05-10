---
layout: post
title:  "Engine Terminal"
date:   2017-5-1 3:14:00 -0500
categories: gamedev programming editor terminal commandline cmd prompt
---


## Control

The Joule Plating project is all about control over the rendering process - and what better instrument of control is there than the command line. It's about as fully general as anything you can find today, and it packs an excellent bang-for-your-LOC. Its impementation is really simple, so why not give it a try?

### Engine Components

There are three main components of the engine owned by main:

 - Scene
 - Effect
 - Camera

Each of these things is implemented in a Lua-style class which can be substituted for a different one on the fly. Each one is a potential target for commands, as we'd like to modify them at will.

How can we distinguish which of the 3 components we would like to issue commands to?

I thought it made sense to differentiate between targets by specifying one byte as the first arg: 's' for Scene, 'e' for Effect, e.g.
`c reset` to reset the camera. This is kind of awkward to have to prepend to every command - can we components consume events? What is the priority order? 


#### First pass - set variables directly

You can set class's variables directly from main - just reach in and add a field: ```Scene['key'] = value```. But if you want a table of values, I don't think you can do ```Scene['table.key'] = value``` - no automatic nested table handling. For that, we'd either need to split the variable name on '.' and create the tables all the way down(something we could do in main) or add a handler function in every Scene, Effect, and Camera. Or both!


#### Second pass - Handler API entry point

```function my_module:onCommand(args)```
This optional entry point can and maybe should be added to every component to allow it to handle terminal commands.

```function my_module:commandHelp()```
This one is even more optional, but think how nice it would be if you gave the next person to use this a clue.


### Default uniform values

I read that GLSL supports default variables for uniforms 
```uniform float speed = 1.0;```, but my Intel driver does not support this. Kind of a shame, as it would nicely decouple the effect shaders from their callers... Instead, we either have to package a table of keys/values with the shader code(in Lua, so it's not too bad) or maybe do some textual processing and keep it in the GLSL source. We could probably grab out any default uniform values at shader creation time... Maybe that's why it's not implemented?



https://www.opengl.org/discussion_boards/showthread.php/165141-Default-value-for-uniform


http://prideout.net/blog/?p=1


### Effects and Multipass

Both Scene and Effect can initiate multiple dependent draws. `effect_chain` still works in a linear queue fashion rather than the more general shade tree, but trees can be coded in either module. Hell, for that matter the camera could have its own buffers too. And run its own render passes.


