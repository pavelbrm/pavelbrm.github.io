---
layout: post
title: "The Style Guide: Package Layout 2 - Naming a Package"
category: Golocron
tags: [golang, m1u2]
---

> People cannot change their habits without first changing their way of thinking.
>
> – Marie Kondō

This is the next piece of the Package Layout guidelines, and it tells you about choosing a good name for a package.


<!--more-->

![](/assets/m1u2_2.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Before We Begin

I've been planning on publishing this draft for a month, but could not quite get to it. Over past couple of weeks the question of naming packages popped up a few times in the code reviews I've conducted, and every time it happened I wanted I had had this draft published. Referring the material below in those conversations would have been helpful.

It's surprising how often in day-to-day work life you can see poorly named packages. The Go Team did a good job in early days of the language in spreading its views on how packages in Go were supposed to be named. However, in recent years, the information has not been on people's radars, and more and more developers just don't pay enough attention to whether the package they're adding is named well or not.

This specific section of the book is supposed to remind engineers about the importance of choosing a good name for a package, and to refresh what a good name means, especially in the Go world.

Here is what you'll learn from reading this:
- [Naming a Package](#naming-a-package)
- [Context and Namespace](#context-and-namespace)
- [What a Name is Made Of](#what-a-name-is-made-of)
- [Name the Directory After the Package it Contains](#name-the-directory-after-the-package-it-contains)
- [Choose Short and Concise Noun](#choose-short-and-concise-noun)
- [Avoid Common, Tools or Util](#avoid-common-tools-or-util)
- [Refrain From Using go and go- Prefixes](#refrain-from-using-go-and-go--prefixes)

My hopes are that you'll find helpful this material, and it helps you in better package naming.


## Naming a Package

The name of a package is one of its most important attributes. It impacts any place where it's mentioned, as well as code in the package. A good package name helps write expressive and understandable software.

Name packages consciously. The process requires thinking and planning. When packages are named well, the code using them is read naturally, and understanding of what it does is much easier. However, even a single poorly-named package in a project can significantly reduce readability, especially if it's an often-used one. The importance of a good name is so high because code is read more often than it's written. That's why you want to get it right.

This section goes into detail on choosing a good name for a package.


## Context and Namespace

The name of a package sets the context for its contents, and is the namespace. The package's context defines its meaning, role and responsibility.

When choosing a name for a package, make sure the name sets the right context, and is a good namespace. It's not a mere label, but a rather significant contributor to overall design, coherence, and readability of a project, no matter whether the project uses the package, or it _is_ the package.

Consider `net/http`. Its context is everything that has to do with the HTTP protocol. It provides everything that we need to either talk to a server on the net, or to serve requests from clients. From the context, it's clear that there are things like requests, responses, headers, and so forth.

But it's also a namespace, because the name of a package, unlike in some languages, is used as part of an expression or statement. For example, the `Request` type. When you see `http.Request` it's clear that this type **is** an HTTP request, not an arbitrary request.

The context of a package is important for both, its authors and users.

As the author, you're already in the package which means there is absolutely no reason to duplicate information that can be derived from the context and namespace set by the name.

It would have been a non-sense to write `http.HTTPRequest`, `http.HTTPResponse`, or `bytes.BufferBytes`. Nonetheless, way too many developers in the Go community  still and often do so.

If you think that that's only applicable to the standard library, here is another, a real-life example. Compare the two:
- `repository.UserRepository`
- and `repository.User`.

The first is absurdly verbose and repetitive; the second is clear, concise and sufficient. The first is legacy coming from languages like Java or PHP, and alludes a poor design, and ignoring Go conventions, or not understanding them. The second is more idiomatic and Go-fashioned. Another example is `handler.AuthHandler` (too verbose, noisy) and `handler.Auth` (much better).

For a user, the name of a package is part of code, of every call and expression in which the package is used. Thus, a good name helps in making code easier to write, but most importantly – read and understand. If the author of a package has put a good thought into the name, it improves the way code is perceived. Otherwise, each interaction with code would take more effort than it should; if you work with such code repeatedly and constantly, you'll soon realise how much more mental energy it takes, and how harder it is to maintain it.

A bad name leads to poorly-named objects, such as types, functions, methods, etc, which leads to a poor package API, which in turn leads to poor code that uses the package. Such code is legacy from the first symbol. This is how the code you write impacts others' codebases. And similarly, this is how your codebase is impacted by someone else's code.

Because of this, take the time and think it through before submitting, as an author, or accepting, as a user or reviewer, code with a new package. Think about your future self, and those who will use the package and maintain it, and code based on it. Only when the name and the contents of a package are balanced and play well together, merge or approve the pull request. Merged code is legacy, and you want it to be good.


### What a Name is Made Of

Before introducing guidelines on naming a package, it's important to understand what does create a name, technically.

Two things do contribute to the name of a package. The first thing is the name of the directory where it's located. The second is the `package` statement, the first non-comment line of any `.go` file in Go. Needless to remind that the directory name can contain hyphens, whereas the value in the `package` statement cannot.

The value specified by the `package` statement **is** the package name. The directory containing a package can be named arbitrary and unrelated to the package name, though it does not mean it's a good idea. Not all of what's possible is worth doing.

While this opens up an infinite number of choices for naming, it's better to stick to the conventions in the community.


### Name the Directory After the Package it Contains

Use the same name for the directory containing a package, and for the package itself.

If your package is named `rpc`, then the directory in which it's located **must** be named `rpc`.

So many projects and libraries violate this simple guideline. Don't confuse your future self, your colleagues and users, and use consistent naming between packages and directories.


### Choose Short and Concise Noun

The name of a package must be a **singular, short, concise and expressive** noun or an acronym, made of lower-case letters.

| Do | Don't |
| :-- | :-- |
| `handler` | `handlers` |
| `storage` | `dataStorages` |
| `http` | `http_something` |

This rule, as any other, has an exception. The name _can_ be a plural noun **if and only if** it provides tools that work with things that can be described as a plural noun. Good examples are `bytes` and `strings`. They are named so not because they contain types that implement bytes and strings. It is because they provide tools for manipulating bytes and strings.

An example of the exception might be a hypothetical package `files` which provides tools for working with files in various ways. But even then, such a package might be questioned since its contents may belong in other packages. And the recent addition of the `io/fs` package proves it. The `fs` name stands for `filesystem`, and encapsulates a variety of tools for dealing with it.

So remember, a good package name is:
- singular
- short
- concise
- precise.


### Avoid Common, Tools or Util

Avoid common and vague names such as `common`, `tools`, `util` or similar.

This is may be one of the most often repeated mantras in Go, yet often a package like this is found in codebases.

Recently, even a previously considered as "okay" suffix `util` has become obsolete with the release of Go 1.16. The Go Team deprecated the `io/ioutil` package in favour of the `os` and `io` packages. So there is no more excuse even for your `dbutil` package, should you happen to have one, because why would it exist, if:
- it can be called `db`
- if `db` already exists, what does prevent you from adding code in it?


### Refrain From Using `go` and `go-` Prefixes

Refrain from using the `go` and `go-` as prefixes or suffixes for a library, project, directory and the package name.

Does it sound controversial? Maybe. But why do we need to prefix anything with the name of the technology it's based on? This simply does not make sense.

The Go package and module system provide everything to guarantee uniqueness of names. It's also impossible to use a library named `somelib` written say in Java. So why would you need to name a Go implementation of anything as `go-somlib` or `gosomelib`? Find a better and unique name instead.

There are several issues with using `go` as a prefix/suffix:
- it adds zero useful information
- if it contains a hyphen, then the library and package it contains have different names, which is confusing. The package inside a `go-somelib` library  is `somelib`, not `gosomelib` or `go-somelib`.

If there is a strong need, desire or other reason to have `go` in the name, consider using it in the account/organisation name. The three most popular services for code collaboration and sharing (Github, Gitlab and Bitbucket), as well as self-hosted solutions (like Gitea and Gogs), use the account as a namespace for projects. Just compare:

| Do | Don't |
| :-- | :-- |
| `github.com/golib/orion` | `github.com/someone/goorion` |
| `gitlab.com/go-lib/crux` | `gitlab.com/someone/go-crux` |
| `bitbucket.org/someone/lyra` | `bitbucket.org/gosomeone/golyra` |

It's easy to see that the examples in the first column are shorter, cleaner, and nicer. This is what we want from names.

One exception needs to be made. It's easy to imagine a situation when an organisation has versions of the same library implemented in a number of distinct technologies. And Go might not be the first of them, and the most straightforward name might have already been taken. In such case, there are just a few options available:
- get creative, and use a different name. Many companies do so, and experience has shown this to be one of the best options. This guarantees uniqueness.
- or, add a suffix to the repository name, but not to the module name. For example, Twitch has `twirp-ruby`  in addition to Go's `twirp`.

| Good | Acceptable |
| :-- | :-- |
| `github.com/myorg/thau` | `github.com/myorg/thau` |
| `github.com/myorg/theta` | `github.com/myorg/thau-go` |
| `github.com/myorg/omega` | `github.com/myorg/omega-cpp` |

With this exception, it's still possible to:
- have a not so different name of a package/module/library
- sort repositories by the name
- unify naming across an organisation.

Having figured out how to name a package, let's see how to better organise it.

---

To be continued...
