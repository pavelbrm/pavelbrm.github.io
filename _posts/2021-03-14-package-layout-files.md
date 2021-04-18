---
layout: post
title: "The Style Guide: Package Layout 3 – Structure and Files"
category: Golocron
tags: [golang, m1u2]
---

> Live life as though nobody is watching, and express yourself as though everyone is listening.
>
> – Nelson Mandela

One of the most important aspects of the package design is its structure. This post provides guidelines on organising files in a package in a sensible way.


<!--more-->

![](/assets/m1u2_3.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Before We Begin

Another topic that is often a subject for lengthy discussions during code reviews is the structure and file organisation of a package. It's often the case that developers without knowledge about code strutcure in Go are resistant or reluctant to the conventions and traditions in the language.

I've been constantly forming and shaping my views on the package design for last several years. It's originally [based](https://blog.pavelbrm.com/programming/2020/06/27/about-software-development-with-golang/#finding-the-signal) on the numerous books and articles, as well as reading others' codebases, and I've continuously revised and improved it in experience and practice.

In this draft, I summarise some of the learnings that, if applied, lead to a better state of each particular package, and overall a healthier codebase as a consequence.

Table of contents:
- [Structure](#structure)
  - [Flat by Default](#flat-by-default)
  - [Not a Sub-Package, But a Package in the Same Directory](#not-a-sub-package-but-a-package-in-the-same-directory)
- [Files in a Package](#files-in-a-package)
  - [Begin With a Single File Named as the Package](#begin-with-a-single-file-named-as-the-package)
  - [Organise Files by Topic, Responsibility and Behaviour](#organise-files-by-topic-responsibility-and-behaviour)
  - [Name Other Files Meaningfully](#name-other-files-meaningfully)
  - [Avoid Creating File per Object](#avoid-creating-file-per-object)
  - [Prefer Fewer Files of Larger Size](#prefer-fewer-files-of-larger-size)
  - [Keep Methods of a Type in The Same File](#keep-methods-of-a-type-in-the-same-file)
  - [Avoid Ambiguous File Names](#avoid-ambiguous-file-names)
  - [Use a Separate File for Documentation When Appropriate](#use-a-separate-file-for-documentation-when-appropriate)

I hope you enjoy. :-)


## Structure

The next most important aspect of the package design is choosing and maintaining a healthy structure.

Similarly to the project layout, the package layout either improves the overall quality of the package, application and project, or unnecessary complicates, causes variety of issues, and just makes it harder to develop and support code.

It's important to structure packages well, as fixing it up at a later stage requires significantly more effort. Writing software is the easiest step in its lifecycle – the better you do from the beginning, the lower the cost of its maintenance.

This section discusses the following guidelines:
- use flat structure by default
- there are no sub-packages in Go.


### Flat by Default

Most of the time, a package should have a flat structure, i.e. no structure at all.

That's it. As said earlier, do not introduce any structure for the sake of an artificial hierarchy. Several flat packages are better than some hierarchy when there is no actual need for that.

For example, when working on the `model` package, it does not make sense to have packages within it for each particular model.

| Do | Don't |
|:--|:--|
| `model` | `model/user`, `model/track`, `model/playlist`|

Here is what objects would look like for the first case:
- `model.User`
- `model.Track`
- `model.Playlist`
- etc.

Now compare it to an alternative, where everything is in a separate package. We would then get something more or less awful, for example:
- `model/user.User`
- `model/track.Track`
- `model/playlist.Playlist`
- an so forth.

However, there is a good reason for placing packages in a directory that has already got a package in it, – the context provided by a good name.


### Not a Sub-Package, But a Package in the Same Directory

There is no such a thing as a sub-package in Go. Instead, when it makes sense and is appropriate, a package can be created in a directory that already has a package in it.

If there is no hierarchical relation between packages, then what would be a reason for such a placement? Like said earlier, a good name sets the proper namespace and context. Therefore, it would be great if it was possible to benefit from using these for other code which fits nicely nearby the package inside the directory.

One of the best examples is the `net` package. Have a look at which packages reside in the `net` **directory** alongside with the `net` package, in the standard library:
- `http`
- `mail`
- `rpc`
- `smtp`
- `textproto`
- `url`.

While many developers are accustomed to think of them as `net/http`, `net/url` and so forth, the reality is that they're `http`, `url` and so on, respectively. What creates a feeling that the `http` package is a sub-package of `net`, is its import path. But that's only an import path. The package is `http`. And because the context and namespace set by `net` are appropriate and suitable for packages that deal with various aspects of networking, it makes a perfect sense to place them in that directory.

A real-life example of such placing would be a `handler/middleware` package. While `middleware` is a package on its own right, placing it under the `handler` **directory** is reasonable, because a middleware has to do with handling requests, and with handlers.

Use this sparingly and cautiously because of the risk of introducing a dependency cycle. The best way to avoid it is to have no shared code between packages at different levels. Design the relations such that dependencies always point into one, the same direction. For example, the package that is located at the parent directory can be imported by packages located deeper, but not the other way around. In some situations, it's reasonable to put all packages at the same level, by shifting what would be the top-level package into a child directory, so each package is in a sibling directory. Note that this is rather rare.


## Files in a Package

The Go package system is simple and powerful. It helps you prepare the foundation for the project; it is very convenient, and allows you to have short and concise names. There is no need to duplicate the information in the name of a package that is already being communicated by the path to the package and its location.

Up until now we've discussed mostly the ephemeral structure of a package - directories and naming. Code lives in files which a package is made of. How the files are named, how many of them, and how is the code split into files, are important and definitive aspects of the package design.

Since Go offers its own approach to packages, it's important to survey guidelines on how to put files in a package together. This section provides some advice on how to organise a package from the file perspective. And then the next unit further expands the topic.

Newcomers from other languages often don't know, nor do they learn the Go conventions, and use approaches that are unnatural and foreign to Go. Even those who envision themselves as seasoned Go developers, continue doing this unconsciously. It can be often found that a Go project looks much alike a PHP, Java, or C# one because the developer did not learn much about Go traditions and conventions. This leads to perpetuation of the problems that Go was created to solve.

Very few people learn the Go way of doing things, and even fewer do actually apply it in practice. You don't even need to open up a single file in a project to notice influence from other languages, and lack of good design. Countless files and poor naming are just a few examples of what could be improved.

A good design benefits from respecting and following Go-specific conventions. Some of the guidelines are:
- begin with a single file named after the package
- keep type's declaration and all its methods in the same file
- avoid creating file per object (type, function, etc)
- prefer fewer files larger in size instead of numerous small files
- when appropriate, move helpers in to a separate file
- when appropriate, use a separate file for documentation.

Let us look at each of the suggestions closer.


### Begin With a Single File Named as the Package

Begin with a single file that is named after the package.

In fact, this is two guidelines combined together:
- start with only one file
- name that file as the package is named.

A default tendency observed in many engineers is creating a taxonomy of files right at the beginning. Those files are named arbitrary, mostly poorly, due to behavioural patterns developed while working with other languages. This is unnecessary, and leads to a mess, not to mention it goes against the good traditions Go has established.

Instead, create just one file. Give it the same name as the package name. And then start adding some good code that you have thought about before writing.

As the package evolves, and new code is being added, it's reasonable to think about when it's the right time to add another file to the package. This leads to a deeper question – when to split code into multiple files?


### Organise Files by Topic, Responsibility and Behaviour

Organise files by topic, responsibility and behaviour that are implemented in them. This is **the** criteria for splitting up a file into two.

A package is started with a single file, and everything resides in that file. At some point, a new piece of functionality is about to be added, but that piece, while relating to the main theme of the package, is slightly different from the rest of the contents of the file. What to do?

The answer depends on what is going to be added. If it's a type which one of the existing types will operate on, then it's better to keep them in the same file. It's also good to keep as many of similar objects in the same file as it makes sense. If you have multiple helper functions that are used as `http.HandlerFunc`, then it makes complete sense to keep them in the same file. Don't create a file per function only because it's a new function.

Sometimes though it's better to split functionality into two files. For instance, you're working on a library, and there are two core parts to it - a server, and a client. It's beneficial to keep them separately, as their responsibilities and roles are quite different. But everything that has to do with either the server or client, should belong in the same file as the other type.


### Name Other Files Meaningfully

When adding files later, name them after the most important type declared in the file, or after the primary responsibility of the contents of the file.

In Go, we name files after the most important thing in it. If it's hard to identify what's more important, then name it after the functionality it provides, or responsibility it carries.

In other languages, a file is usually named after the class in it. While it's alright in Go, and is used often, this is not the only contributor to the name of a file.

Make sure to use a **single noun**. Refrain from using plural nouns or other types of words.


### Avoid Creating File per Object

Avoid creating a file per object (type, function, etc).

Please don't. Go is not Java, PHP, C# or any other language where creating a file per class is either a must, or a poor tradition. Go is also not C/C++ where you used to have a header and implementation file (we will mention this further).

Instead, follow the guideline about the criteria for organising files above. Roles, responsibilities, behaviours – these are the right factors indicating whether the contents needs to be split into files or not.

A file per type leads to the situations that have been considered in the previous sections and in the unit about project layout. Too many files complicate navigation, and increase cognitive load while working with a package, and so forth.


### Prefer Fewer Files of Larger Size

Prefer a fewer number of files of larger size instead of countless small files.

This is a natural consequence of the preceding advice. Fewer files is easier to navigate through. It does not produce clutter neither in the browser or terminal window, nor in the IDE. It's easier to organise and keep tidy.


### Keep Methods of a Type in The Same File

Keep the type's declaration and all its methods in the same file. Period.

One of the worst things one could do is split up the declaration of a type into multiple files. This is bad because of the increased effort required to maintain the mental map of a package, significantly reduced readability, and complicated understanding what the type does, its dependencies and responsibilities.

The two biggest problems with methods of a type scattered across multiple files are:
- understanding what the type is responsible for. You simply never know it until all files in the package are checked.
- making changes to the type, especially changing an existing field of a method. Same problem as above, exacerbated by the fact that you have to check all the files in advance, as the change should be planned and thought about before you implement it. And, if implemented without the full picture, you may discover that the change does actually not make sense because of the way it's used on other place.

So always keep everything that is part of the type declaration in the same file. Don't split the type declaration into several files. When a type is fully declared in a single place, it's easy to read and navigate through the code. The responsibilities and dependencies are easy to notice. It is also just easier to work with, literally.

As soon as you find such a type, the best thing that can be done is immediately gather the type declaration in the single file. It will save a lot of resources for present and not-so-far-future you, and everyone else working on the project.

There is one exception from this guideline, and we'll get back to it later. For now, it's sufficient to say that **the only single reason** for having some parts of a declaration in a separate file is implementing platform-dependent code. Otherwise, unless the exception is in effect, please don't.


### Avoid Ambiguous File Names

Avoid ambiguous names for files, such as `common`, `types`, etc.

The motivation is similar to the name of a package. Such names carry zero useful information about the contents. Our goal is to reduce energy required to understand what's happening in a package. Names like `common` complicate the task because it does not give any hint on what's in the file named as such.

The name `types` does not make sense as per the guidelines given above which recommend naming a file after the most important thing in it, and grouping contents by roles. It's impossible to have a file named `types` in a well-designed package. If you have one, you know what to do.


### Use a Separate File for Documentation When Appropriate

Use a separate file for documentation in a package which provides detailed explanations.

As you know, the top-level comment immediately preceding the `package` clause is the package comment. The first sentence **is** the package comment, and it can be optionally followed by a larger comment that is considered as a package-wide documentation.

Normally, the package comment must be written at the first file of a package. When you provide your users with detailed documentation, it would be good to keep it separate from the rest of code, but still within the package, so the Go documentation routines (`godoc`) are able to parse it.

There is a solution to this. Simply put the documentation along with the package comment in a file named `doc.go`. Normally and conventionally, the only content of this file is the package and documentation comments. Avoid adding anything else here.

```golang
/*
Package mypackage provides a convenient way to do X. <- the package comment

It achieves high performance by doing X using Y and Z.

Here is an example of using it.

Here is another one.

And here are some explanations about why is it valuable or any other useful information.
*/
package mypackage
```

Remember that this only makes sense for packages that have detailed documentation written in a specific way, i.e. the way `godoc` is able to understand. Otherwise, prefer more traditional places for documentation, such as  README files, documentation sites, etc.

Before we get to discussing how to organise a file internally in the lights of package design, there is one topic that is worth mentioning. The package layout is especially important when working on/with cross-platform software.

Next draft will tell you about package design and cross-platform development. Stay tuned in.

---

To be continued...
