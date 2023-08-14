---
title: Welcome to the New Website!
description: ""
date: 2023-06-13T19:00:07.221Z
lastmod: 2023-06-13T19:00:07.221Z
publishdate: 2023-06-13T19:00:07.222Z
draft: false
tags:
  - cmrasys.com
  - Git
  - Hugo
  - Markdown
  - writing
categories:
  - Site Updates
background: ""
displayClass: default
---

After an arduous and protracted development (definitely not due to procrastination), I've finally updated the website with something entirely new.

<!--more-->

## Changes

The site is no longer running on [WordPress](https://wordpress.com/) and is instead a static website being built with [Hugo](https://gohugo.io/). A static website runs a lot faster and I never really needed all of the extra bells and whistles of WordPress. Hugo also uses [Markdown](https://en.wikipedia.org/wiki/Markdown) files for all of the site content which provides a pretty straightforward avenue for writing as well as version control using [Git](https://git-scm.com/).

Though, perhaps most importantly, Hugo gives me all the control I need to make the site into whatever I want it to be without also having to deal with the needless complexity of WordPress. I also don't need the site to do anything all that fancy. It's really just a blog after all. That said, I have put together some useful shortcodes for some extra utility inside a post:

### Random blocks

{{< infoblock Section programming >}}

{{< excerpt Section writings 50 >}}

### Lists

{{< tax_listing categories humor >}}

{{< tax_listing categories poetry >}}

All of the above sections will update automatically whenever the website gets built with new content. But because it's a static website, there's no additional processing or delay on the user's end having to reprocess everything on every page load.

{{< note >}}And of course, I can make little notes.{{< /note >}}

## The Near Future

I'm planning on putting together a few articles on the creation of the website in Hugo, as well as some of the more interesting tricks I pulled off with the shortcodes. I also have plans for doing a series of notes/annotations for older pieces with revisions. The purpose of this is to provide a bit more insight into the actual writing process.

So there's new stuff to look forward to coming soon, now I just have to actually create it...
