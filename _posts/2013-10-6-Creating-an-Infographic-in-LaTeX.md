---
layout: post
title: Creating an Infographic in LaTeX
---
#{{ page.title }}

This is a post I've been meaning to get to for a while.  Since the beginning of 2012, in fact.  And now that I've tried to see if my reference link to census.gov works, it turns out that it *temporarily* doesn't because of the government shutdown... here it is though ( [pdf here](/files/infographic.pdf "infographic pdf") ).

![infographic](/images/infographic.png "infographic")

In my Technical & Scientific Writing class, I had to create an infographic.  [Here is the source code.](https://github.com/sorrell/latexInfographic) Being the LaTeX dork that I am, I searched for LaTeX infographics but couldn't find any.  And there's a good reason for that:  **it's a pain in the ass to make one in LaTeX.**

I did it anyway though, and I based it on [this interaction diagram](http://www.texample.net/tikz/examples/interaction-diagram/ "interaction diagram") (see the very end).  You have to build the ``infographicTop.tex`` first so that it produces a PDF that the ``infographicFinal.tex`` can then go an use as a layer.

**Summary:** If you're familiar with LaTeX, then you'll find the source helpful.  If you're not that comfortable with LaTeX, you can modify the source by plugging in your own pics/titles/stats, but things can get messy quickly!

