---
layout: guide
title: title
---
#LaTeX: \begin{Title Pages, Sections, and TOCs}

##How to create title pages
In all of our previous documents, we have included information like \title, \date, and \author.  All we need to do once we have those components in place is add a nice \maketitle command.

##Sectioning off your document
If we want our TOC to actually do work, we have to give it clues as to how our document is broken up.  We can break major areas into **sections**, **subsections** and paragraphs.  Additionally, we have more granularity if needed - see the list below as to how you can break up your document.
- \part
- \chapter (report style only)
- \section
- \subsection
- \subsubsection
- \paragraph
- \subparagraph
- \subsubparagraph (milstd and book-form styles only)
- \subsubsubparagraph (milstd and book-form styles only)
(Thanks to http://www.emerson.emory.edu/services/latex/latex_132.html for above list.)

##Table of Contents
This is an easy one... just add \tableofcontents to your document and provided you have sections/subsections, you will get a TOC!

<script src="https://gist.github.com/1867583.js?file=latex7.tex"></script>

Next: [11. Document Types](/latexPresentation/doctypes.html)