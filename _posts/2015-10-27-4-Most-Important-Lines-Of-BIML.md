---
layout: post
title: 4 Most Important Lines Of BIML
tags: [SSIS, BIML]
---
# {{ page.title }}

![A1](/images/2015-10-27-4-Most-Important-Lines-Of-BIML/4lines.png)

## tl;dr
You can see exactly where you BIML compile went bust.  [Clone the repository](https://github.com/sorrell/EnhanceDebugging) to see how it works, add the `Biml` folder from the repository to your projects, configure the `Config.cs`, and wrap every BIML file you have in lines 5-8 from the screenshot above. (A more modular solution comes about when we move to single-click-compiles, in a future episode...)


## BIML Debugging
You know you've cursed the BIML error messages.  You've fought with the maddeningly cryptic error messages, but wanted a better way.  Well, there they are - lines 5-8... not much, right?  Well, here's the thing though, those 4 lines let me pinpoint, without a shadow of a doubt, right where my compilation went wrong.  I get the elusive BIML "core dump," or at least all of the BIML until the BimlEngine puked.  If you're intrigued, let's dig in a little, but disclaimer:  this post isn't for the beginner BIMLer, as I don't explain the tiers, or inner workings... that's for a later post.

## Arrange  
First of all, let me say that I do **all** of my editing in Sublime Text, and that I've been working on a package that provides some nice highlighting and functionality... but more about that later.  I then `Alt+Tab` into Visual Studio to compile, so you'll be seeing screenshots from both.  I'm assuming you have BidsHelpers 1.7.0 installed so that you can compile via Visual Studio.

That said, we're going to work off of a project I originally created to showcase [how to get Error Column Names via SSIS Lineage IDs](http://sorrell.github.io/2015/09/14/Getting-SSIS-LineageIDs.html).  This time though, we'll work off of a full BIML base.  The project, when you load in Visual Studio, should show (2) packages:  

1. Handmade Package - as the name implies
2. BimlPackage - the package created by compiling the (5) BIML files

So the (5) BIML files are:

1. 1.Environment.biml - providing some global values
2. 2.FileFormat.biml - a file that outlines the Flat File formats
3. 3.Connections.biml - a file that specifies the Flat File Connections in the Connection Manager
4. 4.Packages.biml - a file that creates the `BimlPackage.dtsx`
5. SaveBiml.biml - a file that saves the fully compiled BIML

It should look like the following image in Visual Studio:

![A2](/images/2015-10-27-4-Most-Important-Lines-Of-BIML/vs.png)

## Assert
Since you're an advanced BIMLer, you'll notice that in `1.Environment.biml`, I'm bringing in my `Debug.csbiml` file.  And you're already wondering what a `csbiml` file is - that's for the "Best Practices" posts, but suffice it to say, that if we have a pure mixture of C# and BIML, I want to know about it.  And that's what `class directives` usually yield in my world.  

Now, out of the box, this BIML compiles if you:

1. Hold `Ctrl`
2. Select in this order: SaveBiml.biml, 4.Packages.biml, 3.Connections.biml, 2.FileFormat.biml, 1.Environment.biml (a pain that will be solved in a later post)

Notice, it creates the same package as the `HandmadePackage.dtsx`.  It should read the flat file just fine, and function according to design.  

Before you pass Go or collect $200 though, check out `4.Packages.biml` and notice that we've added those "4 Important Lines."  Those will come in handy when overengineer our package and get errors...

## Act
As you go along the path of BIML, you'll start wanting to make your code reusable and modular - yes, two very cutting edge concepts in programming :P  

When you do that, you'll start creating folder structures and reusable modules like those seen in this package.  The folders/scripts (SCR_GetLineageIds/SCMP_GetErrorColNameDesc) that help get the Error Column Name/Lineage IDs are two great examples of code that I will reuse forever.  And you'll see in `4.Packages.biml` that I make `include` calls to those falls, rather than inlining them.

That leads to a little problem:  when we overengineer our BIML, we start getting line numbers that don't quite line up.

Try this:  uncomment line 49 of `4.Packages.biml` so that it looks like this:

![A3](/images/2015-10-27-4-Most-Important-Lines-Of-BIML/errorDict.png)

Now you're referencing a `Dictionary` value that doesn't exist.  Here's the error I get from my BIML compile.

![A4](/images/2015-10-27-4-Most-Important-Lines-Of-BIML/bimlError.png)

Consider for the moment that this is a **GIANT** BIML file, and that you've automated and modularized your code... how does Line 121 correspond with the actual error?  Isn't our error on line 49?  Well, since you've wrapped your code in those 4 Important Lines, you can find out.

## Assert Again
Since we went kaput, let's take a look inside the `Biml\Utility\Debug\biml.log` file to see what went wrong:

``````````````
-- biml.log
20152710_205622|kfsqtpc1.0.cs|  Starting Package Creation.
20152710_205622|4.Packages.biml|  <ERROR>  |  Exception in BIML.  Dump written to Y:\Learning\EnhanceDebugging\EnhanceDebugging\Biml\Utility\Debug\Logs\20152710_205622_4.Packages.biml
``````````````

We can see that I started package creation, but I hit an exception... if you're using Sublime Text with the Open-Include package installed, simple click in the filename area, hit `Alt+D` and you'll be whisked to that file.  If you're not using Sublime Text, take a peak in the same directory for that file and open it.

Now, go to the very end of the file and you'll see that we puked on line 121, and that the (otherwise cryptic) error about a Dictionary key actually had to do with this line, which is 49 in our BIML file.

![A5](/images/2015-10-27-4-Most-Important-Lines-Of-BIML/line121.png)

Certainly you can see how helpful this would be by this point.

## Act++
Now, let's open the door that drove me nuts for days.  Selecting the files to compile in a seemingly logical order.

1. Hold `Shift`
2. Select 1.Environment.biml
3. Select SaveBiml.biml

At this point, all of the files should be selected, so that when you right click -> Generate SSIS Packages, you get this error:

![A6](/images/2015-10-27-4-Most-Important-Lines-Of-BIML/outOfOrder.png)

This was where I was frothing at the mouth, not understanding where I went wrong.  Not even understanding the error, really, since it just compiled a minute ago...

Now, open your `<Date>_<Time>_CompiledBiml.biml` file in that same Logs directory.  Can you find your Flat File Connections?  No?  Well, that's at least a clue at what's going wrong, and a fuller picture of what the error message is telling you - so get to work, and click those files in the *right* order!

## Summary
Ok, so now you're seeing how you can start using these tools, and how we can start structuring BIML projects with some sanity.  [Here is the repo for this post, clone and be merry.](https://github.com/sorrell/EnhanceDebugging)
