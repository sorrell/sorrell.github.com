---
layout: post
title: InteractivePrompt - A Generic Base REPL
tags: [REPL, Console, Prompt, Interactive]
---
# {{ page.title }}

While working on a yet to be released ETL tool (tentatively called ETLyte), I created a C# Console app to interact with the DLL.  I was pretty
sad to find that the Console doesn't provide any command history.  And there weren't any tools out there that did provide a command history.  Not just
that, there weren't any tools (that I could find, anyway) that provided a ready-to-go REPL / interactive prompt - which seemed crazy, because
there are many times when you just want to provide a quick CLI interaction, but you don't want to reinvent the handling of `WriteLine` and
`ReadLine`.

![image](http://cint.io/codecompletion.gif)


So I set out to solve the problem, and ended up creating InteractivePrompt, a small library written in C# for implementing your own interactive
CLI / REPL.  I'm still working on bugs and adding more features, but at the moment it does exactly what I want:

 - a command line up/down scrollable, editable history
 - word completions (that can be actively updated after each user command) via Tab
 - a simple lambda expression where you can handle all of your logic and return info to the user
 - standard shell navigation (left/right, Home/End, Ctrl+E, [Ctrl+H was subbed for Ctrl+A], Esc)
 - cross-platform - no Windows specific code/libs are involved

To show off the simplicity of the library, there is a small example [on my GitHub page](https://github.com/sorrell/InteractivePrompt), but I wanted
to provide a little bit fuller of an example of what can be done pretty simply.  So I created [Another CSharp Repl (ACSR)](https://github.com/sorrell/AnotherCSharpRepl).
It's a neat little program that shows how to update the completions list with each command, and also how simple it is to handle commands
coming from the user. The other fun part is intercepting the `add a using reference to the namespace` errors and trying to automatically add the
namespace... there are probably some fun bugs in there, but hey, it's a quick example. :)  Enjoy! 


![image](http://cint.io/acsrOverview.gif)