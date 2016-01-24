---
layout: post
title:  "OpenGL Function Loaders"
date:   2016-1-23 1:15:22 -0500
categories: opengl portable programming scripting
#published: false
---

## Falling from bliss

As long as everything goes smoothly using a [higher-level language with OpenGL][glscript] the experience is magical: it feels kind of like flying. But sometimes you hit a bug the debugger won't catch(e.g. NULL pointer dereference) and it feels like crashing hard to the ground and skidding over dirt and rock.

[glscript]: http://jimbo00000.github.io/opengl/portable/programming/vr/scripting/2016/01/09/vr-scripting.html


Everything in [RiftSkel][RiftSkel] was working fine on both Linux and Windows, but when I did a `git pull` and ran it on my VR system - crash. This was odd enough that I had to try it a few times and do a few sanity checks. Everything had worked fine on that very system when booted into Linux just the previous day. Everything also worked fine on a different machine(Surface Pro 3) running Windows 8 just minutes before.


### Hardware or OS issue?

The table of functionality by GPU and OS looks like this:

[RiftSkel]: https://bitbucket.org/jimbo00000/riftskel

|  GPU/OS    | Win7     | Win8     | Linux |
| ---------- |:--------:|:--------:| -----:|
| **Intel**  |          | fine     |       |
| **NVIDIA** |          | CRASH    | fine  |
| **AMD**    | CRASH    |          | fine  |

<br>

At this point, I am perplexed as to why the program is running fine on both Windows and Linux, yet crashing on this particular machine only on Windows. Confusing me even further, the [pure luajit main][luajitmain] which called into the very same scene code worked fine on the VR machine. It was only in the [embedded luajit within the C++ app][cppmain] that the crash occurred.

The [gamescene][gamescene] script was still working OK. Just as soon as I switched over to [cubescene2][cubescene2](the one with textures), the program crashed. This was a tremendous clue - I bet those with lots of OpenGL experience already know what the problem was. I myself had to hack out all the GL functions call by call until I fould the ones that were crashing - texture functions such as `glTexParameteri` and `glTexImage2D`, and `glGetIntegerv`.

[luajitmain]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/lua/main_glfw.lua?at=master&fileviewer=file-view-default

[cppmain]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/src/main_glfw_ovrsdk08.cpp?at=master&fileviewer=file-view-default

[gamescene]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default

[cubescene2]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/lua/scene/cubescene2.lua?at=master&fileviewer=file-view-default


## OpenGL Function Loaders

It turns out there is some legacy oddness related to [loading OpenGL function pointers on Windows][windowsfuncs]. I had heard of this before but never been bitten by it as the excellent [GLFW][GLFW] library handled it all for me. In the Luajit main, I did as I always did and [used glfw's loader][luajit_use_glfwloader]. But, in the embedded version, my thinking was that I didn't need to use Luajit GLFW bindings as the window was already created by the C++ app, so I could just use `wglGetProcAddress`/`glXGetProcAddress`. Coincidentally, this created no problem with [gamescene][gamescene] as gamescene used no functions from OpenGL 1.1 or earlier. But as soon as a scene added one of those older functions directly exported by `OpenGL32.DLL` - NULL dereference and crash.

The way to handle this is to check for invalid values returned from `wglGetProcAddress` and in that case look up proc addresses from Windows's `GetProcAddress`. GLFW does it [here in _glfwPlatformGetProcAddress][_glfwPlatformGetProcAddress]. Since I wasn't doing it manually, all the OpenGL 1 functions were coming back NULL.

I thought the simplest way around this was to [pass the function pointer for `glfwGetProcAddress`][ljscene] from C++ where GLFW is linked right into Luajit. This required casting it to a double for `lua_pushnumber` then [casting it back to a function pointer using the FFI][loader-cast]. I'm pretty sure this will break with a 64-bit function pointer.

[windowsfuncs]: https://www.opengl.org/wiki/Load_OpenGL_Functions#Windows
[GLFW]: http://www.glfw.org/

[luajit_use_glfwloader]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/lua/main_glfw.lua?at=master&fileviewer=file-view-default#main_glfw.lua-6

[scenebridge_loader]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/lua/scenebridge.lua?at=master&fileviewer=file-view-default#scenebridge.lua-22

[_glfwPlatformGetProcAddress]: https://github.com/glfw/glfw/blob/6b0f6601807ea67ccba374688c136e1c4e85f98b/src/wgl_context.c#L661

[ljscene]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/src/Scene/LuajitScene.cpp?at=master&fileviewer=file-view-default#LuajitScene.cpp-50

[sb_initgl]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/lua/scenebridge.lua?at=master&fileviewer=file-view-default#scenebridge.lua-21

[loader-cast]: https://bitbucket.org/jimbo00000/riftskel/src/a3b5b18d514d38bd8b84b4448a11aae368e809b1/lua/scenebridge.lua?at=master&fileviewer=file-view-default#scenebridge.lua-41


### But why does it work on Intel/Win8?

Does the Intel graphics driver handle the two-phase function lookup silently? 
If so, why don't the NVIDIA and AMD drivers do this?
If not - where do the correct proc addresses come from on Surface Pro 3/Win8?
I would love to find out more of the details about this.


## Everything works again

In the meantime, work can proceed. Back on the jetpack, woohoo!!
