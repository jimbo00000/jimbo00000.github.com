---
layout: post
title:  "VR Scripting"
date:   2016-1-9 11:37:22 -0500
categories: opengl portable programming vr scripting
#published: false
---

The [news][live] of [VRScript][vrscript] brought the promise of fast, cross-platform experience prototyping and sharing using a higher-level language to interface with a VR application's framework. This is exciting for anyone who starts new VR projects from scratch often, as it lets developers crank out small experiments with as little effort as possible.

[live]: https://www.reddit.com/live/vn5n3360s4nz
[vrscript]: https://www.reddit.com/r/vrscript/

## VR Scripting using LuaJIT

Although no VRScript releases or announcements have been made, you can enjoy the low-friction rapid prototyping workflow today with [RiftSkel][RiftSkel] on desktop and DK2. It has a similar scene system using [LuaJIT][luajit] as an embedded language. There's a [proof-of-concept minimal fps game engine][gamescene] working in the latest master that is more or less functionally equivalent to the [targets and throwing stars demo][throwingstarsdemo] from John Carmack's VRScript presentation. This demo:

 - [takes keyboard events to trigger a "shot"][shootfunction]
 - [loads][sampleload] and [plays sounds on shot events][shootfunctionsound]
 - uses [head orientation][setoriginmat] to [aim shots][txshot](shoot from eye)
 - [advances simulation state every timestep][simstate]
 - [does simple collision detection of shots against targets][shotinter]
 - [destroys targets when hit by a shot][shotrem]

[throwingstarsdemo]: https://youtu.be/rMItsZq_n20?t=31m29s

[shootfunction]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default#gamescene.lua-273
[shootfunctionsound]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default#gamescene.lua-270
[setoriginmat]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default#gamescene.lua-261
[shotinter]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default#gamescene.lua-230
[shotrem]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default#gamescene.lua-240
[txshot]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default#gamescene.lua-254
[sampleload]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default#gamescene.lua-144
[simstate]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default#gamescene.lua-219


and does it all with untextured cubes. Here's a somewhat lackluster screenshot:
![game shot](/assets/gamecap2.PNG)

[RiftSkel]: https://bitbucket.org/jimbo00000/riftskel/
[luajit]: http://luajit.org/
[gamescene]: https://bitbucket.org/jimbo00000/riftskel/src/dfdad31a5df7e1fe434745be9d21866af2bb37b7/lua/scene/gamescene.lua?at=master&fileviewer=file-view-default

You can use just about any OS you want. You can play in the now old-fashioned window-on-a-screen format, or strap on an HMD and shoot from your eye like [Cyclops][Cyclops]. All of the code is [MIT licensed][mitlic], so feel free to use it, modify it, build something with it and charge for it - whatever you want. There's really not much to it. Aside from [LuaJIT's awesome performance][luajit_perf], this is the platform's core strength.

[Cyclops]: https://www.google.com/search?q=cyclops+x+men&safe=off&espv=2&biw=1122&bih=706&source=lnms&tbm=isch&sa=X&ved=0ahUKEwic_Krw3J3KAhUGyT4KHa4AAw0Q_AUIBigB#imgrc=qXsW1Gi8GVjPSM%3A
[luajit_perf]: http://luajit.org/performance.html

Just plug into the following interface, outlined in [scenebridge.lua][scenebridge]:

~~~
function on_lua_initgl()
function on_lua_exitgl()
function on_lua_draw(pModelviewMatrix, pProjectionMatrix)
function on_lua_timestep(absTime, dt)
function on_lua_keypressed(key)
~~~

[scenebridge]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/lua/scenebridge.lua?at=master&fileviewer=file-view-default


[mitlic]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/LICENSE?at=master&fileviewer=file-view-default

The [gamescene][gamescene] module written in LuaJIT can be [refreshed and reloaded][refresh] within the app in about 30ms. This amounts to one dropped frame, and is only noticeable to me in the HMD if I press the F5 key repeatedly. Init time will increase for anything that has complexity in its init function, but a base cost of only 30ms seems entirely acceptable, indeed better than I had hoped.

