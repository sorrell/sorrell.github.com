---
layout: post
title: Intermediate BIML Files
tags: [SSIS, BIML]
---
# {{ page.title }}

When I first started learning BIML, and compiling via BIDSHelper, I couldn't believe how much magic was taking place. What absolutely crushed me though was that I couldn't see any of the intermediate files, and debugging seemed *very* painful.  I don't yet have time to devote a good write-up to debugging, but I wanted to share a way to grab these intermediate files for yourself.  

Personally, I wanted to see some of the magic that the BIML Engine was performing.  And the power of [T4 Templating](http://www.hanselman.com/blog/T4TextTemplateTransformationToolkitCodeGenerationBestKeptVisualStudioSecret.aspx) is whole ball of wax itself - impressive.  A forewarning for this little project - you won't be able to make heads or tails of the random filenames, but they contain all of your Bimls (I will give some guidance on the filenames in a future post too).

Anyhow, if you're ready to see the intermediate files, just clone [my BimlWatcher repository](https://github.com/sorrell/BimlWatcher) and get to building.  From the command line, all you *should* need to do is run  
`BimlWatcher.exe outputDirectory`.  
Then, any build you do (via BIDSHelper) will have it's \*.cs files captured and placed in your `outputDirectory`.
