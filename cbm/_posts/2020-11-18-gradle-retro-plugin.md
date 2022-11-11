---
title: Gradle Retro Assembler plugin is released
description: End to end solution for assembling, building and testing your KickAssembler projects.
layout: post
post-url: gradle-ra-plugin
categories: cbm asm c64lib
pic: /cbm/img/gradle-logo.png
---
The idea of a tool that can automatically download all dependencies and build my 6502 projects
arose some time [ago][1]. In the meantime I started using 
[64spec][2] for testing my assembly code. The library itself is great but requires a lot of manual
operations once a great number of test files are created.

I have noticed that Michał Taszycki, an author of [64spec][2], has already written
a proof of concept tool, so that 64spec can be used from command line rather than by using
user interface of Vice. I have looked into [64spec-cli][3] and decided that it would
be possible to reimplement it in Java.

The result of my work has actually exceeded my expectations: it's a fully functional Gradle plugin
written in Kotlin, which not only can assemble and test KickAssembler code, but also has a simple yet
effective dependency management mechanism built in.

## Why builds should be automated?
As a build we understand a process of compiling/assembling source code of a program into
executable form. In the beginning of IT industry this was realized by some dedicated
scripts written in *sh* or *bash*, and then quickly a tool called *make* was created. Simple
command line `make` executed an "instruction" written as *makefile*.

Then, in golden IDE era (90-ties and 2000-ties), a build process was connected with single key stroke - the 
IDE was responsible for finding all the sources in the project and compile them into executable form.
Automation capabilities derived from *make*-like tools were sort of forgotten.

But then, again, the *Continuous Integration* (in short: CI) term became a buzzword. The idea was very simple: a 
product should be build regularly (nightly or even after each repository push). The clear benefit
out of it is that all potential integration problems with the software can be detected very
early vs usually a night before deadline ;-) I don't want to replicate descriptions already existing there so
maybe go and get familiar with [this Thoughtworks page][4] to see why CI is so important.

So automation came back into the play, with dedicated software like CruiseControl, Hudson or Jenkins, and 
most recently a cloud-based Travis-CI and CircleCI. And again, we needed build scripts and build tools.
But this time we had a complete new generation of programming languages. In Java world we have Ant, Maven and
recently Gradle.

When I was back into 6502 assembly, I was especially lacking build tools. Of course I can use *make* to do
virtually anything, but I was sort of used to minimalism of Maven - almost nothing needs to be defined in
Maven build file to build standard Java project. I wanted something similar for KickAss.

## Why Gradle?
I don't like *make* because it is Unix-centric and I personally use both Linux and Windows based workstations.
Of course I can force anybody to install CygWin so that they can build with *make* using Windows machines.
Ant, which is great and portable, is also a bit old fashioned. It uses XML based syntax which does not
look very nice and user friendly.

Gradle is relative new but it is based on similar concept of tasks and it uses Groovy language to write
build definition file. So it is not only more readable than XML, but it also give you freedom to script what you
like including loops and ifs.

## How to use it?
Complete usage instruction is available on [GitHub][1]. The plugin as such is something that does not need to be
separately downloaded or installed. It is a regular Gradle dependency that can be reached from [plugins.gradle.org][6] 
portal. 

## References
* \[1\] [assembler-libraries][1]
* \[2\] [https://64bites.com/64spec/][2]
* \[3\] [https://github.com/64bites/64spec-cli/][3]
* \[4\] [https://www.thoughtworks.com/continuous-integration][4]
* \[5\] [https://github.com/c64lib/gradle-retro-assembler-plugin][5]
* \[6\] [https://plugins.gradle.org/plugin/com.github.c64lib.retro-assembler][6]

[1]: assembler-libraries
[2]: https://64bites.com/64spec/
[3]: https://github.com/64bites/64spec-cli/
[4]: https://www.thoughtworks.com/continuous-integration
[5]: https://github.com/c64lib/gradle-retro-assembler-plugin
[6]: https://plugins.gradle.org/plugin/com.github.c64lib.retro-assembler
