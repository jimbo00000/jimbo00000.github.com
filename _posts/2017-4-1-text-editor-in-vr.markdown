---
layout: post
title:  "Searching for a Text Editor in VR"
date:   2017-4-1 3:14:00 -0500
categories: vr gamedev programming editor texteditor
---


## The text editor

Perhaps the most important component of a self editing system is the editor. I agonized over this for more than a year - which editor should I choose? There are already such a glut of high-class, super-sophisticated editors out there that programmers have had holy wars over them since before I was born! I was sure that even attempting to write one of my own was foolish, arrogant. So I scoured github and the web at large for any and all text editing libraries, packages, apps, programs, plugins, anything that could be appropriated for use in a realtime graphical system.


### Grab full window's pixels

Use your favorite editor as is
Pixel copy is slow, and scales with window size
Not portable - too painful to maintain multiple paths



### Hack an editor from source

I came across Scintilla; probably the best fit for my needs. Daunted by its size, I kept looking. I never found anything smaller.

ScintillaGL - 5 years old with no updates. Good enough for Gargaj and Bonzomatic


https://github.com/Gargaj/Bonzomatic


http://www.scintilla.org/
https://github.com/sopyer/ScintillaGL


### Port the whole terminal

VT system, ncurses
use vi, emacs, ed, joe, pico...
Libraries too complex
Standard too complex!
Coloring, controls codes, etc...
  - don't really want all that



### Write your own


Controls
Key values/layouts

Mouse clicks too

Error layer

Recompile on each keystroke

Hide/show and the "terminal"
