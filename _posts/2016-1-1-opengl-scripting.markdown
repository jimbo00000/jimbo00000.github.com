---
layout: post
title:  "OpenGL Scripting: Python"
date:   2016-1-1 10:37:22 -0500
categories: opengl portable programming scripting
---

Graphics are what makes programming fun for me. I prefer to use OpenGL because I value portability. I've been using C++ for almost the entire 15 years or so I've been programming in OpenGL, but only recently have I discovered the fun in using higher level languages to call directly into OpenGL.

I first started using Python for text processing tasks, appreciating its built in string functions. I soon began to love slicing, list comprehensions, and pip. I used it to [hardcode GLSL shader source files][hardcode_shaders_py] into C++ string literals in headers in order to avoid one more resource deployment headache. Then I found out about PyOpenGL and realized I could cut out not only this [pre-build step][invoke_python], but cmake itself and the entire compiler as well! This seems to me a great way to do more with less.

### Learning OpenGL with Python

Python bindings for OpenGL can open the door to graphics programming for people who are not familiar wih(or prefer not to use) C++. If you're a python programmer who'd like to try learning something new and awesome this year, check out the [opengl-with-python][opengl-with-python] repo. I've even included a set of test scenes([scene01][scene01] - [scene07][scene07]) to gently bring you up to date from the GL 1.x fixed-function pipeline days to modern GL.

#### Setup

 - Install python and add it to your path. I'm using the 32-bit version, as I had some headaches with 64. 32 seems to give more compatibility with libraries.

 - Use pip to install PyOpenGL and numpy[^1].

#### Windowing Framework
The first thing you'll need to do is create a window on your operating system of choice. The easiest way to do this is with a windowing framework like [GLFW][GLFW] or [SDL2][SDL2] (both of them are excellent). GLFW has ready-made bindings available on pip, so I went with that. Try running the file [minimal.py][minimal.py] as follows:
`python minimal_glfw.py`

Once this works, you can plug in callback functions for input events: mouse, keyboard, window resize, etc. to make your program interactive.

#### Ready to draw
minimal.py uses only the simplest GL draw call, `glClear`. If you want to do anything more interesting, you'll need to:

 - compile and use a shader program with `glCompileShader` and `glUseProgram`
 - upload native byte array data(floats, shorts, ints, etc.) with `glBufferData`

Don't fall into the temptation of doing everything in immediate mode with the fixed function pipeline. The right way is to initialize all such arrays once at runtime before any drawing is done, but after a GL context is created. I've created an [entry point for this purpose][scene07_initGL] called `initGL` and a corresponding `exitGL` to deallocate. You'll appreciate how much faster everything runs.

Native arrays are probably the most challenging part of connecting up OpenGL to higher level languages. Most experts seem to recommend numpy for the task, not only for the clarity of its API but its performance as well. I find the numpy dependency to be rather heavyweight to pull in in its entirety just for `numpy.array`, so I've been using python's built-in `ctypes` arrays instead. I have not noticed any differences in these toy examples.

#### Drawing, drawing, and drawing some more 
The main loop calls draw repeatedly, followed by `glfwSwapBuffers`. If vsync is on, the swap call will block and limit the frame rate to the refresh rate of your primary monitor[^2]. If it's off, watch out - the app will tend to become unresponsive to input on Unix-like OSs, spinning so rapidly through its tight main loop that input events can't break in. You might as well leave vsync on for this reason, but seeing that 1300 FPS value does give a rough estimate of your app's base drawing time.


[hardcode_shaders_py]: https://github.com/jimbo00000/RiftSkeleton/blob/master/tools/hardcode_shaders.py
[opengl-with-python]: https://bitbucket.org/jimbo00000/opengl-with-python
[opengl-with-luajit]: https://bitbucket.org/jimbo00000/opengl-with-luajit
[invoke_python]: https://github.com/jimbo00000/RiftSkeleton/blob/master/CMakeLists.txt#L5

[scene01]: https://bitbucket.org/jimbo00000/opengl-with-python/src/978902d99ec4a7537e52060c9b0b1e39064b30e3/scene/scene01.py?at=master&fileviewer=file-view-default
[scene07]: https://bitbucket.org/jimbo00000/opengl-with-python/src/978902d99ec4a7537e52060c9b0b1e39064b30e3/scene/scene07.py?at=master&fileviewer=file-view-default
[GLFW]: http://www.glfw.org/
[SDL2]: https://www.libsdl.org/download-2.0.php
[minimal.py]: https://bitbucket.org/jimbo00000/opengl-with-python/src/978902d99ec4a7537e52060c9b0b1e39064b30e3/minimal_glfw.py?at=master&fileviewer=file-view-default

##### Footnotes:

[^1]: I've tried installing numpy with pip, but it never quite worked on Windows. You apparently need a separately packaged installer executable for that, which complicates its installation for PyPy.
[^2]: VSync is OS- and driver-dependent functionality. Remember to check your settings in your respective GPU manufacturer's control panel as they can override the effect of `glfwSwapInterval`.

[scene07_initGL]: https://bitbucket.org/jimbo00000/opengl-with-python/src/978902d99ec4a7537e52060c9b0b1e39064b30e3/scene/scene07.py?at=master&fileviewer=file-view-default#scene07.py-64