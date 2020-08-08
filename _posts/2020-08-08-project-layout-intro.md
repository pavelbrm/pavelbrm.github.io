---
layout: post
title: "The Style Guide: Project Layout 1"
category: Golocron
tags: [golang, m1u1]
---

> All of life is a constant education.
>
> — Eleanor Roosevelt

This post is opens up a series of the draft contents for [the book](https://blog.pavelbrm.com/programming/2020/06/27/about-software-development-with-golang/) I'm currently working on.


<!--more-->

![](/assets/m1u1_1.jpg)


## Before We Begin

As mentioned in the introductionary post, the book will include three to four modules, and each module is composed of a few units. I will publish the drafts as soon as they're ready. Normally, an article will contain several sections of a unit.

We start with the first module which is The Style Guide. The main goal of the Guide is to provide you with advice on working with projects in general, designing packages, organising contents, and, of course, writing neat and readable code.

This article introduces the first unit of The Style Guide - Project Layout. The unit tells you about how to design the project layout well. The article starts with the introduction, then describes the differences between good and bad layouts, and closes with some details about types of layouts.

- [Introduction](#introduction)
- [Good and Bad Layout](#good-and-bad-layout)
  - [Bad Layout](#bad-layout)
  - [Good Layout](#good-layout)
- [Types of Layouts](#types-of-layouts)

I hope you find something interesting and useful.


## Introduction

Starting from a clean slate is always a pleasure when it comes to a new software project. There is no legacy _yet_, no technical debt and any other sources of anxiety. The pleasant experience can be maintained at a high level throughout the project's lifecycle. One of the key contributors to the experience of developing and maintaining a project is its layout, the file and folder structure and organisation. In the world of Go it's extremely important because a structure can be both helping and limiting, depending on how it's implementated.

This unit focuses on the project layout design. A layout design is how the folders and files are organised in a codebase, and thus in the repository containing the codebase. A layout sets the stage for all further design and architecture decisions for a project, such as package layout, logical and functional grouping. It also affects maintenance routines like producing build artefacts, CI and CD, rolling out and shipping to the user.

Depending on whether a layout is good or not, it may either significantly help and simplify everyday tasks, or it can make it quite complicated and poor. A good project layout helps you leverage and benefit from techology, whereas with a bad design any change to the structure is more of a fight with technology.

The importance of creating and following a good layout is clear. You create a project once, and this operation is technically quick and easy. But you have to work with the project until it's sunsetted, and this may and should last for years. The ease of creating a new folder hides the importance of putting a thought into how to set a project in a way that's both helping, convenient and makes sense. It's better to take some time to think about how this is going to look like in a week, a month, half a year, and two years from now. If this is done _before_ anything is created, then working on the project will be much easier.

The rest of the unit covers the following:
- Good vs. Bad project lauout
- Layout for a library
- Single application layout
- Layout for a monolithic, multiservice repository.

Now, let's figure out what makes a project layout good and bad.

_Disclaimer:_ Terms "good" and "bad" are used as simple words to describe something that feels right, positive, helps and supports, as opposed to something that is less right, inconvenient, slows down and distracts. They're used not as judgements or labels, but as synonims for respective broader terms.


## Good and Bad Layout

You can't know if something is good until you learn what a similar bad thing looks like. A bad thing might be percieved as good _until_ you experience a trully good thing. Take, for instance, coffee. For someone who has never had it, almost any coffee can be considered as coffee, and it may even be unquestioned. But when after trying various kinds you take a sip of Supreme coffee made well, you no longer think that coffee from a nespresso machine is tasty.

So, what makes a project layout bad? Can it even be bad? Why should we bother about it?


### Bad Layout

Have you ever been supporting a project with so long list of files scattered in the root directory, so it takes you several scrolls to get to the README file on the project's repository homepage on Github?

Have you ever been maintaining a project where half of the application is contained in the `main` package, and the entirety of the `main` package resides at the root level of the repository?

How often do you find unnecessary artefacts in a repository after building an application or running tests?

How often did you need to replace or move parts of a project and then fix the imports to fix an import cycle, or when the stucture no longer makes sense?

How often do you find a folder that exists only to contain another folder?

Positive answers to any of these and other similar questions are signs of that you've encountered a bad project layout. When working with such a project, even as simple operation as building a binary can turn into a trouble since you don't know whether the resulting binary will be accidentally included with the next commit or not.

So what are the characteristics of a suboptimal layout anyway? It can be nearly anything that leads to any of the following symptoms:
- a long list of files scattered at the root or any other level of a project
- a long list directories scattered at the root or any other level of a project
- files of different purposes are mixed at the root or any other level
    - code and configuration files (the app's configuration files)
    - code and CI scripts
    - code and tool's configurations
    - images or any other content
- there is no definite place for outputting build artefacts
- `.gitignore` does not exclude anything that should not be committed
- the structure does not support or reflect architecture of a project
    - having both `app` and `server` directories at the same level - does this project provide an app? Or a server? Or it's a poor project layout design?
- the project structure leads to leakage of implementation details to the public API surface level
    - users can use parts of the implementation that are supposed to be internal
    - thus maintenance of the project is now complicated and slowed down since the public API is much broader than it should be
- the project structure is unnecessarily complicated and contains a multi-level hierarchy that implements a taxonomy of packages

A mixture of even two of the symptoms listed above can significantly complicate project development and maintenance in long-term. Even worse may things become if a project is large, and is being developed by more than one person. Suffering can be endless, if that's an open-source project, and it's a popular one.

So you really want to invest in a good project structure that will help and support you as the application evolves. What does make a good layout?


### Good Layout

Now as we have learned some of symptoms of a bad project layout, it's time to know how a good one looks and feels like?

Overall, a good structure should be unnoticeable in everyday work. It exists, but as if it was invisible. Once defined, described and set, it just guides you. It limits the number of mistakes, helps to onboard new developers, supports maintenance routines, and does help in many other different ways. Let's look at some of those.

A good design for layout suits well for a project:
    - if a project is a library, the layout should reflect and _support_ that
    - if a project is an application, then the layout should make development and maintenance easier
    - if a project uses the monolithic repository approach, it's clear where different services, libraries and other parts reside, and where to add new things.

A good layout limits the chances of misplacing something:
    - when files and directories are logically and functionally well organised, it's harder to misplace a new file
    - the structure helps in deciding whether a new file should be a part of the existing tree, or it deserves a new level.

A good layout reduces the mental load required to work with a project day by day:
    - it's clear where binaries should go
    - test coverage reports do not pollute commits
    - changing CI configuration is straightforward
    - files of different purposes are grouped in a way that makes logical and functional sense
    - it's hard to accidentaly commit and push garbage files.

A good layout simplifies navigation through a project:
    - it's convenient and easy to navigate in an editor/IDE
    - the homepage looks nice and well organised
    - in a terminal window, it's easy to work with files using standart and simple console tools.

A good layout helps you to maintain a project in a healthy state:
    - implementation details are kept private, yet still available as exported within a project
    - the possibility of creating an import cycle is reduced by design
    - the structure and hierarchy represent the business logic and relations between different parts of a project.

As you can see, a good layout is more than just the opposite of the symptoms of a bad one. A good layout is the foundation for a development process focused on producing great software. If you have it, than you've got more mental energy to focus on creative solutions for hard problems. If you don't, then part of the energy is wasted.

The layout of a project depends on many factors. The most important one is the kind of a project. The layout will be different for a library, a single application or service, and for a monolithic repository, while there are some commonalities that make better any kind of a project.

As we now have learned what makes a good layout, it's time to know how to structure projects of different types.


## Types of Layouts

The layout of a project depends on its type and purpose. The layout should suit the project well to help supporting it.

A good way to describe the importance of using a good layout is an analogy with clothes. We all have things that suit us well; in this case the clothes fit us naturally so we don't even notice its presence. When a thing doesn't suit well, we might not notice it too, but by the middle of a day we've experienced more stress; when the garment is taken off at the end of a day, it feels much better.

The same is true for a project. Its structure may be unnoticed because it's natural and convenient, whereas with other projects we sometimes can't recognise the source of anxiety. That anxiety, in fact, is a sum of many factors, and one of them might be an incovenient structure of a project.

A project is created once, and is maintained until its end of life. This means that you really want to invest in a good structure so interactions with codebase are on the positive side, and facilitate productivity.

All projects are different, and there is no a hard set of rules one can follow without thinking about a particular use case. Yet we can confidently say that the majority of projects fall into one of the following three categories:
- libraries (packages, modules, frameworks, you name it, depending on the language and environment)
- single services/applications (web services, CLI tools, GUI applications, etc)
- all-in-one, or monolithic repositories (multiple services/applications and libraries).

Some recommendations for different types of projects can be applied to all of them, and some are specific to a particular use case. For example, a library layout will be slightly different from a monolithic one.

By a project we mean a few related and similar things at the same time:
- the directory containing a project
- the repository containing a project.

In the modern world it's hard to imagine a project that is not published on some service for collaboration and synchronisation with other people. Even if a developer is working on a project alone, they still need to keep it in sync between different devices, and to make sure the code is not lost, if something happens to a computer.

Given that, the section assumes that some service is used to store code of a project. While there are many of them, for simplicity we assume it's Github.

The rest of the section focuses on describing guidelines; we start with the commonalities, and then consider aspects that are different. Because it's not hard rules, but rather suggestions, they're flexible, and will differ from project to project. There are also some exceptions, which is natural for any healthy-sense driven guidelines.

_Disclaimer: Neither the author, nor this work are affiliated or benefit in any way from Github._


---

To be continued...
