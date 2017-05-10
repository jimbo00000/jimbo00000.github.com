---
layout: post
title:  "Searching for a Text Editor in VR"
date:   2017-4-1 3:14:00 -0500
categories: vr gamedev programming editor texteditor
---


## The text editor

Perhaps the most important component of a self editing system is the editor. I agonized over this for more than a year - which editor should I choose? There are already such a glut of high-class, super-sophisticated editors out there that programmers have had holy wars over them since before I was born! I was sure that even attempting to write one of my own was foolish, arrogant. So I scoured github and the web at large for any and all text editing libraries, packages, apps, programs, plugins, anything that could be appropriated for use in a realtime graphical system.


### Grab full window's pixels

Surely the easiest way to punt on this whole problem of text editing is to just grab a whole window's pixels into a texture. Every redraw, pixels get updated. While this was *almost* good on a DK2 watching youtube, it's still not *really* what we want.

 - Pro: Use your favorite editor as is
 - Con: Pixel copy is slow, and scales with window size
 - Con: Not portable - too painful to maintain multiple paths


### Hack up an editor from scratch

I whipped up a font atlas renderer, vowing to keep it simple. A terminal was trivial to implement at this point, and a tantalizing reward! But as soon as line breaks are involved... I'm out. Just not willing to put in that level of complexity yet.


### Find an existing library

I came across [Scintilla][Scintilla]; probably the best fit for my needs. Daunted by its size, I kept looking. I never found anything smaller.

[ScintillaGL][ScintillaGL] - 5 years old with no updates. Good enough for Gargaj and 
[Bonzomatic][Bonzomatic]

[Scintilla]: http://www.scintilla.org/
[Bonzomatic]: https://github.com/Gargaj/Bonzomatic
[ScintillaGL]: https://github.com/sopyer/ScintillaGL


### Editor plugin

Sublime Text allows for python plugins - could we grab pixels from there?


### Port the whole terminal

Maybe if we could find a terminal emulator in pure OpenGL, we could get ncurses and vi and all kinds of wonderful stuff. emacs, ed, joe, nano... I was never really all that big into those, but I'm willing to take a dive in if it'll get us an editor in VR.

But, the libraries too complex. The standard is too complex! Coloring, controls codes, etc... I don't really want all that. I don't want to deal with getting all that to build on multiple platforms. *That* is more time than I am willing to invest.


### Write your own

So we're back around to write your own again. Having invested enough time researching workarounds for spending enough time on an OpenGL text editor, I will now invest my time into implementing the new editor.

I think I want something separate as a control terminal, with some kind of shell. I'd like to hide and show these 2 interface components at will. I'm going to hijack the tab and backtick(\`) characters for this purpose. They are spatially near the Escape key which I associate with a context-frame-popping action; a 'jumping out'.


#### Controls

A cursor and arrow keys to move around in the source. Source is a list of lines, each sent as a string to the font renderer. No unprintable characters; ASCII. Delete and Backspace. Control-S to save.

A [small function][mousepos] to translate mouse position into rows,columns lets us place the cursor woith the mouse. But selections? I don't think I want to dive down that well just yet. Let's see what we can do without them.

[mousepos]: https://bitbucket.org/jimbo00000/opengl-with-luajit/src/c77d5e600b00586b0b83d7c0dac940e3d5f504fc/scene/shader_editor.lua?at=master&fileviewer=file-view-default#shader_editor.lua-126

Recompiling small shader code on each keystroke turns out to be quick and *very* satisfying.


#### Error layer

Syntax highlighting, lexing, parsing, TODO
