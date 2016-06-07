---
layout: post
title:  "OpenGL Drivers: Register Pressure"
date:   2016-6-4 21:44:00 -0500
categories: opengl amd glsl compiler
---

I learned something today about OpenGL drivers.[^1] As I submitted my [demo entry for @party][@party], I tried a dry run on the compo machine - and it crashed immediately on exit with a warning:

```
__ProgramInfoLog: WARNING: Too many temp register is used in Vertex shader, it may cause slow execution.
```

Running with debug mode on gave no more information.

[^1]: I also re-learned a lesson I was supposed to have already known, which is **test your program on the target platform beforehand**!


[@party]: http://atparty.untergrund.net/


## Preparation

Rewind back to 2 days prior, when I actually had ample time to fix this. I got the same warning message for the [DNA scene][DNA scene] on a Linux Turks box. I disregarded it, thinking that the new [@party compo machine][compo machine] with its R9 Fury would surely have more registers, and the problem would not manifest come party time.

Well, it turned out that it did.

[DNA scene]: https://bitbucket.org/jimbo00000/sechs/src/f99a72c82ce150ae22ea213b2b28c96161c25e95/scene/dna_scene.lua?fileviewer=file-view-default#dna_scene.lua-76

[compo machine]: http://atparty.untergrund.net/compo-machine-uuuuuupgrade/

## Pressure

So there I am with about 2 hours to deadline and no real clue how to fix this but what I came up with in a hasty google search from the previous night: try to declare variables closer to their use, so the registers backing them can be re-used. I noticed that the other scenes weren't crashing, and they started life as a copy of the DNA scene and were still very similar. The vertex shader seemed the obvious culprit, and the most noticeable difference was the [second array of geometry data][geom array] that only the DNA scene had.

So, realizing I could just calculate those vertex positions on the fly, I wrote out the first array. Came back to the compo machine with version 0002 to find it crashed again. Crap.

### Why did I do this in the first place?

Why was I storing geometry data directly in arrays in the VS? Well, it seemed like a good idea at the time. I wanted to draw a lot of something, and instanced rendering is always a good fit for that. Last time, i used `glDrawElementsInstanced` for the cubes comprising the tentacle of a jellyfish, so I had to loop over an array of them to draw the bunch. This time, I wanted to use instances for the individuals, so I opted to push geometry "forward" in the pipeline, allowing me to reuse it as instances. I was using `glDrawArrays` for this, having forgotten about the `glVertexAttribDivisor` call, which can be set to arbitrary ints. Looking back, I probably could have done this more correctly using that.

[elements draw]: https://bitbucket.org/jimbo00000/jellyfish-the-jam/src/fe691b42fa6a414925f05d97275114af099b3c78/standalone/scene/jellyfish_tentacles.lua?at=master&fileviewer=file-view-default#jellyfish_tentacles.lua-226

[geom array]: https://bitbucket.org/jimbo00000/sechs/src/f99a72c82ce150ae22ea213b2b28c96161c25e95/scene/dna_scene.lua?fileviewer=file-view-default#dna_scene.lua-102

### How do I fix this thing on AMD?

[Now I'm pretty much flying blind][commit msg 1] with only my Surface Pro 3 (w/Intel GPU) and a vague idea of trying to reduce the number variables in shader source. I'm doing that, copying expressions directly into other expressions to eliminate temp variables that could have been declared const. Doesn't the compiler do this for me? Common subexpression elimination, constant folding - isn't this standard practice? When I worked at AMD, the shader compiler did about 50 separate optimization passes over shader code just to squeeze every last ounce of performance out of that which mattered the most. Why couldn't it figure this out when NVidia's and Intel's compilers did just fine?

[commit msg 1]: https://bitbucket.org/jimbo00000/sechs/commits/5831a404650e570ce6db96e84f012e069e49cdd7?at=master
[commit msg 2]: https://bitbucket.org/jimbo00000/sechs/commits/930c1098b25f42d951f0bf132757646bfa766253?at=master
[commit msg 3]: https://bitbucket.org/jimbo00000/sechs/commits/930c1098b25f42d951f0bf132757646bfa766253?at=master
[commit msg 4]: https://bitbucket.org/jimbo00000/sechs/commits/accd34a14321572a3a78aff8d594c171d480fcc5?at=master

Looking back on it, it now appears that it's the GLSL "front end" compiler's shortcoming. Before that compiler team I worked with ever even received code, it had to go through this GLSL front end which is of course very large and cumbersome and one of the primary motivations for Vulkan. And I guess there was where it did *not* do expression folding, or any other technique to reduce the number of required registers to something the GPU actually has.

So at 5:59, I packed up [my last attempt][commit msg 4] at this, tasting bitter failure as I resigned myself to just going home if it crashed. To my great surprise, it worked! The 6 fold Julia set at the beginning got dropped for unknown reasons, but the demo played and won first place.

## Test your code!

The moral of the story - test early and often!
