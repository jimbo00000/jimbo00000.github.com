---
layout: post
title:  "Cross-Platform Libraries"
date:   2017-2-6 09:44:00 -0500
categories: 3d opengl gamedev programming architecture audio sockets
---

Software portability preserves the user's choice of platform. This choice is a necessary, but not sufficient condition for freedom. Since freedom is important to me, I invest the time to familiarize myself with different computing platforms, installing several of them on my primary machine. I highly recommend it as it gives a broader perspective of the nature of computing, as well as a larger skill set.


My first inspiration in this field (LuaJIT game engines) was [malkia's ufo][ufo]. It looked like a really well organized project with a ton of external library support. I've since discovered that most of the included libraries are Windows only, but I am grateful for the effort and at least the acknowledgment of other systems.

[ufo]: https://github.com/malkia/ufo

### Just how portable are we talking here?

There are [a lot of operating systems][OSList] out there. As much as I would love to run code on or at least try every one, it's not feasible. So since I am a graphics enthusiast, I've limited the scope of my portability concerns to those operating systems with working hardware-accelerated OpenGL drivers:

 - Windows
 - Linux
 - Solaris
 - FreeBSD
 - Android
 - MacOS

I would like to see this list grow.

[OSList]: https://en.wikipedia.org/wiki/List_of_operating_systems

## NDK providing a C++ Basis

I managed to get a [working OpenGL skeleton app][HelloGL3] in C++ easily enough - single-threaded with callbacks for display and timestep. The same old main loop thing I've always done, even though that main loop was now hidden somewhere in Java.[^1]

[HelloGL3]: https://bitbucket.org/jimbo00000/android-gl/src/f62144cb8eefb472b94bf1c769714732f3d3d554/HelloGL3/?at=master

## Language

Then it was time to link in [LuaJIT][LuaJIT]. Even though the LuaJIT build process is extremely clean and well designed, and even documented, I couldn't quite get it working on Windows. There's cygwin and MinGW, two extremely welcome POSIX-like compatibility layers, but neither is exactly the same as, say, a Linux system.[^2] So, I just googled around a lot and eventually found a stray arm7eabi binary of `libluajit.so`. I didn't even check which version it was, just dropped it into `jnilibs/` like someone said to on SO and hoped for the best. To my delight, it compiled in and executed and I knew the door was open for dynamic coding on a portable platform.

[LuaJIT]: http://luajit.org/luajit.html

Later, I tried the LuaJIT build process on Ubuntu. [It worked like a charm][android-build] - just point it at the NDK and the makefile takes care of the rest.

[android-build]: https://github.com/jimbo00000/Flickercladding/blob/master/android_build.sh

## Video

It took a good amount of hacking and fumbling around to get LuaJIT calling into OpenGL ES on Android. There is a surprising(to me) lack of documentation on the subject, and in retrospect I'm kind of amazed I got it working at all.

I had to fumble around some more to get LuaJIT linking in to `libGL.so`. I first had to find its location on the filesystem (in `/system/`), then do an `ffi.load` on it. Then I pasted in the `gl3.h` header into an `ffi.cdef` string. Meanwhile, on the desktop, I was [messing around with function loaders][loaders]. I finally got both paths working and refactored to share as much code as possible.

[loaders]: http://jimbo00000.github.io/opengl/portable/programming/scripting/2016/01/23/opengl-function-loaders.html


## Audio

A little later, I thought it would be nice to add some audio into the mix. I noticed that [Bass][Bass] by Ian Luck had an Android build - I grabbed the arm7eabi .so file from the download link at the top of the forum thread, dropped it into `jnilibs` and to my great surprise - actually heard audio from my Android device! This was a big thrill for me. I thought Android was this mysterious thing where none of the rules or concepts of a "normal" development system applied. But finding out that we can just link to shared libraries like we always do seemed to open up a lot of doors.

## Networking

Now that I had audio and video, I wondered what other hooks into the underlying hardware I would need for full game functionality. The one remaining feature that I would love that can't be faked is **networking**.

### Library choices

#### LuaSocket

My first choice of library was LuaSocket. I already use it(if only for the high resolution timer function) in the pure LuaJIT desktop engine, so it would be only natural to use it for actual networking as well.

I managed to get it bulding easily enough. So, .so file in hand, I dropped it in and launched the app. Link error. Tried changing the path from `socket/core.so` to just `core.so` to match the flat lib directory convention on Android, but still no dice.

The LuaSocket `socket/core.so` libary has to make calls into `libluajit.so`. One copy of that library is linked in the build process, is that one statically included in `core.so`? How can `core.so` link in with the libluajit.so that's already in `jnilibs`? Is it possible to use two different libraries? What about the runtime state? If anyone can help with this, please let me know.

#### Other

So I started searching for other networking libraries with Android ports.

ZeroMQ is a strong possibility, though they recommend strongly(for unknown reaons) *not* to use the NDK, but to use Java instead. Maybe I can add some kind of callbacks for this, but for now I've set it aside.

libuv


Raw Sockets? 


#### No native option?

What if there's no exposure of the socket lib to native code? Is it possible to expose socket function callbacks back up to the Java layer? Is there a huge performance penalty for this? Is this something anyone would want to do?


[^1]: Now there's the android-app-glue project which exposes the main loop to the developer. It is on my list of things to try.

[^2]: Now there's bash-on-Windows, which might also be worth a try. Or you could just boot into Linux...

[Bass]: http://un4seen.com