If any part of the code throws an error, the error message will be output to the console and the display will be all black. Make a change to the code and press F5 to refresh.

[refresh]: https://bitbucket.org/jimbo00000/riftskel/src/11fa3fab7258ff4a3b8d7fc7608abfbc11fe515d/src/main_glfw_ovrsdk08.cpp?at=master&fileviewer=file-view-default#main_glfw_ovrsdk08.cpp-436

## Features

As of now, RiftSkel is lacking features that VRScript has been demonstrated to have. These should be straightforward to implement and may even benefit from being written in a higher-level language.



|               | VRScript      | RiftSkel  |
| ------------- |:-------------:| -----:|
| Language      | Racket        | LuaJIT |
| Rapid Prototyping | yes | yes |
| FFI      | yes      |   yes |
| OpenGL   | yes, with<br>geometry wrapper      |   yes |
| Sound loading/playing | yes | yes |
| Platform | Android | Desktop |
| Publishing | yes | |
| Model loading | yes | |
| Image loading | yes | |
| In-world text display | yes | |
| SDF Fonts | yes | |
| Panoramic stereo display | yes | |
| | | |
  
The [FFI(foreign function interface)][ffi] is about the most powerful aspect of using a scripting language - it can link with native libraries in .DLL, .so, or .dylib formats, allowing you to do anything a native app can do.[^1] RiftSkel uses the FFI to call into [OpenGL][opengl] for full graphics capability and [Un4seen's BASS][bass] library for audio.[^2]

[ffi]: http://luajit.org/ext_ffi.html
[opengl]: https://www.khronos.org/
[bass]: http://www.un4seen.com/

Racket has the DrRacket IDE. The excellent and open source [ZeroBrane Studio][zbstudio] can be used to debug LuaJIT in standalone form; not yet sure about embedded code. For debugging, I have a [standalone project available][ogl-luajit] to take the same scene interface in pure LuaJIT.

[zbstudio]: http://studio.zerobrane.com/
[ogl-luajit]: https://bitbucket.org/jimbo00000/opengl-with-luajit

### Future Feature Plan

#### Model Loading
An obj model loader should be easy enough to implement using [Lua Binary I/O][lua_binaryio]. [John mentioned a custom model format][modelformat] exported from [FBX][FBX]. I can't see why any format would *not* be loadable by Lua and hence by LuaJIT - just pull it in as binary and call `glGenBuffers` and `glBufferData`.

[modelformat]: https://youtu.be/rMItsZq_n20?t=10m50s
[lua_binaryio]: http://www.lua.org/pil/21.2.2.html
[FBX]: https://en.wikipedia.org/wiki/FBX

#### Texture loading
There are image loading libraries for Lua, and there is also the FFI. As with model data, I don't foresee any difficulties here. Use `glTexImage2D` and friends.

#### Panoramic Stereo
There is [some free C++ code][OmniPano] to do this that could be ported to LuaJIT. The gist of it isn't too tough: [construct some cylindrical geometry][cylindergeom] and draw it [textured differently for each eye][texture_it]. I'm personally not wild about omnistereo imagery as it'll never work when you tilt or move your head, but it is a simple way to fill an environment with pixel data. The OVR implementation probably does something very smart with multi-resolution or partially-resident textures. The mobile SDK may have that source available.

[cylindergeom]: https://github.com/jimbo00000/OmniPano/blob/master/src/Panorama/PanoramaCylinder.cpp#L289
[texture_it]: https://github.com/jimbo00000/OmniPano/blob/master/src/Panorama/PanoramaCylinder.cpp#L363


[OmniPano]: https://github.com/jimbo00000/OmniPano

#### In-world Text Display

I haven't yet found the kind of lightweight, turn-key solution I'd like for this. There appear to be some handy FreeType SDF tools in the `Tools/` directory of the 1.0.0 mobile SDK. A solid, fast implementation of this would be necessary for the popular yet [disadvised][terrible_idea] [HMD Programming][hmdprogramming] technique, but would not be sufficient for it. This will have to be revisited(hey @brianpeiris).

[terrible_idea]: https://www.youtube.com/watch?v=rMItsZq_n20&feature=youtu.be&t=20m6s
[hmdprogramming]: https://www.reddit.com/r/hmdprogramming

#### Android Build
RiftSkel works right now only on desktop OSs: Windows, Linux, MacOS, PCBSD, Solaris. OVR SDK is supported where available(0.8 on Windows, 0.5 on Linux, MacOS). BASS is optional, used where available. An Android port should be entirely possible, as LuaJIT is known to work on Android(and many other platforms). There are, in fact, some [very well-known developement houses that use LuaJIT][otoy_post].

[otoy_post]: https://www.reddit.com/r/oculus/comments/3t34qc/thoughts_on_vrscript_from_tony_parisi/cx2v2h3


Comparing the scene interface to the one found in the mobile SDK, everything looks similar enough that plugging in to the mobile SDK should be straightforward:
`ovr_sdk_mobile_1.0.0.0.zip/VrSamples/Native/VrCubeWorld_Framework/Src/VrCubeWorld_Framework.cpp`

~~~
virtual void		OneTimeInit( const char * fromPackage, const char * launchIntentJSON, const char * launchIntentURI );
virtual void		OneTimeShutdown();
virtual bool		OnKeyEvent( const int keyCode, const int repeatCount, const KeyEventType eventType );
virtual Matrix4f	Frame( const VrFrame & vrFrame );
virtual Matrix4f	DrawEyeView( const int eye, const float fovDegreesX, const float fovDegreesY, ovrFrameParms & frameParms );

// Passed to an application each frame.
class VrFrame
{
	double			PredictedDisplayTimeInSeconds;
	float			DeltaSeconds;
	long long		FrameNumber;
	ovrTracking		Tracking;
	VrInput			Input;
	VrDeviceStatus	DeviceStatus;
	mutable SystemActivitiesAppEventList_t	* AppEvents;
};
~~~




#### Publishing and Network Connectivity
This is where it gets most interesting. There is no software support for this in RiftSkel yet, but I think the implementation could *possibly* be straightforward, even easy. The procedure for sharing a scene would look something like:

 - Create a scene from a template as a new directory under `scenes/` next to the RiftSkel executable
 - Include all models, textures, sounds, libs, and other resources in the directory
 - Write the LuaJIT code to handle scene's init, destruction, display, timestep and input
 - Test and debug to taste
 - Zip up the whole directory
 - Send it to a friend

Connectivity could be handled very nicely by [RakNet][raknet], a lovely networking library quietly acquired and [open-sourced][raknet_license] by Oculus over a year ago. It handles voice chat, NAT punchthrough, and peer-to-peer connection. It also does TCP, so writing some kind of server for content should be entirely possible. Discovery of scenes and other users would greatly benefit from a centralized server somewhere with search interface, or you could just hop into a chat room and share your IP address.

Of course it will be a massive security breach to run untested third-party code on your machine. My guess is that security is the reason why VRScript hasn't been released. My advice: be prepared to reinstall your entire OS from scratch at any time. And watch out for bootsector viruses. Does the Rift have firmware that can be flashed?

[raknet]: https://github.com/OculusVR/RakNet
[raknet_license]: https://github.com/OculusVR/RakNet/blob/master/LICENSE

## Fun

Developing using a scripting language is more fun. It's easier to just hit a button and reload, rather than recompiling and running every time. Lua is a fun language - small, easy to use, and fast. LuaJIT is even faster, and more powerful.

So I hope you try it out. This is how I want to do development from now on.

##### Footnotes:

[^1]: As long as there is a dynamic library supplied: the FFI can't link into static libs(as far as I know).
[^2]: A very fully-featured library that has enjoyed some popularity in the demoscene, BASS has 3D audio capability.

