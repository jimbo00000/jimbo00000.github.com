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


### The Model and View matrices

So the first time I tried to move my head around and examine the specular highlights on an object, it was not right. The highlights were stuck to the model! After the standard amount of flailing around [shotgun debugging][shotgun debugging], I had tried all possible permutations and discovered there was no way this approach was ever going to get it right. I needed another variable.

I now see that the presence of this new variable is due to the fact that the light exists in a space as well as the model, and moving the model does not move the light. Moving one's head relative to *world space* moves both the light and the model in *view space*. To move the model relative to the light, we need to tease apart the **model-view** matrix into its constituent parts: **model** and **view**.

[shotgun debugging]: https://en.wikipedia.org/wiki/Shotgun_debugging

### API Breaking change ahead

So, this means I have to break all the scene code that has been written by adding a new parameter to `render_for_one_eye`. Here's the current signature:

~~~lua
function scene.render_for_one_eye(view, proj)
~~~

Since Lua is a stack machine, we could just add the `model` parameter to the end of the list, and if it's not passed in, it will be `nil`.


~~~lua
function scene.render_for_one_eye(view, proj, model)
	if model == nil then
		mm.make_identity_matrix(model)
	end
	...
~~~

Something doesn't sit right with me about this, the order of the matrices should be model-view-projection. But then again, maybe the model matrix can be thought of as the optional one, and therefore makes sense in the last position as it can be omitted?

~~~lua
function scene.render_for_one_eye(model, view, proj)
	if proj == nil then
		-- Passed in only view, proj matrices
		view = model
		proj = view
		mm.make_identity_matrix(model)
	end
	...
~~~

Is this option ten times uglier?


## Rotation UIs

Using the Unity editor interface has also provided me with a bit of perspective on this issue. Clicking and dragging in the window with the rotation tool/state active produces one effect, and holding the alt key while clicking and dragging produces another. These two rotation schemes are called **Orbit** and **Flythrough**.

The orbit scheme fits perfectly for a smallish object; something that can be held in your hand. The flythrough scheme fits for a large environment; something that you can observe from within.


#### How to choose which UI?

Some *scenes* draw a small object and would be a perfect fit for the orbit scheme. These scene-objects could also be a really great fit for some hand control interaction, i.e. the [Vive wand][ViveWand], [Razer Hydra][Hydra] or Oculus Touch. But how will we know which scenes are appropriate for this? Where is the center and the bounding box for the ones that are?

[Hydra]: https://bitbucket.org/jimbo00000/riftskel/src/ac4dfa51f11ca07d1e0e34dd4e44fd447d7cf630/src/Scene/LuajitScene.cpp?at=master&fileviewer=file-view-default#LuajitScene.cpp-258

[ViveWand]: https://bitbucket.org/jimbo00000/riftskel/src/ac4dfa51f11ca07d1e0e34dd4e44fd447d7cf630/src/Scene/LuajitScene.cpp?at=master&fileviewer=file-view-default#LuajitScene.cpp-343

I suppose the best way to do this is explicitly. Maybe we can add some kind of metadata field indicating this information for the host app so it can make the best use of any available interactive resources(mouse, etc.) to display the thing. This puts an extra burden on the programmers on *both* sides.

#### Determine it implicitly

Is this possible? Can the spatial extents of a scene be determined merely by examining the code? I think this might be intractably difficult, considering what can be done in shader to any scene data. And it kinda completely falls apart for shadertoy-style scenes that eschew the vertex pipeline entirely.

For now, I think it's best to just keep scenes centered on the origin and of a size around 1m cubed(everything is in meters). There will probably be some metadata fields added to the "standard", and once that starts taking shape this will be there as well.

#### Metadata fields, Lua and generality

Lua seems like a good fit for this general metadata: the table structure lets us add pretty much whatever and gracefully ignore it if any field is not present. It felt natural and easy for the [Vive wand][ViveWand] integration, even if it's not the cleanest thing in the world. 
