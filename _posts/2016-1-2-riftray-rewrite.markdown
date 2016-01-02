---
layout: post
title:  "RiftRay Refactor"
date:   2016-1-2 11:37:22 -0500
categories: opengl portable programming scripting
#published: false
---

Once RiftSkeleton was rewritten and clean for OVR SDK 0.8, [RiftRay][RiftRay] was ready to follow suit. I like to keep programs as small and understandable as possible, so I tend to introduce new features to the RiftSkeleton app first to see how they integrate in the simplest possible environment. Once that's done, I accept the extra cost of re-merging the code again into the "derived" project's codebase(in this case RiftRay), sometimes using [WinMerge][WinMerge] on both source directories. It works well enough on tiny projects like these, anyway.

### The Compositor

There were a few major changes to the API since 0.5, the most significant being the compositor. Though it is all closed source, I have the vague impression that this compositor is a separate process using Direct3D that shares texture resources with the user's OpenGL app via some unknown IPC method[^1]. GL and DX have opposite conventions for vertical texture coordinate. My guess is that this is the reason why the [old FBO scaling][old_viewport] went like this:

![Corner scale](/assets/bridge_corner_scale.png)

and the [new FBO scaling][tex_oddness] looks like this:

![Middle scale](/assets/bridge_middle_scale.png)

I like that Oculus named the flag for this `ovrLayerFlag_TextureOriginAtBottomLeft`, which states the meaning directly(as opposed to something like GL_coordinate_convention, which is conceptually once removed from the actual locations of texture coordinates in screen space). 

The code for this method is perhaps slightly less elegant than the previous way of just setting `glViewport` with scaled width and height, but only by a couple of operations. Happily, `glScissor` still operates the same way. I love how simple it is to dial in performance in a pure fragment-bound app like RiftRay by just calling that single function to set just a tiny piece of state, and how none of the rest of the code has to know about it.

### In-world quads

Coming along with the compositor is the `ovrLayerType_Quad` layer type, which takes care of multi-resolution "pane" rendering. My previous implementation of this was the [PaneScene][PaneScene] class, which I reused for both the [DashboardScene][DashboardScene](containing the [AntPane][AntPane]) and the [ShaderGalleryScene][ShaderGalleryScene], which drew all the thumbnails of shadertoys floating in space. The main difference between these two classes is that one of them stays put in your FOV when you move around using the keyboard(the "chassis" abstraction), one does not. Since the [OVRScene][OVRScene] class which shows the tracking camera bounds is also in "chair space", I pushed the variable all the way down to the IScene interface: [m_chassisLocalSpace][m_chassisLocalSpace]. But, this abstraction kinda leaked when it turned out we had to [construct both local and world space matrices][renderLocalWorld] to then pass into the [_RenderScenesToStereoBuffer][_RenderScenesToStereoBuffer] function, which became a catch-all of gross abstraction leakage. [^2] I stuck with it then because I like the simplicity of the [list of scenes][list of scenes] and just adding more objects into the world, letting them share a depth buffer. I'll be interested  to see if the [ovrLayerEyeFovDepth][ovrLayerEyeFovDepth] layer type will ever allow fragment depth testing.

RiftRay has some [ray-pane intersection code][intersect_code](backend courtesy of [GLM][GLM]) allowing you to use the HMD's gaze to move a mouse cursor on the quad. I was very happy to find out that Oculus again seems to have done the most correct thing here, centering the quad on the origin, exposing its pose and size and creating its vertices on the XY plane. It was easy to plug my own implementation into the new compositor. The end result of all this is better, more uniform performance and much less code in the user app. Less code is great, but I am also leery of ceding too much control to an opaque blob.

### Input support

The Sixense SDK support is out now in the name of simplicity. I see the HandPoses field in [ovrTrackingState][ovrTrackingState] and the [ovrInputState][ovrInputState], but have not tried using them, staying with [glfw joystick support][joystick()]. I also had a [Transformation class][txfmclass] used to concatenate manual translations and rotations from a 6DOF controller that got axed in favor of just [orienting the quad towards the HMD][eyerayholding].

I'd ultimately like to write as little code as possible to support as many devices as possible while keeping the project agnostic to headset manufacturer. With that goal in mind, giving API control to an interested party seems like a tactical blunder. Nevertheless, Oculus is still the only game in town. So for now, [the main source file][maincpp] is OVR SDK all the way. My plan is to just duplicate all that functionality with a new main for each new VR SDK.


[RiftRay]: https://github.com/jimbo00000/RiftRay
[WinMerge]: http://winmerge.org/?lang=en
[old_viewport]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/AppSkeleton/OVRSDK06AppSkeleton.cpp#L444
[tex_oddness]: https://github.com/jimbo00000/RiftRay/blob/master/src/main_glfw_ovrsdk08.cpp#L390

[PaneScene]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/Scene/PaneScene.h#L26
[DashboardScene]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/Scene/DashboardScene.h#L21
[AntPane]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/Scene/AntPane.h#L14
[ShaderGalleryScene]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/Scene/ShaderGalleryScene.h#L21
[m_chassisLocalSpace]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/Scene/IScene.h#L33
[OVRScene]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/Scene/OVRScene.h#L22
[renderLocalWorld]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/AppSkeleton/OVRSDK06AppSkeleton.cpp#L592
[_RenderScenesToStereoBuffer]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/AppSkeleton/OVRSDK06AppSkeleton.cpp#L372
[intersect_code]: https://github.com/jimbo00000/RiftRay/blob/master/src/Scene/HudQuad.cpp#L147
[GLM]: http://glm.g-truc.net/0.9.7/index.html
[list of scenes]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/AppSkeleton/AppSkeleton.h#L117
[ovrLayerEyeFovDepth]: https://developer.oculus.com/doc/0.8.0.0-libovr/structovr_layer_eye_fov_depth.html
[ovrTrackingState]: https://developer.oculus.com/doc/0.8.0.0-libovr/structovr_tracking_state.html
[ovrInputState]: https://developer.oculus.com/doc/0.7.0.0-libovr/structovr_input_state.html#details
[joystick()]:https://github.com/jimbo00000/RiftRay/blob/master/src/main_glfw_ovrsdk08.cpp#L793]
[maincpp]: https://github.com/jimbo00000/RiftRay/blob/master/src/main_glfw_ovrsdk08.cpp
[txfmclass]: https://github.com/jimbo00000/RiftRay/blob/2.0-OVRSDKv0.6/src/FlyingMouse/VirtualTrackball.h#L14
[eyerayholding]: https://github.com/jimbo00000/RiftRay/blob/master/src/Scene/HudQuad.cpp#L230

[^1]: I also suspect this has something to do with Win8's "tiles", visible in the taskbar previews and the *Metro* aka *Windows 8 Style UI!!* aka *Microsoft design language* start menu replacement. Making these eminently shareable sounds like good news for a VR desktop, but perhaps bad news for portability to other OSs, depending on how it's implemented. Also, are there security implications to this? Attackers may be able to see your windows.

[^2]: I remember a large refactoring to turn that from Scene-loop-inside-eye-loop to Eye-loop-inside-scene-loop in order to avoid the overhead of switching programs(at the cost of switching render targets). I still haven't measured the performance either way, so perhaps the effort was totally useless.

