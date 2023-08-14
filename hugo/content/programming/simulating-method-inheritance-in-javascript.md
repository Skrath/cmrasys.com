---
title: Simulating Method Inheritance in Javascript
description: Javascript doesn't use class-based inheritance in the traditional sense. This is a short guide to simulating method inheritance.
date: 2023-06-13T18:11:09.941Z
publishdate: 2013-09-13T16:24:30.701Z
draft: false
tags:
  - Constructor
  - Inheritance
  - OOP
  - class
  - function
  - magic methods
  - prototype
categories:
  - Javascript
background: ""
displayClass: article
lastmod: 2023-06-13T18:11:11.023Z
---

[Javascript](http://en.wikipedia.org/wiki/JavaScript) is inherently a classless object-oriented language, so performing a lot of the things we're used to in say, PHP, doesn't work the same way. This doesn't mean that these things are impossible though. In fact, Javascript is extremely open-ended and very fluid in how it implements OOP. So what if we need to extend the existing functionality of a function of an object, effectively replicating method inheritance?

<!--more-->

## The Problem

Let's say we have an Accordion object that our site is using and we want to attach Google Analytics tracking to one of the functions of that Accordion object. This method may be called directly, or more likely, via an event observer. Certainly, the easiest thing to do would be to just add the code to the existing function and be done with it, but let us also assume that we cannot or should not have access to the actual accordion code. Javascript is great at letting us completely replace a function, but we also want to preserve the original functionality.

## The Original Implementation

We don't really care that much about the actual accordion code, so we're going to mostly ignore that. Here's the actual instantiation of the Accordion object that we care about:

{{< highlight javascript >}}
var accordion = new Accordion('checkoutSteps', '.step-title', true);
{{< / highlight >}}

There's not a whole lot going on here. We're simply passing some classes and other config information to the accordion constructor. Everything else is contained in, and controlled by, the actual accordion code that we're not going to be dealing with.

## Faking Inheritance

The first thing we need to do is "replace" the original Accordion class with our own:

{{< highlight javascript >}}
// We need to extend the functionality of the base Accordion class
function custAccordion(elem, clickableEntity, checkAllow) {
    this.parent = new Accordion(elem, clickableEntity, checkAllow);
    this.prototype = this.parent;
}
{{< / highlight >}}

Right away you'll notice that we're using a bunch of parameters that were defined by the original accordion class `(elem, clickableEntity, checkAllow)`. This is the same function definition being called from the original new `Accordion()` call. Even though we don't do anything with these variables, it's important that we put them in here or our resulting object won't be created properly.

Next, we create a new variable `this.parent` which contains the original Accordion object. We're doing this so that we can make sure we retain a pure copy of the original object so we can call its functions later.

In the last line, we're setting the function's prototype to that of the previously defined this.parent. This actually allows us to completely duplicate the `Accordion` object into the `custAccordion` object. At this point, we have two copies of the accordion object, one of which is contained by the other inside of the `this.parent` variable.

Now we need to perform our actual modification, which is replacing the function `openSection()`:

{{< highlight javascript >}}
// Hook into the openSection function, add our GA tracking logic,
// and make sure to call the original function.
custAccordion.prototype.openSection = function(section) {
    // Put whatever logic you need in here
    _gaq.push(['_trackPageview', 'some useful info here']);

    this.parent.openSection(section);
}
{{<  / highlight >}}

What we're doing here is completely replacing the function `openSection()`. The contents of the replacement are whatever you need them to be. In this case, I'm simply putting in a call to a Google Analytics function. The last line is the important piece however, here we're calling the original `openSection()` function which is contained inside of our `this.parent` variable. Depending on your needs, this can be called from the beginning of the new function, or even in the middle of your new code.

Finally, we need to instantiate our new Accordion object:

{{< highlight javascript >}}
var accordion = new custAccordion('checkoutSteps', '.step-title', true);
{{< / highlight >}}

As you've probably noticed, this looks identical to the original instantiation except that we're calling `custAccordion()` instead of `Accordion()`. If we're worried about other people instantiating the original Accordion object instead of the new one in some other file, we could always completely replace the Accordion prototype with our new one:

{{< highlight javascript >}}
Accordion.prototype = custAccordion.prototype;
{{< / highlight >}}

Then we just need to make sure our prototype updates get called early on in the load stack of the website and anyone who creates a new `Accordion` object will actually be creating our new `custAccordion` object.
