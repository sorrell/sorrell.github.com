---
layout: post
title: Getting Started With manim
tags: [manim, python]
---
# {{ page.title }}

I've been working on creating a Linux tutorial lately and I wanted to make the guide not so... drab. The 3blue1brown videos really have grown on me, and I wanted to do something with manim.

Unfortunately, it wasn't so easy to get started, as documentation is still in the early staged with manim. With some poking around, however, I was able to create this sequence, which I was satisfied with.

![manim gif](https://raw.githubusercontent.com/sorrell/sorrell.github.com/master/images/cli_prompt_example.gif)

My [first approach](https://gist.github.com/sorrell/b8ff936a6f1ab8c1446943487959b1a9) though felt really clunky with a lot of manual `play()` and `wait()` calls, where I was specifically choreographing the fading in and out of text and descriptions.

I knew that describing command line prompts/results would lead to a lot of this repetition, and would be very tedious, but I'd be following the same script of:

  1. Show the line
  2. Fade out everything but the first Mobject
  3. Fade in descriptions and subsequent Mobjects

So, I created a small library that condenses that clunky gist into a few simple lines of code, and called it [manim-linuxMobject](https://github.com/sorrell/manim-linuxMobject). Check it out! 