---
layout: post
title: Biml Revolution
tags: [SSIS, BIML]
---
# {{ page.title }}

In the next couple of weeks, I plan on releasing a series of blog posts that will,  hopefully, help revolutionize the way we all currently BIML.  

First, here's where I'm coming from:

* I don't have Mist to use as my IDE
* I am pained by having to click multiple files in the Visual Studio IDE to compile a package (I know, first world problems)
* That same "multiple file" pain creates a headache when trying to remember which files depend on each other
* There's no written methodology on how to structure a complex BIML project
* Debugging **is a NIGHTMARE**

So, what are the solutions?  Well, as I said, over the next couple of weeks we'll embark on making the BIML world a better place.  Here's a preview of what's next:

* The Four Most Important BIML Lines - A post about making debugging easier, and "core dumping" your broken BIML.
* Single Click Compilation - A post about creating BIML makefiles, so you can hand a project to an intern and simply say "Click Here."
* Edit in Sublime, Compile in VS - A post(s), and revealing of an open source project, about a Sublime Text theme I'm developing and setup that currently offers syntax highlighting, and relies on Open-Include to jump to files.  Hopefully to include the XSD autocomplete suggestions, and a hack of OmniSharp Server to read cs/biml files.
* BIML On The First Alarm - The beginning of an open source book about, what I believe to be, BIML best practices. How should folders be structured? How should files be named?  How do the directives work, and when should I use `code` versus `include`?  Plus, tons more.


I'm excited to get rolling on all of this - so look for the first post about comprehensive BIML debugging soon!
