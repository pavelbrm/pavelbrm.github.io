---
layout: post
title: "The Style Guide: Package Layout 3 - Structure and Files"
category: Golocron
tags: [golang, m1u2]
---

> Live life as though nobody is watching, and express yourself as though everyone is listening.
>
> – Nelson Mandela

One of the most important aspects of the package design is its structure. This post provides guidelines on organising files in a package in a coherent, sensible way.


<!--more-->

![](/assets/m1u2_3.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Before We Begin

Another topic that is often a subject for lengthy discussions during code reviews is file organisation of a package. It's often the case that developers without knowledge about code strutcure in Go are resistant or reluctant to the conventions and traditions the language has.

I've been constantly forming and shaping my views on the package design for last several years. It's originally [based](https://blog.pavelbrm.com/programming/2020/06/27/about-software-development-with-golang/#finding-the-signal) on the numerous books and articles, as well as reading others' codebases, and it has been continuously revised and improved by experience and practice.

In this draft, I summarise some of the learnings that, if applied, lead to a better state of each particular package, and an overall healthier codebase as a consequence.

