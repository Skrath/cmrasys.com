---
title: Displaying a Random Excerpt in Hugo
description: How to extract and display a randome selection of a post with a Hugo shortcode
date: 2023-06-17T17:40:38.043Z
lastmod: 2023-06-17T17:40:38.899Z
publishdate: 2023-06-17T17:40:39.837Z
draft: false
tags:
  - pseudorandom
  - shortcode
  - shuffle
  - split
  - templating
  - delimit
categories:
  - Hugo
background: ""
displayClass: article
---

While I was first designing this website I knew I wanted to have the ability to display excerpts from various posts/writings. There isn't any built-in functionality for doing this, but [Hugo](https://gohugo.io/) does contain all of the necessary tools to build this feature yourself.

<!--more-->

## An Example

{{< excerpt Section writings 50 >}}

This excerpt is being generated with a simple shortcode:

{{< highlight go-html-template >}}
{{</* excerpt Section writings 50 */>}}
{{< / highlight >}}

The contents of the shortcode template can be seen below:

{{< highlight go-html-template >}}
{{ $seed := now.Unix }}
{{ $post := index (shuffle (where .Site.RegularPages (.Get 0) (.Get 1) )) 0 }}
{{ $words := split ($post.RawContent) " " }}
{{ $start := mod (add (mul 13 $seed) 97) (len $words) }}
<blockquote>
    ...{{ delimit (first (.Get 2) ( after $start ($words))) " " }}...
    <cite><a href="{{ $post.RelPermalink }}">{{ $post.Title }}</a></cite>
</blockquote>
{{< / highlight >}}

## Usage

The first two parameters ("Section" and "writings") tell the shortcode what content to look for and are passed to Hugo's `where` function using `(.get 0)` and `(.get 1)`:

{{< highlight go-html-template >}}
{{ $post := index (shuffle (where .Site.RegularPages (.Get 0) (.Get 1) )) 0 }}
{{< / highlight >}}

The rest of this line is shuffling the results of the query and then pulls out the very first result. In the above example this means that we're pulling a random post from anything in the writings section, but we can also grab a specific post by being more specific:

{{< highlight go-html-template >}}
{{</* excerpt Title "Why I Don't Eat Raisins" 25 */>}}
{{< / highlight >}}

{{< excerpt Title "Why I Don't Eat Raisins" 25 >}}

The very last parameter, `(.get 2)`, is the number of words to display. I decided to go with words instead of characters to promote better readability of the output.

## How the Excerpt is Generated

The important part is figuring out how to only display a discreet portion of the post. Hugo has a `shuffle` function for randomizing an array but this isn't quite what we need. What we really need is to pick a random spot in the text and then pull out the required number of words after.

First, we'll `split` the post's content into words. This is done by delimiting by an empty space `" "`:

{{< highlight go-html-template >}}
{{ $words := split ($post.RawContent) " " }}
{{< / highlight >}}

Technically, Hugo doesn't include any functions for random number generation. But it does have competent math functions which we can use to generate random numbers.

Pseudorandom number generation typically uses a timestamp as a seed:

{{< highlight go-html-template >}}
{{ $seed := now.Unix }}
{{< / highlight >}}

Then we need to pick a random spot to start our excerpt:

{{< highlight go-html-template >}}
{{ $start := mod (add (mul 13 $seed) 97) (len $words) }}
{{< / highlight >}}

Don't worry too much about the specifics of the math going on here, this is a pretty basic [pseudorandom number generator](https://en.wikipedia.org/wiki/Pseudorandom_number_generator). The important part is `(len $words)`. This is the total number of words found in the post's content. We're using this to define the upper bounds of our random number as we don't want to accidentally go past the end of the post.

Finally, we extract the words we want and rebuild our text:
{{< highlight go-html-template >}}
{{ delimit (first (.Get 2) ( after $start ($words))) " " }}
{{< / highlight >}}

This might look a little confusing as there's some nesting going on performing multiple actions. We're extracting our excerpt with `first (.Get 2) ( after $start ($words))`. Remember that `(.get 2)` refers to our word count specified in the calling shortcode. So if we specified 25, this line is saying "get the first 25 words after `$start`", where `$start` is the random number generated previously. Hugo doesn't mind if our length happens to exceed the number of remaining words.

Because this returns an array of words we need to put them all back together into a text string. We can do that with `delimit` which performs the exact opposite operation of our earlier usage of `split`, once again using `" "` as a delimiter.

## Some Caveats

You may have noticed that all of the examples I posted here are from pieces of prose. The reason is that there's nothing in the shortcode that guards the output of the text. So, if we pull an excerpt from a post that contains other shortcodes, we might accidentally render an additional shortcode *inside* our excerpt. Additionally, pulling a random excerpt from a piece filled with programming code will look quite ugly as none of the code formatting will be displayed in the excerpt.
