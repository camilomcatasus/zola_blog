+++
title = "CamTerm"
[taxonomies]
tags = ["Rust", "WASM", "Trunk", "Macroquad"]
+++

[CamTerm](https://term.camilomcatasus.dev) is a pseudo terminal renderer that runs in WASM. 

## How

Every about section under **projects** as well as my resume are written in markdown.
The markdown is parsed and rendered to a buffer of chars which is then drawn to the screen.
The same markdown files are used to make a more searchable and disability friendly blog, 
that can be reached by pressing the button underneath any article.

Most of the website is built in **Rust** using *macroquad* as a sort of game engine. 
I initially tried to get a project built with *ratatui.rs* and *xterm.js* to work but the complexity overwhelmed me eventually.
So I built a pseudo terminal renderer in macroquad, using its simple api for drawing to the screen. 

I'm using trunk to serve all my static files.

## Issues

After trunk compiles rust to wasm, it adds a nonce to the end of the file name. 
This makes it difficult to target the wasm from javascript, which is needed for macroquad.
To resolve this I had to create a [utility](https://github.com/camilomcatasus/trunk_repl) that gets the name of the wasm file and rewrites 
the javascript in the index.html to point to the wasm file. 
