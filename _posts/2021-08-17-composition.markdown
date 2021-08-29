---
layout: post
title:  "Composition"
date:   2021-8-17 3:14:00 -0500
categories: demo prod programming graphics
---


## Control

[@party 2018](http://atparty-demoscene.net/) saw the release of [Cratered](https://www.pouet.net/prod.php?which=76558), what I consider to be my best demo work, to decent audience acclaim.
This post will sketch out my design process for the demo.

 This post is, of course, 3 years late. I meant to post it back then but got carried away with other projects. 2021 got me feeling the demoscene vibes again, so I dusted off foguete and gave it a whirl, picking up where I left off.


## Hobby Development

Since this is all unpaid work, it pays to keep things as easy as possible. I want to be able to pick this code up as quickly as possible after not having looked at it for 3 years, and having forgotten most of the details of how it works. Minimizing the time spent reading the code and trying to understand it is paramount.

One thing I've noticed about being forced to read and understand old code is that the longer the file, the more unpleasant the process is. I've found that once a module crosses the 1000 LOC line, it gets dramatically less fun to work with. Modules below 500 LOC, by contrast, can be a pleasure - they are easy to read, understand and change. I suspect the unpleasantness is super-linearly related to LOC; perhaps quadratic or worse.


Another thing I've noticed is that *working* code is hundreds of times easier to debug and maintain than *broken* code. If I can at least see the thing run, I'll know whether or not a change breaks it. To this end, I've tried to make heavy use of composition in the cratered demo, treating scene modules as building blocks. Each one is individually testable/viewable, and then included into a higher-level scene. The leaf nodes are then nothing to worry about provided they're kept small. They can be developed and tested in isolation, and may even be put aside as "good enough" with incomplete inplementations as mockups in an unfinished prototype.



## Cratered Design

Here's a dependency graph of most of the modules used in cratered:
![cratered_composition](/assets/cratered_composition.png)

My intention was to break up functionality into manageable file units. Something like OOP - each lua file is basically a subclass of scene(with initGL, renderEye and update functions).


- **cubemesh**: 426 LOC - *No drawing*.  6 subdivided square grids, and a compute shader to re-calc normals.

- **gridcube**: 310 LOC. Lighting and drawing concerns for cubemesh.

- **moon**: 382 LOC - *No drawing*. Compute shaders to perturb gridcube's verts by a little bit, and form craters by perturbing verts locally.

- **ejecta**: 722 LOC. GPU particle system of instanced obj files(which ended up as simple cubes anyway). Compute shader to timestamp instance positions and orientations.

- **coma**: 258 LOC. Another derivative of gridcube, this one simply deforms the sphere into a football shape and draws a plasma effect in fs. This one, iirc, was kind of rushed and could have benefitted from some cleanup and refactoring, but deadline was nearing.

- **comet**: 90 LOC. A simple container holding **coma** and **word**. Does the billboarding computation before displaying the word at the core of the comet.

- **comets_in_space**: 113 LOC. A simple container scene to lay out 3 comets against a starfield background.

- **meteorite**: 378 LOC. All the logic to tie the comets, moon and ejecta together. Even has some keyboard functions to type in strings and launch them at the planet with a click or enter press.

- **space_scene**: 161 LOC. Includes 3 scenes, and some dubious hacky logic for resetting state from the synctracker.[^1]


### Upsides
Looks like it got about 6 modules high or so, which is pretty cool. I have the impression that we can keep going up almost indefinitely - I've yet to see where this breaks down. Most of the modules are deliciously small, which keeps them fun to use and re-use.
Modules are also built to operate standalone as much as possible, effectively acting as their own unit tests.

### Downsides
There is some coupling between modules, with modules "reaching in" and poking around with the data in the modules they include. Some API/interface is worth adding in the child modules to ease the job of whatever's trying to drive it.

I'm seeing now that there seems to be unnecessary duplication, particularly starfield. I'd rather have just one of those, and add it to the end of a draw list, with some kind of top-level scene container holding that list... And I've always felt that "container" scenes should be codeless, somehow, and consist of just a simple list of scenes.


### Future Work
- Some kind of data-driven or node-based GUI for making scene compositions
    - Data channels as edges in the graph
- Extract coupling logic into dedicated modules

### Conclusion
Composition eases the sporadic development as long as components are kept magageably small.


[^1]: Looking back, I'd much rather not have evolving state in a sync'ed demo. Can't see any other way to do the cratering deformations, though.
