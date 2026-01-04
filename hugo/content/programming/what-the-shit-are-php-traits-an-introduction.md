---
title: What the Shit are PHP Traits? An Introduction
description: ""
date: 2023-06-13T08:13:39.176Z
publishdate: 2013-06-30T14:47:23.642Z
draft: false
tags:
  - OOP
  - classes
  - constructor
  - hierarchy
  - inheritance
  - instantiation
  - traits
categories:
  - PHP
displayClass: article
background: programming/php-code.jpg
lastmod: 2023-06-13T08:13:40.412Z
---

With version 5.4.0 of PHP, came something called [traits](http://php.net/manual/en/language.oop5.traits.php). Granted, this is a fairly new version of PHP so there's a good chance most developers won't be using it on a live server for at least another 25 years, but it's still good to learn about these kinds of things.

Similar to extending classes with child classes, traits allow you to share code, functionality, and variables between objects. The difference here is that sharing code via classes only allows you to share vertically. That is, you have to create and use a strict hierarchy of classes if you want to share code with all of them. This often creates "pyramids" of classes where every class is based on some highly basic originating class. This structure *can* be fine, but it's not always appropriate from a taxonomical perspective. Traits, on the other hand, allow you to share code *horizontally*, meaning that there doesn't need to be any explicit relationship between the objects.

<!--more-->

## What the Hell Does That Mean?

This means that you can define discreet behaviors, attributes, etc that can be shared across hierarchy lines. You can now share code between two objects that are not related in any way. Now, obviously, something like this can be pretty easily abused. Knowing when you should be using a class hierarchy versus using traits is pretty important and might not be immediately obvious.

Luckily, if you go to the PHP documentation page for traits, you'll find a giant suite of examples that are vague and effectively meaningless. Though, they do teach you how to properly use traits, so that's nice.

## Examples Would be Nice

So, instead of throwing you out into the wilderness with nothing but a vague notion of how to use traits, I thought I might provide some examples of how I've been using them lately.

### The Problem

Most of us have probably written a constructor before. A lot of these constructors are pretty simple and merely set some basic variables and probably look a lot like this:

{{< highlight php >}}
public function __construct($first, $second, $third) {
    $this->first = $first;
    $this->second = $second;
    $this->third = $third;
}
{{< / highlight >}}

Now, there's nothing wrong with having a constructor like that, but if you have a bunch of fairly simple classes that all have basically the same constructor (albeit with different parameters), then you're looking at a good use-case for traits (assuming those classes aren't already related to each other). So, instead of writing the same code over and over again in multiple classes, you can write a single trait and have your classes use it.

### The Solution

Here's a pretty simple solution that I've been using a version of:

{{< highlight php >}}
trait BasicConstruct {

    public function __construct($value_array = []) {
        foreach ($value_array as $name => $value) {
            if (property_exists($this, $name)) {
                $this->$name = $value;
            }
        }
    }
}
{{< / highlight >}}

The first thing you should notice is that the constructor now takes a key-value pair array. You don't necessarily have to make the constructor work this way, and you can in fact do the same thing using the same parameterized list as our previous constructor. I just personally prefer to build my objects using arrays for parameters. The benefit of this constructor over the previous one is that I can put my parameters in any order and provide any number (or none) of them. This also makes sure that your class actually has the properties you're trying to set by checking for them via `property_exists()`. What this means is that if your class doesn't have one of the properties in the supplied parameter list, it will simply ignore it. This isn't technically required, but it will prevent your constructor from attempting to set properties that don't exist and throwing a run-time error.

Now you can simply have an existing class using the trait:

{{< highlight php >}}
class Character {
    use BasicConstruct;
    ...
}
{{< / highlight >}}

And performing an instantiation is easy as well:

{{< highlight php >}}
$someGuy = new Character(['name' => 'Steve', 'face' => 'dumb']);
{{< / highlight >}}

Of course, you should keep in mind that this constructor is pretty simple right now. This also makes it a lot harder to "extend" the constructor with child classes of the original classes that are using it. Basically, if you need to change the constructor, you would need to re-write it in your class. This means that performing custom setup operations that would typically occur in a constructor method become a bit more difficult to pull off. *However*, you can always improve the trait to offer this kind of functionality.

## Wrap Up

Hopefully, this should give you some ideas on how to properly use traits. But remember, the code-sharing functionality in traits and classes can sometimes be used interchangeably, so you need to put some careful thought into determining which you should actually be using. Just because you can hit screws with a hammer doesn't mean that's what you're supposed to do.
