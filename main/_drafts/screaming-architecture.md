---
title: Screaming architecture
description: This is my first post here.
layout: post
categories: main
pic: /main/img/smell-of-code.jpg
---

> When you look at the top-level directory structure,
> and the source files in the highest-level package,
> do they scream "Health Care System" or "Accounting system"
> or "Inventory Management System"?
>
> Robert C. Martin,
> Clean Architecture: A Craftsman's Guide to Software Structure and Design

Architecture is set of rules that, when applied to your code, helps in keeping costs of your changes low.
These rules must be properly communicated to developers, maybe can be to some extend enforced either by proper tool (ArchUnit) or by carefully designed module system and dependency management.
But there is usually some level of freedom and flexibility that must be allowed otherwise project drifts into totalitarian mode of operation.

Even fully controlled, rigid architecture guards can be easily dodged.
Once I joined a project that has a fairly big codebase written in Java.
What I immediately noticed was an enormous number of static inner classes, scattered across the source without apparent reason.
The explanation was quite surprising: a chief architect from other country handed over his project to the new team but required explanation for each newly created Java class, usually requiring a removal as such claiming, that this class is not necessary.
Apparently team has noticed, that he just tracks newly created files, so if there is any way in Java to avoid creating them but still being able to create new classes...

So sometimes instead of hard enforcing maybe it is better to shape our architectural rules in a way, so that it suggests developers how the code should be created.

## The meaning of screaming
Lets try to be a developer now.
How developer gets its daily work specified?
He usually takes something like _user story_.
User story rarely says: "go to package `xyz` and change line 69 of class `abc`".
It rather says "extend _register new user_ use case with new kind of user.
We don't know how relevant classes are even named.
It can be `UserService` in package `logic`.
If I am new in the project, I would rather ask product owner to show me the view in UI when user gets registered.
Then I would look at the URL and track it down in the code to the relevant controller.
Later it gets pretty simple, just analyze code down to the right place where a proper `if` statement can be added.