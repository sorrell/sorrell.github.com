---
layout: post
title: The Overdue Biml Updates
tags: [SSIS, BIML]
---
# {{ page.title }}

I know it's been way too long since I promised these changes, but I've been busy with other non-Biml related things.  That said I wanted to give a couple of updates...

## The Biml Makefile
I meant to release this months ago, and I have polished it a whole lot, but it's a great proof of concept on how to make a Biml makefile without violating any terms or agreements.  It hooks into the active Visual Studio instance (yes, totally hacktastic) to compile your Biml...

That said, it makes my life a WHOLE lot easier when dealing with multiple includes/tiers/dependencies and trying to organize my Biml life.

[Have a look at it here.](https://github.com/sorrell/MakefileBiml)

## The Biml syntax highlighter
Visual Studio, as you know, is so awful for syntax highlighting that it hurts.  I've created this crusty, big time work in progress, Sublime Text syntax theme.  It's not yet an official package, but I'm slowly working toward that.  For now you can install it manually.

![subiml](/images/subiml.png "subiml")

[You can check out that repo here.](https://github.com/sorrell/subiml)