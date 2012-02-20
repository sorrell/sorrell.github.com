---
layout: guide
title: overview
---
#LaTeX: \begin{Overview}

##History and Landscape
Donald Knuth is the father of the study of algorithms in computer science.  Legend has it that after his first book on this subject, he got an advance copy from his publisher who was using "new" digital technologies.  It must have looked like absolute hell, because Knuth spent the next 10+ years of life creating TeX, the typesetting language.  Like most programming languages, TeX inspired people to make higher-level (read easier) derivatives of the typesetting language.  For one, people wanted a PDF that they could pass around (as opposed to a .dvi or .ps file).  Thus, came pdfTeX engine.  People also wanted easier way to create documents, and a person name Lamport created the La(mport)TeX macro. 

What a source of confusion... engines, macros, and TeX - oh my!  When Knuth created TeX, it was an engine and macro (well, he had "default" kind of set of commands) in one - you spoke the language of the TeX compiler!  Of course, people got tired of that.  The following picture is an overview of TeX engines and macros.  To put it simply, here's how you can think of these terms:
![A1](/images/latex/EngMacro.jpg)
- Macros: the "language" you are using.  When we type a document in LaTeX format, we are using commands that are native to the LaTeX macro... think of the LaTeX macro as your personal interpreter to the TeX engine.  
- Engines: these are the TeX compilers.  They take your (La)TeX document and transform it into a PDF (in the case of pdfTeX).  Engines like XeTeX are particularly suited to exploiting the power of OpenType fonts, and yield extraordinarily beautiful documents.  Engines like LuaTeX can interpret the Lua programming language from within a .tex document!  Very helpful for charts/data processing!  

Next: [3. Getting Set Up](/latexPresentation/setup.html)