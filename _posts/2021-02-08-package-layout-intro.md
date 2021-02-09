---
layout: post
title: "The Style Guide: Package Layout 1 - Intro"
category: Golocron
tags: [golang, m1u2]
---

> Simple can be harder than complex:
>
> You have to work hard to get your thinking clean to make it simple. But it's worth it in the end because once you get there, you can move mountains.
>
> – Steve Jobs

With this post I continue publishing drafts of the book I've been working on since June 2020, Golocron – Software Development with Go. And this is the first post in a series of the new unit.


<!--more-->

![](/assets/m1u2_1.jpg)

<sub>_Photo credit: [Simone Hutsch](https://unsplash.com/@heysupersimi)_</sub>


## Before We Begin

I finished last year with the closing post for the first unit dedicated to project layout. If you have not read it yet, highly recommend you reading it throught. The posts of the unit can be found by [the m1u1 tag](https://blog.pavelbrm.com/tags/#m1u1).

But it's time to move on, and this post opens up a series dedicated to the second unit of the book. The unit is called "Package Layout", and, as the name suggests, it is about desiging packages. Here is what you'll learn from it:
- introduction to Go packages, and why it's important to get them right
- choosing a good name for a package
- structuring a package, and organising files in it
- designing packages in a cross-platform setup.

This post starts with a general introductin to the topic, and explains why it is important to design packages well in general, and in Go in particular.

I hope you'll find something interesting and helpful in this material.


- [Package Layout](#package-layout)
- [What is a Package](#what-is-a-package)
- [When to Create a Package](#when-to-create-a-package)
- [Keep Public API as Narrow as Possible](#keep-public-api-as-narrow-as-possible)


## Package Layout

Alright, as we now know how to organise a project, it's time to think about next layer in the application design. In Go, applications are built of packages which makes a package an important part of the overall architecture of a project. This unit focuses on guidelines for designing packages well.

A package is one of the core concepts in Go. A piece of code cannot exist outside of a package, so a package is created before anything else. Packages are **the** high-level building blocks for an application. They *provide* us with different abilities and tools to accomplish a task. Therefore, whether an application is well designed or not depends on how the packages are organised in the first place. As packages are the vital for the ecosystem, it's important to get this part right. A good package layout will be mostly unnoticed, as it feels just right and natural, whereas a poor layout can lead to many potential issues, and is disturbing in many different ways.

The goal of this unit is to provide guidelines on how to work with packages - create, structure, and maintain, including:
- when and why create a package
- what to put in it
- how to choose a proper name
- how to organise a package
- supporting code for multiple platforms.

Packages in Go are different from similar concepts in other languages. Since it's a modern language, it approaches code organisation in a unique way that takes into account what has and has not worked well, and introduces some new ideas. As a result, we get a powerful system that makes it easier to use, write and distribute software. Failing to acknowledge the difference from other languages, and to learn thinking about code organisation the Go way, are two of the major contributors to poor application design, issues with dependency management, leaked implementation details, lack of encapsulation and wrong abstractions, and so forth. When a package is designed in a wrong way, expect all sorts of problems.

What we want for out applications is quite an opposite. A coherent architecture, effortless dependency management, meaningful encapsulation, narrow and minimal public API, ease of maintenance, and so on. To get there, we continue with a package. Our next step is getting a good understanding of what it is.


## What is a Package

You may already know that a package in Go is defined as a collection of files in the same directory that are compiled together. A file contains variables, constants, types, and functions, and they're visible in other files within the package. Visibility of a symbol outside of the package is controlled by the first character in the name: a capital letter makes a symbol exported, and a lowercase letter marks it as unexported. A package in Go **provides** a way to accomplish a set of tasks.

While sounding simple, this definition gives us all that we need to lay out the foundation for a good software design. Having thought about what it really means, and how to employ it, you realise that the package paradigm in Go is meant to embrace and support many of the good practices in software engineering. The responsible engineer _knowns and applies_ the [SOLID design principles](https://en.wikipedia.org/wiki/SOLID).

So how does exactly the package system help us with software design? This is what packages help us to do:
- provide namespace and set the context
- group code by responsibilities, roles and functionality, and keep them clear (SRI), i.e. encapsulate what varies
- facilitate extension over modification (OCP)
- build and maintain proper abstractions (LSR)
- keep interfaces and public API narrow (ISP), i.e. hide implementation details and decoupling
- maintain healthy dependencies between objects (DIP)
- distribute and reuse code (DRY)
- keep it simple, testable and atomic (KISS)
- and so on.

What a Go package is not? The most important thing to remember is that packages are not means of/for implementing a hierarchy. The hierarchy of libraries in Go is **flat**. Bear this fact in mind. Each time when you encounter `net/http`, it does not mean that `http` is a descendant of `net`, or a sub-package. This is done simply as a matter of namespacing and setting the context, and no more than that. The `net` package is completely independent, and can be used on its own, so the `net/http` package is (although it does use `net` internally). In spite of being inside `net`, there is no hierarchical or inheritance relation between the two. The `http` package is _just_ located within the `net` package's folder.

Building a firm understanding of the packages' nature and purpose is one of the keys to designing changeable and maintainable software. Without this understanding, anything you do, even if it *seems* to be right, will eventually fall apart. It does not matter how well a house is built, if it stands on a weak and shaky foundation.

One of the most often mistakes is creating a package, not knowing when to and when not to do so. On one extreme, a project may have too much code, with too different roles mixed in a single package. On the opposite, a project may abuse the use of packages so that there is way too many of them. Neither of these situations helps in building great software.

You need to find balance, and knowing when it's a good time for creating a package is helpful.


## When to Create a Package

A package is created to **improve** the design and architecture of software. This is the single and only reason for the existence of a package. What does it mean? The software design principles help us in understanding this.

A package should be created to accomplish one or more of the following goals:
- encapsulation of varying parts of a project
- decoupling and setting boundaries between various parts
- implementing abstract and extensible data and processes
- providing reusable and reliable functionalities
- distributing code.

That is all about it. If a package exists, but its existence does not support at least one of the goals above, something is wrong with the design. Here are some reasons to question the package's existence:
- if a package does not make sense without another package (in a sense that it can't be used on its own)
- if a package is located inside another package, and can't be used outside it, and it's not an `internal` package
- if use of a package leads to an import/dependency cycle.

When planning a package, the following properties of code should guide the process:
- roles that objects in it play
- responsibilities that the code carries
- functionality it provides
- behaviours of objects.

Having clarified when to create a package, and before we can next steps, it's good to think about the lifecycle of a package, and how its design and contents influence the maintenance process.


## Keep Public API as Narrow as Possible

Keep the public API of a package as narrow as possible. Put another way, keep the number of exported objects and elements at a minimum.

To understand why this is important, we need to remember a few things. First, packages in Go are building blocks, and a unit of distribution. A useful library  found on the net is often a single package. A useful piece of software is made of a set of packages. Therefore, the API surface of a project is a superset of APIs of all packages in it. This makes it very important to design each individual package mindfully and consciously.

Second, in the vast majority of cases, it is the behaviour what software provides us with, not the concrete features. In other words, we're more interested in accomplishing a task by making the library do something we need, rather than in what fields are available on the library's objects. A library that exposes only behaviour, or behaviour and a minimum of features, is easier to both, use and maintain. Packages with lots of details exposed to the user tend to increase cognitive load: users are forced to think more than they really need to use the library, and authors have to keep in mind how each change would affect the user.

Third, one characteristic of a great software is that it is impossible or hard to misuse it. In other words, good libraries tend to minimise the risk of error by being designed and implemented in a way that helps to use it correctly. Such libraries also have minimal to no side effects, often secure by default, have and modest resource consumption.

Fourth, the only thing that is constant about software is CHANGE. Change means maintenance, finding and fixing bugs, changing implementation details for good, such as performance or security, adding new features, and more. As people use the software, they become more dependent on it. If a library does something valuable, it may become popular, and this is where things get complicated. The more people use the library, and the longer they do it, the harder it is to introduce changes, and the higher the risk of breaking others' systems. Even a simple update might take a long period of time to introduce, and even longer to be fully adopted by the users.

**When writing software, you need to plan and organise it in such a way as to maximise the ability to change as much as you can, as fast as you can.** In the context of package planning and design, this means that you need to keep the number of exported objects and elements at a possible minimum, and expose behaviour with a reasonable amount of features. This helps to:
- hide implementation details
- make maintenance easier for both, authors and users
- reduce dependency of the user's software on your software
- allow introducing changes faster.

At this point we know what a package is, when to create one, and that it's important to design it carefully. A more practical advice on how to achieve most of these goals in code will be discussed further in this and following units. The primary focus of the remainder of this unit is the structure of a package. We start with a very special one, `main`, and then survey guidelines for regular packages.

The next draft post will continue the discussion with the `main` package, and go deeper.


---

To be continued...
