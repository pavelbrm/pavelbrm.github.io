---
layout: post
title: "The Style Guide: Package Layout 5 – Cross-Platform Packages Part II"
category: Golocron
tags: [golang, m1u2]
---

> We know only too well that what we are doing is nothing more than a drop in the ocean.
>
> But if the drop were not there, the ocean would be missing something.
>
> – Mother Teresa

This is the second part in the series about designing cross-platform packages. It explores more advanced options for conditional compilation.


<!--more-->

![](/assets/m1u2_5.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Before We Begin

This post continues from where we left in [the last post](https://blog.pavelbrm.com/golocron/2021/04/17/package-layout-cross/). As a reminder, there we began a discussion about cross-plaform packages, and started with basic principles for cross-platform development, and continued with options for conditional compilation. The discussion stopped on the file name suffix option.

So here we continue the topic and talk about a more flexible way of managing compilation – build tags. Tags are helpful when code implements functionality for a few platforms which can't be expressed as a suffix on a file name, for example. They are also useful for conditional compilation in general.

Table of contents:
- [Build Tags](#build-tags)
  - [Basics](#basics)
  - [More Control](#more-control)
  - [Passing Tags](#passing-tags)
  - [Using Only Build Tags](#using-only-build-tags)
- [Using Suffixes and Tags](#using-suffixes-and-tags)
- [Notes](#notes)

I hope you enjoy.


## Build Tags

Build constraints in Go are expressed in a form of build tags. As you may already know, a build tag is a line comment that begins with `// +build` and lists the conditions under which a file should be included when compiling and/or testing the project.


### Basics

The following rules are in effect when using build constraints:
- multiple tags may be listed on the same line
- multiple lines with build tags may appear, one next to another
- tags may appear in any kind of source file (not limited to Go)
- constraints must appear close to the top of the file
- they may be preceded only by blank lines and other comments
- a series of build tag lines must be followed by an empty line (to distinguish from the documentation)
- all this means that in Go files, build constraints must appear only **before** the `package` clause.

There are also certain rules for writing and grouping build tags:
- when listed on the same line, space-separated tags are interpreted as `OR`
- if an option contains a comma, then it's evaluated as `AND` of its separated terms
- allowed symbols for a term are letters, digits, underscores and dots
- a term can be negated when preceded with an exclamation mark `!`
- when constraints are expressed as multiple lines, the overall result is the `AND` of individual lines
- a special tag of `// +build ignore` can be used to exclude the file from consideration.

Here are some examples:
- a simple build constraint

```golang
// +build linux,arm windows,386
```
It results to the following: `(linux AND arm) OR (windows AND 386)`.

- multiple build tags

```golang
// +build linux darwing
// +build amd64
```
It results to the following: `(linux OR darwin) AND (amd64)`

- with negation

```golang
// +build linux darwin,!cgo
```
Which results to `(linux) OR (darwin AND NOT cgo)`.


### More Control

The use of build tags is not limited to only platforms. Other conditions can be expressed in the very same way, for even more advanced control over the compilation. In the example below, we want to offer three tiers in the service – free, silver and gold. Each subsequent tier should include the features from the preceding tier. This is how it can be achieved on the compilation level which may help protect your app against cracking and/or bypassing licensing terms:
- define a base line, with no tags, in `base.go`
- define a the first paid tier, or `silver` level, in `silver.go`

```golang
// +build silver
```

- define a the second paid tier, `gold`, in `gold.go`

```golang
// +build silver
// +build gold
```

Notice that we have to list the both tags, `silver` and `gold` for the second tier, as we want it to include the features from the preceding tier. That's because of the way how the boolean logic is expressed using the constraints.


### Passing Tags

Now, how to tell compiler about constraints?

Build constraints are inferred from three places, two of which you already know:
- some environment variables, such as `GOOS`, `GOARCH`, `CGO_ENABLED`, etc
- the suffix of the name of a Go source file
- when explicitly passed as a special build flag of comma separated values when running `build`, `test` and other commands, as `-tag tag1,tag2,tag3`.

Note that the file name's suffix method is mentioned here. That's because, as said earlier, under the hood it works as a build constraint. When a file has a suffix that matches a valid supported operating system, architecture, or a combination of the two, the file is considered to have an implicit build constraint set to the values in the suffix.


### Using Only Build Tags

Having reminded ourselves what a build tag is, let's continue exploring options for cross-platform development.

As mentioned earlier, conditions for platform-dependent compilation can be expressed using file name suffixes. In a basic scenario, when the implementation details are different for, say, `windows` and `linux`, it's easy to express with `mypkg_windows.go` and `mypkg_linux.go`, in addition to `mypkg.go`.

However, what to do when two different platforms have the exact same implementations? One way is to simply have two separate files with same contents. For example, `app_darwin.go` and `app_freebsd.go`. But this kind of duplication is not something we normally want, unless we have to.

A much better approach is to use build tags for expressing conditions for the compilation process. It helps in making things cleaner and removes the need of duplicating the code. Add a build tag to the file, and give it a meaningful name that does not conflict with the supported operating systems, say `app_unix.go`:

```golang
// +build darwin freebsd
```

Now whenever the code is built on each of the mentioned platforms, or with the help of the environment variable `GOOS`, code will be included in binaries for either of the operating systems.


## Using Suffixes and Tags

Finally, let's consider another possible situation where a feature's implementation is different for some platforms, but is the same for others. The suffix-based method can be used in combination with build tags.

To illustrate this situation, let's assume the low-level implementation is different for Windows on the one hand, and Linux with macOS on the other, though the latter is the same for Linux and macOS. Then a reasonable approach would be to put the Windows-specific code into `myapp_windows.go` file, and let the suffix control inclusion of this code. For Linux and macOS, put implementation into `myapp_posix.go` and add the following build constraint at the top of the file:

```golang
// +build linux darwin
```

And that's it. The environment variables and/or build flags passed to `go build` or `go test` are now in control of what's included during the process.


## Notes

As you see, Go offers convenient tools for writing, building and shipping cross-platform software. Those who wrote programs for more than one platform in the past agree that Go has made huge progress in making the process easier. Those who haven't got cross-platform experience yet would probably not be surprised, if the first experience happened to be with Go. But then they would definitely notice the difference if faced cross-platform development with other languages.

These tools are great, but that's not enough on its own to produce great software and developer experience in short and long term. To achieve a good level of efficiency we need something else, and that's what the next section is about – actual advice on cross-platform packages.

---

To be continued...
