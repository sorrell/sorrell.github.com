---
layout: post
title: Dreamhost With Jekyll (and moodle)
tags: [Dreamhost, Jekyll, rvm, ruby, moodle, git]
---
#{{ page.title }}

I am doing some web hosting on Dreamhost, and I'm using their cheap, cheap, shared hosting plan.  This means that I'm given a  user slot on one of their shared (virtual?) machines.  That means I'm rootless, and in the shell world, that sucks!

Also I wanted to install Jekyll, Pygments, and do that with an up-to-date version of Ruby.  Now, I'm sure the following instructions will be outdated within a year and a half ([like these](http://williamting.com/posts/2012/04/02/set-up-ruby-on-rails-on-dreamhost/)), but here goes.  From here on out **LM> is Local Machine** and **RM> is Remote Machine** when you see the shell snippets.

### 0. SSH into your Dreamhost shell
So from your terminal that should look like:

    LM> ssh myUser@myDomain.com

If you haven't set up ``authorized_keys`` yet, don't worry, we'll get to that. 

### 1. Installing RVM
Installling rvm *should* be trivial:  

     RM> bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)  

### 2. Install a non-antiqued Ruby
This is where it got tricky for me.  I encountered a problem after trying to install ruby-2.0.0.

    RM> rvm install ruby-2.0.0
    RM> ERROR: Missing required packages: libreadline6-dev, libyaml-dev, automake, libtool, libffi-dev

Grr.  This was maddening.  Finally I hunted down some information that [recommends](http://rvm.io/packages) that you **NOT** do the following because it's not a very thoroughly tested option.  Oh well, I prefer working at this point, so onward:

    RM> rvm autolibs rvm_pkg

Now you should be able to install a newer Ruby!

    RM> rvm install ruby-2.0.0

**NOTE:** If this looks like it's just hanging (on libtool maybe), check your logfile - it's probably not hanging, it's just waiting for you to **press a key to continue.**

Now, after installation, verify your Ruby version with ``ruby -v`` and then source your *rc file (.bashrc/.zshrc) with ``source ~/.bashrc``.  

### 3.  Install Jekyll with Pygments
This seemed to be the easiest part of the install

    {% highlight bash %}
    RM> gem install jekyll
    RM> mkdir ~/soft && mkdir ~/packages && mkdir ~/packages/lib && ~/packages/lib/python && cd ~/soft
    RM> echo 'export PYTHONPATH="$HOME/packages/lib/python:/usr/lib/python2.6"' >> ~/.bash_profile
    RM> source ~/.bash_profile
    RM> wget http://pypi.python.org/packages/source/P/Pygments/Pygments-1.6.tar.gz
    RM> tar -xvzf Pygments-1.6.tar.gz
    RM> cd Pygments-1.6
    RM> python setup.py install --home=$HOME/packages  
    {% endhighlight %}

### 4. Setup public key SSH login on Dreamhost (authorized_keys)
This is just from the Dreamhost wiki.  You should probably verify that the correct key is cat'd into your authorized_keys file after you do it...

    {% highlight bash %}
    LM> scp ~/.ssh/id_rsa.pub user@example.com:~/

    RM> cat id_rsa.pub >> .ssh/authorized_keys
    RM> rm id_rsa.pub
    RM> chmod go-w ~
    RM> chmod 700 ~/.ssh
    RM> chmod 600 ~/.ssh/authorized_keys
    {% endhighlight %}

### 5. Create your Git repos for Jekyll 
To do that, we have to set up our repos locally and remotely (thanks to [this blog](http://blog.zerosum.org/2010/11/01/pure-git-deploy-workflow.html))... also don't forget to modify the ``myUser`` parts below!

    {% highlight bash %}
    LM> mkdir repo && cd repo
    LM> git init 
    LM> git add * 
    LM> git commit -m "Created my repo" 
    
    RM> mkdir gitprojects && cd gitprojects
    RM> mkdir repo.git && cd repo.git
    RM> git --bare init
    
    LM> git remote add origin ssh://myUser@myDomain.com/home/myUser/gitprojects/repo.git
    LM> git push --all origin
    {% endhighlight %}

### 6. Setup Git to Run Jekyll when it is pushed to (auto-publishing FTW!)
This is the part that I really wanted.  I **love** how GitHub Pages allows you to just push a markdown post to the server and **boom**,  it's published as a blog post.  Again, these tips are thanks to the blog linked to in step 5.  We need to navigate to ``/home/myUser/gitprojects/repo.git/hooks`` and edit the ``post-receive`` file.  Here's what you should put in there (you may also need to ``chmod ug+x`` according to it and mods by me)

   {% highlight bash %}
    #!/bin/sh
    GIT_REPO=$HOME/gitprojects/repo.git
    TMP_GIT_CLONE=$HOME/tmp/myDomain.com
    PUBLIC_WWW=$HOME/myDomain.com

    git clone $GIT_REPO $TMP_GIT_CLONE
    jekyll build --source $TMP_GIT_CLONE --destination $PUBLIC_WWW
    cd ~ 
    rm -rf $TMP_GIT_CLONE
    exit
    {% endhighlight %}


That's it!  You made it!  And hopefully this saves you some of the headache I went through!