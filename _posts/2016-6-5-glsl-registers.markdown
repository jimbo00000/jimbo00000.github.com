---
layout: post
title:  "OpenGL Drivers: Register Pressure"
date:   2016-6-4 21:44:00 -0500
categories: opengl amd glsl compiler
---

I learned something today about OpenGL drivers. As I submitted my [demo entry for @party][@party], I tried a dry run on the compo machine - and it crashed immediately on exit with a warning message about lack of registers. So I also re-learned a lesson I was supposed to have known, which is **test your program on the target platform beforehand**.


[@party]: http://atparty.untergrund.net/


## Preparation

Rewind back to 2 days prior, when I actually had ample time to fix this. I got the same warning message for the DNA scene on a Linux Turks box. I disregarded it, thinking that the new [@party compo machine][compo machine] with its R9 Fury would surely have more registers, and the problem would not manifest come party time.

Well, it turned out that it did.

[compo machine]: http://atparty.untergrund.net/compo-machine-uuuuuupgrade/

## Pressure

So there I am with about 2 hours to deadline and no real clue how to fix this but what I came up with in a hasty google search from the previous night: try to declare variables closer to their use, so the registers backing them can be re-used. I noticed that the other scenes weren't crashing, and they started life as a copy of the DNA scene and were still very similar. The vertex shader seemed the obvious culprit, and the most noticeable difference was the second array of geometry data that only the DNA scene had.

So, realizing I could just calculate those vertex positions on the fly, I wrote out the first array. Came back to the compo machine with version 0002 to find it crashed again. Crap.

Now I'm pretty much flying blind with only my Surface Pro 3 and a vague idea of trying to reduce variables in shader source. I'm reducing the number of variables in the source, copying expressions directly into other expressions to eliminate temp variables that could have been declared const. Doesn't the compiler do this for me? Common subexpression elimination, constant folding - isn't this standard practice? When I worked at AMD, the shader compiler did about 50 separate optimization passes over shader code just to squeeze every last ounce of performance out of that which mattered the most. Why couldn't it figure this out when NVidia's and Intel's compilers did just fine?

Looking back on it, it now appears that it's the GLSL "front end" compiler's shortcoming. Before that compiler team I worked with ever even received code, it had to go through this GLSL front end which is of course very large and cumbersome and one of the primary motivations for Vulkan. And I guess there was where it did *not* do expression folding, or any other technique to reduce the number of required registers to something the GPU actually has.

So at 5:59, I packed up my last attempt at this, tasting the futility of failure as I resigned myself to just going home if it failed. To my great surprise, it worked! The 6 fold Julia set at the beginning got dropped for unknown reasons, but the demo played and won first place.

## Test your code!

The moral of the story - test early and often!
