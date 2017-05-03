---
layout: post
title:  "What is an Engine?"
date:   2017-1-28 1:44:00 -0500
categories: 3d opengl camera gamedev programming architecture
---

## What is a game engine anyway?

When I would demo VR programs to the attendees at [Boston VR Meetups][BostonVR], some people would ask with a smile: "Did you make this with Unity? Or Unreal?" As if there was no other conceivable choice! No, I did not use either of those - nor did I use CryEngine, Stingray, Lumberyard, Unigine, Ogre, or any of the [hundreds of available engines][Engine List].

Forgiving these kind people for any ignorance, I've wondered how best to answer that question. Firstly, any kind of frustration or derision in my response would be completely unnecessary(please forgive me if any came through). Any deeply technical answer would likely be well out of the scope of a question  asked in politeness and casual interest. So how best to answer in a single sentence?

Usually I would begin my answer with simply "OpenGL", which was often met with something of a blank expression. I would then explain that I had written all the code in the app[^1] to handle rendering, input, etc. and there really is no "engine" component, per se. I would continue by torturing that metaphor, explaining that the code was perhaps more like a "bicycle"- simple, self-contained, with few moving parts. Human-powered, perhaps? Low emissions? Is anyody still listening at this point?



[BostonVR]: http://bostonvr.org
[Engine List]: https://en.wikipedia.org/wiki/List_of_game_engines

[^1]: They're called "experiences" now(for VR). I still think of them as "programs".


## OpenGL with Higher Level Languages

In the past couple of years, I have been enjoying programming directly to the OpenGL API using languages *other than* C and C++. I've found the experience freeing, eye-opening, and very pleasant.

I whipped up a couple of example demos(frameworks? test beds?) using higher convenient higher-level languages that have OpenGL bindings available. Python was an obvious choice as it is just so pleasing to look at and work with, and has excellent, well-maintained bindings. Java was another obvious choice as the lingua franca of the very popular Android platform, but one that I personally found overly restrictive(maybe I'm doing it wrong). LuaJIT with its FFI was an incredibly fortuitous choice, and the one that I've stuck with to this day. I think it is an absolutely *fantastic* lever to bootstrap a high-level environment([some might argue it is a Lisp][Lua is Lisp]) while mainting a strong connection to low-level details.

[Lua is Lisp]: https://github.com/meric/l2l


<div style="
background-color: #eeeeee;
margin-left: 40px;
padding: 10px;
border: 10px solid white;
//outline: #222222 groove thick;">
<font size="+2">Higher level language aside</font><br>
<br>
I'm open to trying out a new language for programming OpenGL. I had my eyes on Ruby, but was not able to make it work on Windows. I would warmly welcome any assistance on the matter.
<br><br>
Javascript is another good choice of language, and works great with WebGL in a browser. I'd like to try plugging it into regular desktop OpenGL 4.5, but not sure how to go about it. I'd probably start with a small implementation like Duktape.
<br><br>
Racket is another option, and it has some bindings already. DrRacket IDE is a fully functional environment, and there is even <a href="https://github.com/jeapostrophe/racket-android">an Android option: racket-android</a>.
<br><br>
Something functional like Lisp or Haskell might be educational. Perl would be another good option for our aging programmer friends.
<br><br>
I think all a candidate language really needs is a way to allocate memory "natively" and lay out data in specified formats(32 bit float, 8 bit unsigned char, etc.). Sometimes that's handled by the binding code to present a more idiomatic interface to the language.
</div>



## Feature Accretion

So working with LuaJIT and OpenGL over the months and years, features began to collect in my repository. First it was the "Scene" abstraction, then the post processing layer. A timestep function, init/exit functions. Then came a layer to decouple the windowing toolkit(glfw in this case) from the rest of the code. Once I extracted a camera "class", I realized that this thing had indeed become an engine.

So I pulled up a copy of what I think is the definitive source on the subject: [Game Engine Architecture by Gregory, Lander, Whiting][Game Engine Architecture]. The book presents a [layered architecture diagram][gamearch_layers] that identifies all the major components. I use this image like a menu to browse from, picking features as needed. With each new feature choice comes the decision of how to implement it - wondering which existing library can be put to use and its characteristics, especially what its dependencies are. Should I bring in another C++ dependency? A binary? How will it integrate with the system? Since I'm so enamoured of the LuaJIT programming language, I'll inevitably ask: Why not write it in Lua?

 - I'd rather deal with LuaJIT code.
 - I know LuaJIT code will be portable.
 - It'll probably be smaller.
 - It'll probably be fast enough

The only components I can see that can't be done in LuaJIT are in the Platform Independence Layer and are tied to specific hardware.

 - Memory (LuaJIT has it)
 - File System (LuaJIT has it)
 - GPU (OpenGL via FFI)
 - Audio (Bass via FFI)
 - Networking
 - Threads


[Game Engine Architecture]: https://www.amazon.com/Game-Engine-Architecture-Jason-Gregory/dp/1568814135
[gamearch_layers]: http://www.gameenginebook.com/img/fig-runtime-arch.jpg

## Architecture

Here are the main components of the [opengl-with-luajit][opengl-with-luajit] repo and how they depend on each other:

![dependency graph](/assets/luajit-engine-arch.png)

[opengl-with-luajit]: https://bitbucket.org/jimbo00000/opengl-with-luajit

Here are the modules within the util component and their dependencies:

![dependency graph](/assets/luajit-utils-arch.png)

Stateless collections of functions live at the bottom layer, and components compose one another to build higher level utilities. You can just keep building higher and higher and see where the perf starts to kill you.


### Engine Strengths

Many engines are out there with very similar feature sets - what sets this engine apart? I'd say that this is primarily a **code-oriented** engine. While many engines present some sort of GUI to allow users to create something with no code, this one exposes its every possible aspect to the programmer.

I've also tried to keep LOC to a minimum while still being as feature-complete as possible. Some kind of bang-for-your-LOC metric is what I'm optimizing for. Hot-loading scene modules is another selling point, and since it's all LuaJIT it should all be editable at runtime.

Portability is always very important to me. Right now the engine works on Windows, Linux, MacOS[^2], TrueOS[^3], Solaris[^3], and Android. This preserves the user's choice in platform.

[^2]: MacOS support for OpenGL is lacking due to strategy tax and generally uncharitable business practice.

[^3]: TrueOS and Solaris may not have a working build of Bass.

### Shortcomings

The one major feature that is still a big unknown in this engine is **threading**. There is no straightforward way(that I know of) to spawn a thread in LuaJIT - we have to rely on the underlying OS and an API like pthreads for that. One option is providing a callback and using IPC like sockets to communicate with different `Lua_State`'s on each thread. It might make sense to pre-create a thread pool with one thread per physical core. OpenGL context sharing is another unknown here - do we want to do it? Why and how?

[Lua Lanes][lanes] are another potential alternative...

[lanes]: https://github.com/LuaLanes/lanes

For the time being, this is still just a hobby project so I think I'll keep the simplicity goal and just push it as far as it can go. I've been trying to do as much as possible in GL compute anyway, so maybe just one thread is adequate for any purposes that a single mediocre developer could need.


