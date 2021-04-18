---
layout: post
title: "The Style Guide: Package Layout 4 – Cross-Platform Packages Part I"
category: Golocron
tags: [golang, m1u2]
---

> The most important thing is to read as much as you can, like I did.
>
> It will give you an understanding of what makes good writing and it will enlarge your vocabulary.
>
> – J. K. Rowling

This draft is the first in a series about designing packages in the cross-platform context. It covers some basic theory and tells about the simplest way to control the Go compiler depening on a platform.


<!--more-->

![](/assets/m1u2_4.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Before We Begin

It's been a while since the last post. Several past weeks at work have been really intense. I was also busy in my personal time, so it was hard to find an hour to prepare and publish a draft. Nonetheless, here goes a new post with its contents written about a month ago or so.

As said in the first sentence, this post is the first in a series covering package design in the lights of developing cross-platform software. We begin with some basic principles and terminology, and then turn to the practical side of the question. Here we consider the simplest way to control the compilation process, and the following drafts will go deeper, and show a few more cases.

Table of contents:
- [Cross-Platform Code](#cross-platform-code)
- [Basic Principles of Writing Cross-Platform Code](#basic-principles-of-writing-cross-platform-code)
- [Cross-Platform Options in Go](#cross-platform-options-in-go)
  - [Terminology](#terminology)
  - [Compiling For Multiple Platforms](#compiling-for-multiple-platforms)
  - [Developing for Multiple Platforms](#developing-for-multiple-platforms)
  - [The File Name](#the-file-name)

As usual, I hope you learn something new and useful from this material. ;-)


## Cross-Platform Code

The guidelines for designing packages would have been incomplete without considering writing software in which implementation details depend on the platform the software was created for. This brings us to the topic of cross-platform development in Go, from the package design perspective.

Writing, maintaining and deploying cross-platform code has never been as simple as it is with Go. It's not to say it's easy, but it has become much easier and simpler than it was before Go had come. The language provides us with built-in support for cross-compiling for multiple platforms so we don't even need to run the target system to build a binary for it. Tooling is excellent in this area, compared to other languages.

However, working on a cross-platform project is different from a single-platform one. All design and architectural decisions now are multiplied by the number of platforms you support. The need to properly test code and prove its correctness sets higher requirements for the design of services in the project.

Depending on the practices used to develop such a project, cross-platform experience might be as usual and smooth as single-platform, or it might turn into a nightmare. It is important to understand some basic principles of writing cross-platform code. The principles tell you what tools when to use. Careful planning, cautious and mindful approach to implementation are also necessary to avoid many pitfalls.

This section aims to aid you with advice on working with cross-platform code from the package organisation perspective. We will consider the following aspects of developing packages for multiple platforms:
- Basic Principles of Writing Cross-Platform Code
- Cross-Platform Options in Go
- Package and File Organisation.


## Basic Principles of Writing Cross-Platform Code

_Note: The content below is relevant to some other languages, though here are talking mainly about Go. A reasonable amount of simplifications is in place. Pay attention, and **think** about what you've read._

In one sentence, writing cross-platform code can be almost indifferent from writing for a single platform, **if** you've done the homework, i.e. have learned and followed good design and architecture principles and rules. The principles we're referring to are SOLID, some of the design patterns and common sense. Otherwise, if the good practices are neglected, chances for success are dramatically lower, or even zero.

As a general rule of thumb, all **the business logic must be platform-agnostic**. It is the implementation details what varies from platform to platform, but core algorithms remain common for all of them. This is important because even two platforms significantly increase the complexity just because all potential outcomes are now squared. Any design decision should be aligned with the main goal – reducing entropy in the software.

The starting point, and an obvious way to reduce entropy, is to **keep the amount of cross-platform code at a possible minimum**. The less code is different between platforms, the easier it is to guarantee its correctness, maintain and test it.

With the two principles applied, the likelihood of a project ending up in a better shape is higher. Though it's still possible that the project might not be able to achieve its goals due to specifics of one of the supported platforms. If it is the case, there are essentially two potential scenarios. One is trying to maintain all features on par for each platform. While this is the ideal and desired situation, it may often lead to waste of resources since not everything is equally possible across available platforms.

The other possible scenario is a reasonable and pragmatic trade off. What helps in staying efficient is to **know when to mock missing cross-platform features or skip them at all**. Not every system feature has something similar in another system.

For example, there is no direct alternative to Windows's Registry in Linux or FreeBSD. Of course, in Unix, Linux-based systems and its derivatives we have a kernel and the `sysctl` API or similar, but that's not exactly the same thing is the Registry (One may argue that `systemd` has been working hard to become a Linux counterpart of the Windows Registry). Trying to mimic its behaviour would be tedious and unjustifiably complex. In such case, two potential alternatives are available – one is to abstract the business logic such that it does not depend on a particular system feature, and the other is to skip one system that does not support some functionality that is important in another system.

To get a sense of how implementation details can vary across many platforms, have a look at [the list of differences in the `net` package](https://golang.org/pkg/net/#pkg-note-BUG) in the Go standard library.


## Cross-Platform Options in Go

One of the key strengths of the Go programming language is its powerful tooling focused on developer's productivity. Fast compilation and ease of dependency management are vital for the ecosystem. But tools for writing and building multi-platform code are what make Go truly special.

Writing and maintaining cross-platform software (without distribution) is a two-stage process:
- the code needs to be written in a way that allows changing its behaviour based on the platform it's running on
- the code then needs to be compiled for each supported platform.

We talk about the second stage first, and then dig deeper into the first.


### Terminology

Before we get too far in the discussion, it's good to agree on the terminology used in next sections.

In this work, as well as in the community, we use the following terms with the corresponding meanings:
- Go operating system, or simply operating system, as per the environment variable `GOOS` – one of the target operating systems Go supports. Expressed as one of the predefined values.
- Go architecture, or simply architecture, as per the environment variable `GOARCH` – one of the the target system architectures Go supports. Similarly to the operating system, it's one of the predefined values.
- Go platform, or just platform, a combination of the two, an operating system and architecture, in this order. This term is not used in code or as a setting, but is used in discussions and documentation. When you see a use of the term "platform" in a conversation that is related to cross-platform software in Go, it means a combination of an operating system and architecture. For instance, these sentences are written on the `darwin/amd64` platform.

It's worth reminding that each of `GOOS` and `GOARCH` and, therefore, the platform, can, and usually is, defined implicitly by inferring the value from the environment, if not set explicitly. When you're running `go build` on your Intel-based Mac without setting the environment variables, `GOOS=darwin` and `GOARCH=amd64` are supplied to the compiler. In this case, the target platform is `darwin/amd64`. On a Raspberry Pi under Linux, it would be `linux/arm` and additional `GOARM`. Similar effect can be achieved on any other platform with variables explicitly set as follows `GOOS=linux GOARCH=arm GOARM=6`.

While we're on this topic, it's no harm in repeating that when only one of the variables specified to a value that is different from the current system value, the other is inferred from the current system. For the rest of the book, if not mentioned explicitly, the implied value of `GOARCH` is `amd64`.

Okay, now we can move forward. The next section is a quick refresher of the compilation process, and then we dive into creating packages that support several platforms.


### Compiling For Multiple Platforms

Even if a project has no code that depends on a platform, the binary format has to match the requirements and expectations of a target system. In Go, there are two options for compiling code for more than one platform:
- simply building on each of the target platforms
- cross-compile for each of the supported platforms.

With the first option, the code for each of the supported platforms is built on that platform. In its simplest form, the build process looks no different from what we do every day – `go build` is `go build`, after all. What differs is the artefact which is specific to the platform it has been built on.

The second option offers a slightly different approach. The result is controlled by a pair of well-known environment variables that tell the compiler what it should produce. This process is also known as **cross-compilation**:
```bash
GOOS=freebsd GOARCH=amd64 go build -o myapp-freebsd-amd64
GOOS=linux GOARCH=arm go build -o myapp-linux-arm
GOOS=windows GOARCH=386 go build -o myapp-windows-386.exe
GOOS=darwin GOARCH=amd64 go build -o myapp-darwin-amd64
```

Which of the two options when to use? It depends on your environment, priorities, available resources and limitations. The native compilation option is the easiest from the tooling perspective. You simply run the same set of tools to test and build code, and get the results. A major downside is the need to run and maintain instances of all platforms, which might be expensive as it requires some (potentially, significant) additional resources (i.e. costs you time, attention and money).

On the other hand, the second option is less demanding to the build infrastructure. All code is built on the developers's machine or in a single instance of CI/CD. Platform-agnostic code is tested on the most available platform, but running tests for platform-specific code still requires native instances.

So choose wisely. The guideline is to use cross-compilation for producing release artefacts, and to use specialised, automated test environment to run tests against platform-dependent code.

It's worth noting that, in its simplest form, a cross-platform program has no differences in the implementation between supported platforms (not counting potential differences in the standard library). In such case, all what is needed is compiling a binary for each platform. That's exactly what we've just covered.

Compiling code for multiple systems is a relatively easy step. But to build something, we have to write it first. The question is how to do it, and do it in a right way.


### Developing for Multiple Platforms

When working on functionality which involves special features of an operating system, a specialised implementation is required for the varying part. Then, different implementations need to be incorporated with the platform-independent business logic of the project.

Go offers **build constraints** to enable conditional compilation. This allows to write specialised code for a particular platform, and guarantees that the code is included and available only when compiled for that target system. For all other platforms, which are not covered by constraints, such code simply does not exist.

Build constraints can be expressed in two ways:
- using the file name
- using build tags.

When compiling, the two environment variables `GOOS` and `GOARCH` and/or the build flag `-tags` are used to control the process. Specifically, these settings tell the compiler when and what to include in the resulting binary.

As you will learn or remember soon, while the two ways under the hood work in the same way (due to the `go/build` package), in everyday development each of them has own pros and cons. They can be used independently in simple scenarios. More often though, they're used together as it allows more flexible behaviour, but sometimes it's just the only way to express the constraints. Let's briefly recap what each of them is, and then learn how to employ them to make the design of a package better.

_NOTICE:_ The contents below covers topics such as using build tags and file names for conditional compilation. The text has been written when Go 1.16 has been released. Everything below applies for Go versions _up to 1.16_. Starting from version **1.17**, things will change. There is [a proposal](https://go.googlesource.com/proposal/+/master/design/draft-gobuild.md) which has been [accepted](https://github.com/golang/go/issues/41184). This is going to be quite a large language change, on par with introduction of Modules. I will keep this part of the unit updated as the new behaviour is available, and it's clear what happens to the existing approach. For now, we focus on the existing way of managing compilation.


### The File Name

We start with the method based on the file name suffix. If a source file has a suffix that is a valid operating system, architecture or a combination of the two, then the file is compiled only for that platform.

Here are some examples, for a package `mypkg`:
- `mypkg_linux.go` is included in the binaries for Linux and any architecture (of course, for those that are supported by Go)
- `mypkg_amd64.go` is included in binaries for any operating system running on the `amd64` architecture (also known as `x86_64`, `Intel 64`, but these are not valid names in Go)
- `mypkg_windows_386.go` is included in the binary only when targeted at x86-based 32-bit Windows
- `mypkg_posix.go` is included on any platform (remember, `platform = os + arch`, hence on any os and any arch) as `posix` is not a valid operating system nor architecture identifier for the Go toolchain.

_When_ to include a file is controlled by the environment at the build time:
- when running `go build` the values are derived from the system the build is running on, i.e. your machine or the machine the command is executed on
- or when `GOOS` and/or `GOARCH` are explicitly set.

This also works with testing. To write a platform-specific test you simply add the familiar `test` suffix to the end of the corresponding file name or to any file that will include such code:
- `mypkg_linux_test.go` is included only when running `go test` on Linux, or `GOOS=linux go test` on any system
- `mypkg_amd64_test.go` is included only when running `go test` on an x86-based 64-bit system, or `GOARCH=amd64 go test` on any other architecture
- `mypkg_windows_386_test.go` is included only when running `go test` on an x86-based 32-bit Windows, or `GOOS=windows GOARCH=386 go test` on any system
- and so on.

For all available GOOS and GOARCH valid values and combinations please visit [this page](https://golang.org/doc/install/source#environment).

As you see, the approach offers an easy way for separating code for a particular system. It's enough in simple cases or when code is different between all the systems your project works on. But what if a more fine grained control is needed? Or what if code is the same for Windows and Linux, but is different for macOS?

Another use case is including/excluding code at the compilation time based not only on a platform, or even not on the platform at all, but on a custom criterion. For example, you may want to include some features in the free version of your product, and other features only in the binary shipped to the paying users. You may even have multiple paid tiers with three sets of features, and the corresponding binaries should include all features from the preceding tiers plus something else. Or you may even want to offer different implementations of the same feature based on the tier?

The answer to these and many other questions about conditional compilation is build tags.

---

To be continued...
