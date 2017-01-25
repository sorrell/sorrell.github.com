---
layout: post
title: Finding non-matching line beginnings in Vim
tags: [Vim, Regex, Flatfile]
---
# {{ page.title }}

This one is going to the interwebs so I don't forget it... extraordinarily useful and so simple.

I sometimes have to dig through fairly lengthy flat files and find out why they won't load into our ETL process. 
In today's case, I had a 500,000+ line file which contained some unexpected line breaks.  Our files have a 
format of: four-digit company identifier, and then the rest of the record's data.  It looks like: `1234 MoreData...`.

So the problem was that I needed to find all instances of a line that didn't start with this company's
identifer, which we'll say is 1260 (So I needed ^1260).  To do that, I used this handy command:

```
:v/^1260/
```

Which brought back these nice results.  Notice above the blue 'Normal' bar is line 471596 and it starts with our
1260 ID.  Below the blue 'Normal' bar are the search results, and you can see that NONE of them start with 1260.

![](/images/vim-results.png "vimresults")


Now, I needed to save these results.  A simple copy and paste out of the Vim results buffer didn't preserve my 
tab delimiters and changed them into spaces.  So to output the results to a file you simply add ` .w! >> newfile.txt`
so that your final command looks like:

```
:v/^1260/ .w! >> newfile.txt
```

Final note, the `!` after the `.w` will create a new file.  If the file already exists, just use the `.w`.