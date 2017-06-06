+++
author = "alan"
tags = ["nerd", "wordpress", "hugo", "static site", "go"]
date = "2017-05-29T23:39:00-04:00"
description = "Static Site Generation with Hugo - it is so freakin easy!"
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
title = "Why wait? Adopt hugo for Static Site Generation"

+++

Why... oh why... why did I wait so long?

I've been talking about switching to a static site generator for years,
since [jekyllrb](https://jekyllrb.com/) started gaining popularity.

I understood the benefits, but I had no compelling reason to switch...
_(sort of how I resisted `git` for a few years because `svn` worked well enough)_

But who wants a WordPress site?

Well, ok, maybe you are using 12 plugins and doing fancy stuff.

But if all you are doing is hosting a handful of pages and perhaps blog posts,
Wordpress is just something you have to maintain and worry about security on.

By contrast, static sites are super-cheap or **free** to host, and are **unfuckwithable**.

---

Anyway, I finally decided that I'd dust off my website,
a [personal site](https://zeroasterisk.com)
and a [code/dev site](https://code.zeroasterisk.com).
I looked at a couple of React static site generators,
and a few other node based ones... but I didn't really like their final static output.
They tried to be too fancy, more like "single-page-app" generators
but what I really want here, is _dead-simple HTML/JS/CSS_.

So I re-reviewed [hugo](https://gohugo.io) again and decided to adopt it.

Reasons for choice:

- Robust community
- Simple static HTML outputs (good for static hosting a multi-page site)
- Simple to extend and modify themes
- Flexible markdown/metadata (mostly)
- Excellent local server for live preview/development
- No dependencies or complicated install (easier to recommend to others)

### How to get started with Hugo

Installation took &lt; 15 minutes:

```sh
brew install hugo
hugo new site zeroasterisk.com
cd zeroasterisk.com
mkdir themes
git clone https://github.com/jeblister/bulma.git themes/bulma
vim config.toml
```

> NOTE there are a lot of pre-built [Hugo themes](https://themes.gohugo.io), and once you download,
> they are very easy to customize, preview, and use.

Running the server locally is super simple _(and crazy performant)_:

```sh
$ hugo server -w

Started building sites ...
Built site for language en:
0 of 3 drafts rendered
0 future content
0 expired content
336 regular pages created
144 other pages created
0 non-page files copied
239 paginator pages created
24 categories created
43 tags created
total in 394 ms
Watching for changes in /Users/alan/Code/Websites/zeroasterisk.com/{content,static,themes}
Serving pages from memory
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

Building final outputs is even simpler:

```sh
$ hugo server -w

Started building sites ...
Built site for language en:
0 of 3 drafts rendered
0 future content
0 expired content
336 regular pages created
144 other pages created
0 non-page files copied
239 paginator pages created
24 categories created
43 tags created
total in 668 ms

$ ls public
2005  2007  2009  2011  2013  2015  about  categories  img  post  wp-content  index.html  robots.txt
2006  2008  2010  2012  2014  2017  alan   css         js   tags  404.html    index.xml   sitemap.xml
```

### How to Migrate from WordPress to Hugo

I had several years of _(sporadic)_ history in WordPress.

I used [SchumacherFM/wordpress-to-hugo-exporter](https://github.com/SchumacherFM/wordpress-to-hugo-exporter)
which is a really simple tool to export WordPress posts/pages into markdown files for Hugo.

1. download zip of plugin
2. unzip into `<my_wordpress_path>/wp-content/plugins/`
3. cd into that dir an `php hugo-export-cli.php`
4. `scp` the `/tmp/wp-hugo.zip` to my laptop

### How to host for free with GitHub and CloudFlare

I already used GitHub for projects.

I already used CloudFlare and had my domain DNS there.

So - I made a GitHub project for
[hugo-zeroasterisk.com](https://github.com/zeroasterisk/hugo-zeroasterisk.com)
and used `gh-pages` branch for the public content...
Here is the [article I followed for setup](https://gohugo.io/tutorials/github-pages-blog/)
but I did make a few changes when I setup my
[publish_to_gh_pages.sh](https://github.com/zeroasterisk/hugo-zeroasterisk.com/blob/master/publish_to_gh_pages.sh) script.

Once setup - Telling CloudFlare to "listen" to GitHub was as simple as a CNAME setup on CloudFlare and on GitHub.

So now it's hosted, for free, super-fast, on a CDN.
