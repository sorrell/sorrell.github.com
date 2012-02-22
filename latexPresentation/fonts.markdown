---
layout: guide
title: fonts
---
#LaTeX: \begin{Fonts, Spaces, Line Breaks, and Landscape}

##Bold, Italics, and Typewriter
![A1](/images/latex/fonts.png)  
As you can see, you have two choices, the command version or the declarative.  Either works.  Additionally, you can use \emph for italics. See Gist below for implementation.

##Coloring your text
In LaTeX, we have to use special commands to color our text (we also have use a package called "color").  It's a fairly simple process, though, as just use the \textcolor{our-color-here}{text-goes-here} command.  See the Gist below for an example.

##Sizes
Font sizes aren't as simple as a toolbar-click anymore.  They aren't at all difficult to change though.  If we want to change the default font size, we can add it as an argument to the \documentclass command.  To change the size of a font inline, there are commands that we can add to the text (see the Gist code below).

##Font family
While the default font (Computer Modern) is a very nice looking font, sometime you want to change it.  To change it to a sans-serif font, try \renewcommand{\familydefault}{\sfdefault}.  Additionally, [check out this link](http://www.cl.cam.ac.uk/~rf10/pstex/latexcommands.htm).

##Margins
The easiest way to change the margins in a LaTeX doc is to use the "geometry" package.  Again, see code below.

##Landscape mode
A common feature you may want to use is the "Landscape" mode, when you need your page to be wider than it is tall.  In that case, we add the [landscape] argument to the \documentclass command.  See the Gist below.

<script src="https://gist.github.com/1862016.js?file=latex6.tex"></script>

##Line Breaks and Spaces
- Spaces are denoted with ~
- Line breaks are denoted with two backslashes - \ \

Next: [10. Title Pages, Sections and TOC](/latexPresentation/title.html)