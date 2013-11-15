---
layout: post
title: "Vim with System Clipboard"
date: 2013-11-15 16:14
comments: true
categories: Tip Vim OSX
---

With vim I got really addicted to keyboard-only usage. But there is one thing I was always missing: integration with the system clipboard. I had to switch back to the mouse for that. As I learned today system clipboard is supported by vim. Unfortunately the version packaged with OSX is not compiled with this feature enabled.

<!-- more -->

You can check this yourself by typing `vim --version` on the command line. The output should contain something like `+clipboard` (`+` means enabled). Probably you have a `-` instead. Luckily with [homebrew](http://brew.sh/) you can easily install vim with clipboard enabled.

### 1. Install vim

```
brew install vim
```

This will install vim in the `/usr/local/bin` directory. So far the original vim is still in place and used. We have to give the brew version precedence.

### 2. Update PATH order

An easy way to make sure the homebrew version is loaded is to make sure its path is defined before the system path. Locate your `PATH` configuration.

For <a href="http://en.wikipedia.org/wiki/Bash_(Unix_shell)">bash</a> it's one of these files:

* ~/.bash_profile
* ~/.bash_login
* ~/.profile

For [zsh](http://en.wikipedia.org/wiki/Z_shell) open this file:

* ~/.zshrc

Make sure `/usr/local/bin` is declared before `/usr/bin`:

```
export PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin...
```

Reload the config, with zsh for example:
http://localhost:4000/blog/2013/11/15/vim-with-system-clipboard/

```
source ~/.zshrc
which vim
vim --version
```

The vim path should be `/usr/local/bin/vim`. And `vim --version` should contain the `+clipboard`. A nice side effect of changing the PATH order is that you can also update git using homebrew.

### 3. Enable clipboard in .vimrc

As a last step you have to enable the system clipboard in vim. Open your `.vimrc` and add the following line:

```
set clipboard=unnamed
```

From now on you should be able to use the system clipboard when using vim verbs (`y`, `d`, `p` etc).
