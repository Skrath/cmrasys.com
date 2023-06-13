---
title: Directory Sorting with Bash
description: A simplified look at using bash scripting to perform directory sorting based on the file structure depth.
publishdate: 2013-08-18T17:22:44.803Z
date: 2023-06-13T02:54:42.338Z
lastmod: 2023-06-13T02:54:43.778Z
draft: false
tags:
  - cut
  - directory
  - file
  - find
  - list
  - NSIS
  - Nullsoft
  - SED
  - sorting
categories:
  - Bash
background: ""
displayClass: article
---

A while back, I was working on creating an installer for an in-house project using the [Nullsoft](https://en.wikipedia.org/wiki/Nullsoft) installer ([NSIS](http://nsis.sourceforge.net/Main_Page)). It’s an interesting system but it’s a bit flaky in a couple of areas. One of these areas is installing and/or removing entire directory structures. It appears that the system is generally designed for explicit definitions of all directories and files. While this may often be a perfectly acceptable solution, I wanted a more dynamic system. Because I’ve been developing the installer externally to the actual development of the project, I don’t keep track of what files or directories need to be included, nor do those files and directories remain stable.

<!--more-->

The NSIS system uses its own scripting language to build the installer. There are many various macros out there that do all sorts of things. There are undoubtedly several that do exactly what I’m trying to do here, but I’m not very familiar with the NSIS system. Subsequently, I came up with a rather odd solution. I’m using a bash script to generate the NSIS script. This allows me to do many interesting things, one of which is the ability to pass the script information from the build file I’m using to orchestrate the entire process.

However, I still need to explicitly list all of those directories and the files within them. Additionally, because I’m running the script on a Linux computer and the installer will be running on Windows, directory paths don’t correlate properly because of each OS’s usage of backslashes and forward slashes. I’m providing two variables to the script: `$APP_NAME` and `$APP_DIR`. `$APP_NAME` doesn’t come into play here, but `$APP_DIR` is where all of the actual work will be taking place. Here’s the block of code that generates the directory and file definitions for the NSIS script:

{{< highlight bash >}}
CUR_DIR=$( pwd )

cd $APP_DIR

for dir in $( find . -type d | cut -d. -f2- ); do
    WIN_DIR=$( echo "$dir" | sed ‘s|/|\\\\|g’ )
    SCRIPT="$SCRIPT setOutPath \$INSTDIR${WIN_DIR}\n"
    for file in $( find ."$dir" -maxdepth 1 -type f | cut -d/ -f2- ); do
        SCRIPT="$SCRIPT File ../../$APP_DIR$file\n"
    done
    SCRIPT="$SCRIPT\n\n"
done

cd "$CUR_DIR"
{{< / highlight >}}

A lot of this looks pretty insane, so I’ll go through the lines. I’m not going to go into detail about the specific mechanics of the shell commands here.

{{< highlight bash >}}
for dir in $( find . -type d | cut -d. -f2- ); do
{{< / highlight >}}

This is the first part of note. What this is doing is generating a list of directories using the find command. The `cut` is there to simply strip off the dot that appears at the beginning of the listings.

{{< highlight bash >}}
WIN_DIR=$( echo "$dir" | sed ‘s|/|\\\\|g’ )
{{< / highlight >}}

Because I’m writing a script that’s designed for a Windows file system I have to change all of the forward slashes into backslashes. Additionally, because I’m eventually going to be sending this output through a final echo using the `-e` parameter I have to use four backslashes. Essentially, I need to send two slashes through to the echo statement, telling it to treat it like a normal backslash. To do this, I have to escape both of them here, giving me a total of four. Yes, this is silly.

{{< highlight bash >}}
SCRIPT="$SCRIPT setOutPath \$INSTDIR${WIN_DIR}\n"
{{< / highlight >}}

This just appends the actual path declaration to my `$SCRIPT` variable. This is the variable that will get sent through echo into a file at the end of the script.

{{< highlight bash >}}
for file in $( find ."$dir" -maxdepth 1 -type f | cut -d/ -f2- ); do
{{< / highlight >}}

This is very similar to the first for loop, except it’s designed for files. In this case, I have to use `cut` to strip off the slashes that the find program sticks onto the listing.

After that, it’s just spitting out the actual files and some minor formatting to make the final script a little more readable. Towards the end of the script I need to do a very similar thing: delete all of these files and directories for the uninstaller. At first, I thought I could use the same loops, replacing the commands with their respective deletion counterparts, but an odd problem arose. The NSIS uninstaller won’t delete a directory if it has any files in it. What this means is that I have to make sure that I clear out everything in the right order. I need to start with the deepest directories first and work backward.

As far as I know, there’s no standard way to get a listing sorted this way, but I figured out a few tricks to make it work:

{{< highlight bash >}}
for dir in $( find . -type d -printf "%d %h/%f\n" | sort -nr | cut -d. -f2- ); do
{{< / highlight >}}

This is very similar to the first loop in the install section but with a few important changes. First of all, I’m specifying my output for the find command. I’m using `"%d %h/%f\n"`. The important part of this is the `%d`. This is the depth of the directory. The rest of it is essentially the same output you would normally get. At this point, it’s pretty easy to pipe the list into sort and organize the whole thing by that initial depth integer. After that, I can send it through `cut`, which will peel off not just the extra dot but also the depth as well. What I’m left with is a clean list of directories reverse-sorted by depth.

This may not be the most optimal solution to this problem but it’s the first one I came up with and it certainly looks pretty interesting.
