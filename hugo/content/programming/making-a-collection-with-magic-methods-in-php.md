---
title: Making A Collection With Magic Methods in PHP
description: A brief overview of how to use some of PHP's magic methods when creating an object collection class, such as __get(), __set(), and __invoke()
date: 2023-06-13T18:12:03.552Z
publishdate: 2013-07-05T22:34:44.741Z
draft: false
tags:
  - class
  - collection
  - get
  - invoke
  - magic methods
  - object
  - set
categories:
  - PHP
background: ""
displayClass: article
background: programming/php-code.jpg
lastmod: 2023-06-13T18:12:04.782Z
---

A lot of PHP frameworks have something called "collections" (though, I'm sure some of them use a different term). Basically, these are special classes whose sole purpose is to hold a bunch of the same type of objects within them. They usually provide some extra functions for working with those objects and also a bit of data protection by making the contained objects themselves private or otherwise unavailable. A lot of this intrinsic functionality is provided by using magic methods.

<!--more-->

I'm not going to cover every single bit of functionality that one might include in a respectable object collection class, but I would like to go over some neat tricks that you can pull off using [magic methods](http://php.net/manual/en/language.oop5.magic.php).

## First, A Concept

Let's start with the basics:

{{< highlight php >}}
class ObjectGroup {
    private $container = [];
}
{{< / highlight >}}

Seriously, that's all you actually need. Of course, since our only member variable is private, there's not a whole lot of usage we can get out of this currently. There are a lot of different ways you can populate the `$container` variable, so let's just assume that you already have a way of doing that. We're going to focus more on providing a useful way to access the contents of the collection from outside the class.

## OK, Let's Get A Magic Method

For this exercise, we're going to assume that the `$container` variable is a key-value pair array and will create a `__get()` magic method to access it:

{{< highlight php >}}
class ObjectGroup {
    private $container = [];

    public function __construct() {
        $this->container['steve'] = new Character('blah', 'blah');
    }

    public function __get($name) {
        return $this->container[$name];
    }
}
{{< / highlight >}}

Now you can access the contents of the container by simply using `$characterContainer->steve`. The `__get()` function is invoked whenever any access to a private or even non-existent variable occurs. Since the `$container` is private, it's not accessible and the `__get()` function is invoked instead, which just returns the object inside the container with the key specified. This current setup isn't very robust and will cause errors if you attempt to access a key that doesn't exist within the `$container` variable.

## How About Something Moderately Useful?

As you may have already guessed you can use the `__set()` function in almost the exact same fashion to store something in the `$collection` variable. But that's super boring, so we're going to do something a little different.

{{< highlight php >}}
public function __set($name, $value) {
    $item = $this->collection[$name];
    return $item($value);
}
{{< / highlight >}}

Wait, what? If we're assuming that the `$collection` variable is full of objects, how can we attempt to invoke it like a function? All we need to do is provide an `__invoke()` method on our container objects:

{{< highlight php >}}
class Character {

    public function __invoke($value = '') {
        $this->status = $value;

        return $this->status;
    }
}
{{< / highlight >}}

Now, if we use `$characterCollection->steve = "dead";` the collection class will find the proper variable in the $collection array, and then invoke it with whatever is after the equals sign. The `__invoke()` magic method is called whenever an object is attempted to be used like a function, which causes our contained object's `$status` member variable to be set to the value supplied from the `__get()` call.

While this does make the assumption that our contained classes possess the `__invoke()` magic method, it also defers actual setting logic to the contained class itself. We could have any number of different kinds of classes all performing different and specific setting logic without having to modify the container class itself.

## Wrap Up

Obviously, the examples shown here are overly simplistic, but they should provide a few ideas on how to make some interesting interactions between classes using magic methods. Don't forget that there are also lots of ways to ensure that contained objects possess the functions you need via hierarchies, interface implementations, and traits.
