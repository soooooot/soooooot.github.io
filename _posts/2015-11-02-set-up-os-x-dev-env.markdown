---
layout: post
title:  "Set up my dev enviroment on OS X"
date:   2015-11-02 21:41:54 +0800
categories: env dotfiles
---

My target is to clone my ubuntu develop environment to my new Macbook. 


### dotfiles
previously, I created a repo called `configrations` just for share my .vimrc code. and there's only a file, what it can be is a backup file, and maybe can share with others. Everytime I want to change my enviroment, I need to copy that file down, and rename it to ~/.vimrc manually. 
And everytime I changed the files, I need to copy the ~/.vimrc to my repo file, and then commit it.
It's so... ineffective and infunctionable.

but now, please let me introduce you a useful repository : [dotfiles][dotfiles] .
actually, it's not a repository, it's kind of style or pattern for us to backup, restore, sync, learn, share our .config files.

First time I got intouch with `dotfiles` is the time I watch the tutorial video: [vim + tmux - OMG!Code][vim-tmux-omg]. After I traversed the recommmand repo in the [dotfiles unofficial site][dotfiles], but I go back to fork [nicknisi's version][nicknisi-dotfiles]. First impressions are firmly entrenched, right? 


Here's [my dotfiles][soooooot-dotfiles], but now it's extremely simple, I will keep on completing it in these days and future.


### ctrl-r back-reverse-search not working

Cut it short, and make a quick view: it's something about ctrl-r on iTerm2/Oh-my-zsh/`set -o vi`.

This time, I try to switch to zsh, since there's powerful [oh-my-zsh][oh-my-zsh] project there. It's so cool when you cd a git directory, it will automatically propmt which branch you are in. I'm pretty enjoy in the feature, so that it's so hard for me to commit to a wrong branch.

in the default Terminal.app, everything works well but the color. It only support for 16 color, but my previous colorscheme is base on base16-color. So I switch to iTerm2.

but in the iTerm2, I found ctrl-r, i.e. the back-reverse-search, quick search for history, it's not working.
When I enter ctrl-r, I can hear the alerm sound, and nothing prompt on the screen.

I kept struggling to seek for the iTerm2 issue solution or to get back to use Terminal.app and change a colorscheme for nearly 1 hour. finally, I read the [github issue][issue-search-solution], it solves the problem and save my life, thanks the contributors again!

add the follow to `.zshrc` files.

{% highlight sh %}
bindkey -M viins '^r' history-incremental-search-backward
bindkey -M vicmd '^r' history-incremental-search-backward
{% endhighlight %}


[dotfiles]: https://dotfiles.github.io
[vim-tmux-omg]: https://www.youtube.com/watch?v=5r6yzFEXajQ
[nicknisi-dotfiles]: https://github.com/nicknisi/dotfiles 
[soooooot-dotfiles]: https://github.com/soooooot/dotfiles 
[oh-my-zsh]: https://github.com/robbyrussell/oh-my-zsh
[issue-search-solution]: https://github.com/sorin-ionescu/prezto/issues/596  
