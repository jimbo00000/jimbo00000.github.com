---
layout: post
title:  "Cameras as Matrices"
date:   2016-10-23 21:44:00 -0500
categories: 3d opengl camera matrix modelview projection
---

There was a John Carmack tweet about this, but I was unable to find it. It went something  like:

> If you are going to use a camera class, don't bother trying to make it maximally general. Just pass in a matrix.

I adopted this strategy in all my latest projects that use LuaJIT to call into OpenGL, but I now think I didn't go far enough. RiftSkel's [on_lua_draw/render_for_one_eye][on_lua_draw] takes two matrices: modelview and projection. I have to admit that I had a cargo cult understanding of the model and view matrices and have conflated them for so long, I didn't even realize why you might want two separate matrices. Finally, after trying to implement specular lighting in VR, I now understand(a little bit better).


[on_lua_draw]: https://bitbucket.org/jimbo00000/riftskel/src/ac4dfa51f11ca07d1e0e34dd4e44fd447d7cf630/lua/scenebridge.lua?at=master&fileviewer=file-view-default#scenebridge.lua-74


### The Modelview, Model and View matrices

So the first time I tried to move my head around and examine the specular highlights on an object, it was not right. The highlights were stuck to the model! After the standard amount of flailing around [shotgun debugging][shotgun debugging], I had tried all possible permutations and discovered there was no way this approach was ever going to get it right. I needed another variable.

I now see that the presence of this new variable is due to the fact that the light exists in a space as well as the model, and moving the model does not move the light. Moving one's head relative to *world space* moves both the light and the model in *view space*. To move the model relative to the light, we need to tease apart the **model-view** matrix into its constituent parts: **model** and **view**.

[shotgun debugging]: https://en.wikipedia.org/wiki/Shotgun_debugging

### API Breaking change ahead?

So, this means I might have to break all the scene code that has been written by changing the signature of `render_for_one_eye`.

~~~lua
function scene.render_for_one_eye(view, proj)
~~~

I want to add the model matrix to that parameter list

~~~lua
function scene.render_for_one_eye(view, proj, model)
~~~



Since Lua is a stack machine, the value effectively defaults to `nil`. We are free to ignore it. Some developers find the stack-based architecture to be a mistake, and something does feel kind of loose about it. But it can be convenient.


~~~lua
function scene.render_for_one_eye(view, proj, model)
	if model == nil then
		mm.make_identity_matrix(model)
	end
	...
~~~

I thought while I was in there I'd change the function name to renderEye. Now maybe I won't bother.

#### Safeguarding camera controls against the change

By renaming a new entry point `renderEye`, main can check if the new entry point exists, otherwise calling the old one with a concatenated modelview matrix.


## Movement and Rotation UIs

Now that there's a camera class that can be swapped in and out for other cameras, different classes can specialize:

  - static camera
  - animated camera (uses `timestep`)
  - user-controlled camera (uses input functions)

Camera modules now call into glfw and SDL to get input - maybe there should be another layer on top: `cameraControl`, which itself receives input from the windowing library, and attaches to a simpler camera model which holds only position, orientation, velocity, etc.

Unity has provided me a model of UI controls. Clicking and dragging in the window with the rotation tool/state active produces one effect, and holding the alt key while clicking and dragging produces another. These two rotation schemes are called **Orbit** and **Flythrough**. The orbit scheme fits perfectly for a smallish object; something that can be held in your hand. The flythrough scheme fits for a large environment; something that you can observe from within.

#### Handsets in VR, Mouse on Screen

Way back when developing RiftSkeleton with the [Razer Hydra][Hydra], I added an extra input "channel" for hand manipulation. Mapping the hand channel to model matrix and the head to view matrix is a very satisfying fit. With a screen interface, it would be nice if the control scheme could provide input to both view and model matrices. A seemingly sensible way is to use left button for model rotation, and right button for view position.

[Hydra]: https://bitbucket.org/jimbo00000/riftskel/src/ac4dfa51f11ca07d1e0e34dd4e44fd447d7cf630/src/Scene/LuajitScene.cpp?at=master&fileviewer=file-view-default#LuajitScene.cpp-258

[ViveWand]: https://bitbucket.org/jimbo00000/riftskel/src/ac4dfa51f11ca07d1e0e34dd4e44fd447d7cf630/src/Scene/LuajitScene.cpp?at=master&fileviewer=file-view-default#LuajitScene.cpp-343



For the screen controls, we'll rotate about the origin and have assume a camera distance for the view matrix. We hope we'll be close enough to see everything, but not too close. So far, keeping objects to around a meter in size sounds like a reasonable guideline. It's right in the sweet spot of the float datatype, and we can intuitively scale objects and scenes as necessary when composing environments from multiple parts.


#### Left click Model, Right click View

We'd like to control the model and view matrices independently. 

This way we don't have to wonder about the context in which UI control is used and how big the scene extents are...

A modifier key could work for this - or the other mouse button. Since this program has no right-click menus, there's a whole mouse button available for use in partitioning input space. This way we can use modifiers in conjunction to do translation *or* rotation on click & drag, for both view and model. This seems like an intuitive and useful fit, so far.
