---
layout: post
title:  "RiftSkeleton Rebirth"
date:   2015-12-31 21:37:22 -0500
categories: vr opengl portable programming
---
My VR development had stalled for about two months because I dug myself in too deep trying to support all possible VR SDKs and software platforms while [not repeating myself][DRY]. When the 0.6 OVR SDK came out with the compositor, the change in API and general style was too great for the structure of the old [RiftSkeleton][RiftSkeleton], and it basically broke. The prospect of expending all the effort required to change the code to accommodate it was just too daunting, so I chickened out on it and worked on [something else]({% post_url 2016-1-1-opengl-scripting %}).

The only way forward was to [start a new one from scratch][riftskel].

Why did the code get so complex? What was the old path and why was it a dead end? I was trying to separate out all dependencies so they were easily interchangeable. I wanted the basic app to be as completely *agnostic* as possible: to work with multiple VR SDKs(OVR, OSVR, none), multiple windowing frameworks(GLFW, SDL2, SFML), multiple OSs(Windows, Linux, MacOS, FreeBSD, Solaris). To that end, the [AppSkeleton][AppSkel] class tree was supposed to pull out all 3D-world-related navigation and scene management into the base class, leaving the subclasses like [OVRAppSkeleton][OVR08AppSkel] to worry about the API-specific business like `ovrSwapTextureSet` or `OSVR_ClientContext`. Then I could just add a new subclass to handle a new VR API. `OpenVRAppSkeleton : public AppSkeleton`, no problem! I even considered adding a SConstruct file on top of CMakeLists.txt to diversify build systems. The only bedrock-level requirements were C++ 11-or-so and OpenGL 3-ish or better.

This abstraction really started to break with SDK 0.5, when the runtime would handle buffer swapping. Suddenly, there was a functional interaction between *windowing framework* code and *VR SDK* code, so that meant there had to be a branch somewhere in the GLFW code(and the SDL2 code). Each branch is an artifact of some VR API or runtime difference of opinion, some different way of doing things.

This got ugly *really* fast. On top of that, I had this whole movable dashboard system implemented with Hydra support that was almost completely obviated by the `ovrLayerType_Quad` layer type. [RiftRay][RiftRay] has all the "floating quad in world space" stuff implemented and working in what I called the "[PaneScene][PaneScene]" abstraction, which was drawn to a larger render buffer(FBO) separately from the more expensive raymarched shadertoy scenes. Brad Davis does the same in [his book][ORIA] and it allows text in floating windows to avoid the blurry graininess of downscaled rendering.

OVR SDK 0.6 gives the floating window panes in space to us for free, [complete with pose and size in world space][ovrLayerQuad_dox] - it'd be crazy not to use them. And I think OVR SDK 0.8 is supposed to be API-compatible with the upcoming 1.0(if not binary compatible - where did I hear this again?), so now was the time to get this working. [The main source file][main_glfw_ovrsdk08.cpp] is now only 600 lines long with GLFW and OVR SDK code mixed together in there, and it's so much nicer to be able to [refer][45] to [global][257] [eye][419] [poses][510] without having to pass pointers through encapsulation barriers.

At least the [IScene][IScene] interface is still intact, so all the time I never got around to spending on designing graphical scenes for VR wasn't wasted. :)

[DRY]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[RiftSkeleton]: https://github.com/jimbo00000/RiftSkeleton
[riftskel]: https://bitbucket.org/jimbo00000/riftskel
[RiftRay]: https://github.com/jimbo00000/RiftRay
[PaneScene]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/Scene/PaneScene.h
[RenderSeparately]: https://github.com/jimbo00000/RiftRay/blob/4a20e57a8bf07bf4ce2a17806fe1e7650de1a040/src/AppSkeleton/OVRSDK05AppSkeleton.cpp#L595
[shadertoy]: https://www.shadertoy.com/
[ORIA]: https://www.manning.com/books/oculus-rift-in-action
[AppSkel]: https://github.com/jimbo00000/RiftSkeleton/blob/master/src/AppSkeleton/AppSkeleton.h
[OVR08AppSkel]: https://github.com/jimbo00000/RiftSkeleton/blob/master/src/AppSkeleton/OVRSDK08AppSkeleton.h
[ovrLayerQuad_dox]: https://developer.oculus.com/doc/0.6.0.0-libovr/structovr_layer_quad.html#aa252260b430aea18052a106ed8b67818
[main_glfw_ovrsdk08.cpp]: https://bitbucket.org/jimbo00000/riftskel/src/920d0264658d195f31f91d0108f12834989ea0d3/src/main_glfw_ovrsdk08.cpp?at=master&fileviewer=file-view-default

[45]: https://bitbucket.org/jimbo00000/riftskel/src/920d0264658d195f31f91d0108f12834989ea0d3/src/main_glfw_ovrsdk08.cpp?at=master&fileviewer=file-view-default#main_glfw_ovrsdk08.cpp-45
[257]: https://bitbucket.org/jimbo00000/riftskel/src/920d0264658d195f31f91d0108f12834989ea0d3/src/main_glfw_ovrsdk08.cpp?at=master&fileviewer=file-view-default#main_glfw_ovrsdk08.cpp-257
[419]: https://bitbucket.org/jimbo00000/riftskel/src/920d0264658d195f31f91d0108f12834989ea0d3/src/main_glfw_ovrsdk08.cpp?at=master&fileviewer=file-view-default#main_glfw_ovrsdk08.cpp-510
[510]: https://bitbucket.org/jimbo00000/riftskel/src/920d0264658d195f31f91d0108f12834989ea0d3/src/main_glfw_ovrsdk08.cpp?at=master&fileviewer=file-view-default#main_glfw_ovrsdk08.cpp-419
[IScene]:https://github.com/jimbo00000/RiftSkeleton/blob/master/src/Scene/IScene.h
