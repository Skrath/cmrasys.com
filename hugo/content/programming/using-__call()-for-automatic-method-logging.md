---
title: Using __call() for Automatic Method Logging
date: 2023-06-13T08:14:39.685Z
publishdate: 2014-05-26T22:32:52.006Z
draft: false
tags:
  - call
  - class
  - invoke
  - log
  - logging
  - magic
  - method
categories:
  - PHP
background: ""
displayClass: article
description: A short explanation of using the __call() magic method in order to create automated method logging
lastmod: 2023-06-13T08:14:41.136Z
---

I have found that time and time again, having good logging can be one of the most essential tools for developing and/or debugging something. It gives you a better insight into what is actually happening in your program. It lets you keep a history of how your program has been performing. It can give valuable debugging data in live environments where editing the code is generally a bad idea. Because of all of this, I try to log as much as I can.

One of the easiest things to do for logging is to simply log every function and/or method call. I'll leave the details of creating an actual logging system for another time, but this often means going into each and every one of your method calls and adding something like `Debug::log();` at the very least. Obviously, this can get very tedious and makes it very easy to simply forget to add in your logging call.

<!--more-->

## A More Automated Solution

Since we're talking about functionality that is designed to occur with every method call, it stands to reason that we should be able to automate it. Technically, the best method for modifying existing method calls and attaching additional functionality is called "[Monkey Patching](http://en.wikipedia.org/wiki/Monkey_patch)" and is not currently supported by PHP. However, there are a few tricks that we can use to get fairly close results.

PHP has quite a few ["magic" methods](http://www.php.net/manual/en/language.oop5.magic.php) that do all manner of interesting things. One of them is `__call()` and it is automatically invoked when code attempts to access an inaccessible method. In short, it lets you catch every call to any method name in a class so long as that method either doesn't exist or is not accessible.

What this means, is that you can effectively overload all of your class's methods and have them automatically invoke a logging method. Below is a fairly simple `__call()` method that does just that:

{{< highlight php >}}
public function __call($methodName, $arguments) {
    $methodList = get_class_methods($this);
    $methodCallName = '_' . $methodName;

    if (in_array($methodCallName, $methodList)) {
        Debug::Log(null, 16, true);
        return call_user_func_array(array($this, $methodCallName), $arguments);
    }
}
{{< / highlight >}}

This method first gets a list of the class's methods via `get_class_methods()` which we will check against in a moment. The next part builds a new `$methodCallName` by prepending an underscore to the original `$methodName`. Why are we doing this? Remember, `__call()` will only get invoked if a method is *inaccessible*, so we need to rename our methods by putting an underscore in front of them. This way, whenever a piece of code attempts to call that method it won't exist, and `__call()` will be invoked instead. `__call()` will then prepend an underscore to the method name, and if that new name is found in the `$methodList` it will call that method. Additionally, it will also call our logging method.

We are also using `call_user_func_array()` in order to invoke our method as we don't know anything about the arguments being used and need to pass them through with an array. Additionally, we need to remember to return the results of our call in order to properly pass through any results that the called method might be returning.

The only remaining step is to go and rename any of our methods that we want to have logging by prepending an underscore in their name. We don't need to modify any of the code that *calls* these methods as the `__call()` method itself will handle the renaming logic. Again, without monkey patching, there is no way to fulfill this functionality without renaming all of the methods.

## Other Uses

This sort of usage for `__call()` is very suitable for being defined inside of a trait that can then be applied to numerous classes that you need logging on. Though, you will need to keep in mind that a class can only have one definition for `__call()` so be careful if you need different behaviors in different classes.

Obviously, there are lots of additional uses for something like this other than just calling logging methods. One common usage is to set up automatic getters and setters for a class. One use that I plan to use is that of automatically creating [observer](http://en.wikipedia.org/wiki/Observer_pattern) points for easier method extendability. Whatever you might decide to use it for, the `__call()` magic method certainly has a lot of usefulness.
