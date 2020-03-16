---
layout: post
title: WSL and ZSH named directories
tags: [WSL, ZSH]
---
# {{ page.title }}

The more I use WSL, the more I love being able to seemlessly switch between Windows and Linux. And this is after more than a decade of exclusive Linux (Ubuntu/Mint) use. That said, I've found myself needing to tweak efficiencies into my new workflow. For instance, I now sometimes need to access files in my Windows file directories. But that's located all the way in `/mnt/c/Users/myWindowsUsername` and that's just too much to type all the time :)  

## Enter ZSH

I have been using zsh for years, and I love how powerful and customizable it is. If you haven't used it, you should really take a look at it with frameworks such as [https://ohmyz.sh/](Oh My ZSH) or [https://github.com/sorin-ionescu/prezto](Prezto) (I use Prezto). These frameworks can build a **ton** of efficiencies into your workflow on their own.

zsh on its own, however, has the capability itself to improve the command line workflow of accessing Windows files. To make navigation in zsh faster, we will utilize a feature called [http://zsh.sourceforge.net/Doc/Release/Expansion.html#Static-named-directories](Named Directories). zsh comes with a number of "builtins" and you can read about them with `man zshbuiltins`; The builtin we will use is called `hash`, which is an interface into two hash tables, the command hash table and the named directory hash table. 

The named directory hash table contains entries that get expanded by using the `~` (tilde) coupled with a string. The `~` on its own expands to your `$HOME` directory in Linux, which is usually located in `/home/myLinuxUsername`. So let's see how it works:

![create new entry](/images/wsl-zsh/create-entry.png)

Let's analyze what the lines above are doing:

  - `hash -d` without any proceeding arguments prints the contents of the specified hash table. When we specify the `-d` flag, we are specifying the named directory hash table. Notice how it's empty at first, and then the next time we print the contents, our `winhome` value is there.
  - `hash -d string=/path/` specifies to add the key of `string` with a value of `/path` to the named directory hash table. 
  - Note that zsh automatically changed the printout of our working directory from `c/Users/nicho` to `~winhome`... so it must be working!

So, how do we use this? 

![use new entry](/images/wsl-zsh/use-new-entry.png)

  - We cd to `/tmp` just for the visual sake of not being in our `~winhome` directory anymore - this should help visualize the "expansion" of the `winhome` hash table entry.
  - Notice how `~winhome` just works when we redirect the `echo` command to create the new `test.txt` file.

## Hash Table Entry Lifetimes

A quick word on the hash table entry lifetimes - the named directory hash table will clear as soon as you exit your shell. So if you made a mistake and misspelled an entry above, no big deal, just exit the shell.

Alternatively, you can clear the hash table with the `r` flag. Note again, we first specify the `-d` flag and how zsh automatically notices our `winhome` alias is gone.

![clear table](/images/wsl-zsh/clear-table.png)

So, how do we make these entries permanent? We can't technically make them permanent, but can repopulate the hash table automatically every time we log into a zsh shell by adding those commands to our `.zshrc` that is in our `$HOME` in Linux. And for the sake of laziness, I'm going to trim the `winhome` value down to just `w` to make accessing it even faster. I'll also add capital `W` with the same path just in case my finger is stuck on shift after using the tilde.

```sh
echo "hash -d W=/mnt/c/Users/nicho" >> ~/.zshrc
echo "hash -d w=/mnt/c/Users/nicho" >> ~/.zshrc
```

Now, when I log into my zsh, those values are already in the hash table.

![login](/images/wsl-zsh/login.png)