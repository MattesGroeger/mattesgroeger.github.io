---
layout: post
title: "From Wordpress to Octopress"
date: 2014-01-14 20:00
comments: true
categories: Blog
---

For a long time I wasn't happy with my [Wordpress](http://www.wordpress.org) blog. Maintenance and hosting seemed unnecessary complex. When researching alternative solutions I found [Octopress](http://octopress.org/). In this article I highlight the reasons that made me switch.

<!-- more -->

## Problems with Wordpress

Wordpress is a self-hosted [PHP](http://php.net/) + [MySQL](http://www.mysql.com/) blog system. As the code is [open source](http://wordpress.org/download/source/), getting compromised with zero-day exploits is easy. That's one reason why Wordpress and its plugins provide frequently updates. But even though I updated often I got a lot of spam comments. Even the captcha system I had in place didn't stop them.

It is very convient to write blog posts with Wordpress as it comes with a [WYSIWYG editor](http://en.wikipedia.org/wiki/WYSIWYG). But that means also that the content is residing on the server only. In the worst case I could loose all my data if the server breaks. Therefore I have to make backups of all files (images, videos, ...) and the database regularily.

Also I want to experiment with layout changes locally. Therefore I have to setup everything on my computer as well. When done, it needs to be uploaded via FTP. That's rather annoying. I want something that is versioned with [git](http://git-scm.com/) and deployed automatically on each commit.

## Octopress

Exactly this is possible with Octopress. At it's core it uses [Jekyll](http://jekyllrb.com/docs/usage/)'s static site generator which is also used for [GitHub Pages](http://pages.github.com/). Load times are super fast because its just static HTML.

Blog posts are written in the [Markdown](http://daringfireball.net/projects/markdown/) format which then gets translated into HTML. I really enjoy writing in Markdown. [The syntax](http://daringfireball.net/projects/markdown/) is concise and let's me focus on the content. [This is how](https://raw.github.com/MattesGroeger/mattesgroeger.github.io/source/source/_posts/2013-11-15-vim-with-system-clipboard.markdown) a raw blog post looks like.

Octopress comes with a number of plugins that enhance the possibilities of Markdown. Examples are [code blocks](http://octopress.org/docs/plugins/codeblock/), [block quotes](http://octopress.org/docs/plugins/blockquote/) or [html5 video tag](http://octopress.org/docs/plugins/video-tag/).

The provided default template (classic) allows for an easy start. Making changes to the layout [is simple](http://octopress.org/docs/theme/). Stylesheets are written in the [scss format](http://sass-lang.com/documentation/file.SCSS_FOR_SASS_USERS.html). My blog layout is an adjusted version of the [whitespace theme](https://github.com/lucaslew/whitespace).

The best part is the deployment via [GitHub Pages](http://octopress.org/docs/deploying/github/). Blog posts are created on a git branch (`source`). To deploy changes I run `rake deploy` which:

* Generates the HTML
* Merges everything to `master`
* Pushes `master` to GitHub

GitHub will autmatically update the website afterwards. You can also deploy to [other hosting providers](http://octopress.org/docs/deploying/). Make sure to also push your `source` branch in order to have a backup on the server. With git under the hood you can always roll back changes.

With GitHub Pages you can use your own domain for the Octopress blog. This way the user won't even notice that you use GitHub under the hood. Find a detailed explaination on how to do it [in the GitHub FAQ](https://help.github.com/articles/setting-up-a-custom-domain-with-pages#setting-the-domain-in-your-repo).

## Conclusion

Static pages load much quicker than dynamic ones. But you have to rely on external services for dynamic content. Imagine the comment provider (Disqus) stops his service and all comments would be lost. I for myself take this risk in exchange for easier maintainance.

So far I'm pretty happy with Octopress. If you are interested, go and [check it out for yourself](http://octopress.org/).
