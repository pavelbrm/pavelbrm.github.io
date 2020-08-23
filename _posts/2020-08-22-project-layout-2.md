---
layout: post
title: "The Style Guide: Project Layout 2"
category: Golocron
tags: [golang, m1u1]
---

> Out of clutter, find simplicity.
>
> – Albert Einstein

The second post in the the series of drafts of the book continues the topic started [recently](https://blog.pavelbrm.com/golocron/2020/08/08/project-layout-intro/). It introduces guidelines for laying out a project which are common for various kinds of projects, and then considers guidelines for a library and single application.


<!--more-->

![](/assets/m1u1_2.jpg)


## Before We Begin

This post continues The Project Layout unit of The Style Guide. Here we talk about some common ways of making a project easier to work with, and then go into some specifics of working on a library and single application projects.

Most of the advice in this piece is language-agnostic, some suggestions can be applied with a slight adjustment by projects written in something different from Go, and a few are unique to Go.

- [Common](#common)
  - [Provide Files Required for a Project](#provide-files-required-for-a-project)
  - [Keep the Number of Root Elements at Minimum](#keep-the-number-of-root-elements-at-minimum)
  - [Keep Any Level Clean and Up to Date](#keep-any-level-clean-and-up-to-date)
  - [Don't Create Artificial Hierarchy](#dont-create-artificial-hierarchy)
  - [Group Non-Code Files by Their Purpose](#group-non-code-files-by-their-purpose)
  - [Provide the Makefile](#provide-the-makefile)
  - [Store Credentials Somewhere Else](#store-credentials-somewhere-else)
- [Library](#library)
  - [Provide a Good README File](#provide-a-good-readme-file)
  - [Keep the Top-Level List of a Manageable Size](#keep-the-top-level-list-of-a-manageable-size)
  - [Provide Examples](#provide-examples)
  - [Provide Benchmarking Results](#provide-benchmarking-results)
  - [Examples](#examples)
    - [Good Examples](#good-examples)
    - [Can Be Improved](#can-be-improved)
- [Single Application](#single-application)
  - [Use `cmd` Directory for Entry Points](#use-cmd-directory-for-entry-points)
  - [Use `bin` Directory for Binaries](#use-bin-directory-for-binaries)

As usual, my hopes are that you find something interesting and new from the post.


## Common

The ultimate goal of choosing an appropriate structure is to keep things nice and tidy which simplifies interactions with a codebase.

This includes:
- providing required files for a project
- keeping the number of elements at the root level at a reasonable minimum
- keeping the root (and any other) level clean and up to date
- grouping non-code files by their purpose
- refraining from creating unnecessary hierarchy of folders
- having a single entry point for routine tasks
- storing credentials somewhere else

Let's talk about these in more detail.


### Provide Files Required for a Project

A project should contain files that describe its purpose, provide required information for the programming language and tools, and simplify its maintenance.

The following files are considered as required:

| Name | Description |
| --- | --- |
| `.gitignore` | Provides rules that govern what's allowed to be committed. It helps keeping a repository free from garbage files. |
| `go.mod`, `go.sum` | Identify a Go module. |
| `README.md` | Provides the description and any useful information and documentation for a project. |
| `LICENCE` | Provides clarity on licensing for a project, and helps to avoid potential legal inconveniences. |
| `Makefile` | Provides a single entry point to all maintenance and routine tasks. |

The files listed above are a bare minimum, but required for a modern project. You do this once, but if it's done right, it helps you all the way further.

One thing to note is that even with no code at all, a repository is likely to contain about 6 files. This is why the next suggestion is important.


### Keep the Number of Root Elements at Minimum

The number of folders and files at the root level should be as minimal as it's possible and makes sense.

It's important because it allows you to stay focused on the task you intend to perform, and not be distracted by the need of wading through the files, scripts and configs.

To continue our clothes analogy, it's like with a wardrobe - in the morning it's better to take a nicely folded thing from a shelf. But this convenience costs you the responsibility. To get it, you have to fold clothes neatly after the use, and it's also important to carefully choose what to put into the wardrobe.

While this suggestion sounds vague, and it is, it can help you. Each time when you're about to add something at the root level, by default question the need. Usually, things are set at the root level at the beginning, and then added here rarely. When you justify the presence of something at the top level, think twice about a better place for a file/folder, or if it's needed at all.


### Keep Any Level Clean and Up to Date

Similarly to the root level, any level of the file tree in a project should be clean, with reasonably minimal number of files, free from garbage, and up to date.

This is important because:
- long file listings make navigation harder
- poor organisation complicates maintenance of the mental model of a project in the developer's head
- out-of-date files take up extra mental and physical space
- accidentally committed files are misleading and confusing.

When a new developer joins a project, how much time do they need to get a firm understanding of what's going on in a project? Does the structure help and guide your colleagues through the project? Does it reduce the effort to keep the important details in their heads? Or do they have to stop for a moment each time when stumbled upon a file which no one knows about, whether it's current or not?

A well-organised and maintained project itself helps people in their everyday jobs. Each time when adding a new file, think twice what _else_ can be added in it further. When creating a directory, consider different scenarios a few months in advance. You don't want to end up with a mess one year after, and tidying up a larger and live project is much more of an effort than keeping it clean everyday. If the structure is supported on a daily basis, it will pay off in long-term.

While it's important to have a neat structure, sometimes it's easy to get too serious about a hierarchy. This is what the next suggestion is about.


### Don't Create Artificial Hierarchy

Keep the directory tree in a project as flat as it possible and makes sense. In other words, don't create artificial hierarchy just for the sake of hiding something.

This applies to both, files containing actual code (and we will discuss it in Unit 2), and other files like scripts, configurations, documentation, etc.

A general piece of advice here is that any directory that contains only a single or multiple directories, or a single file, should be questioned for its purpose and existence. Does it belong in here, or somewhere else? Sometimes it might not be clear at the beginning, and this is why it's important to think about evolution of a project some time further.

Consider a situation with scripts. One might have a root-level directory called `scripts`, and the directory contains several files providing tools for CI/CD process, maybe tests, some pre- and post-install steps. That's a good example.

On the opposite, consider the same number of files but located under separate directories like `scripts/build/build.sh`, `scripts/install/install.sh`, and then `scripts/ci.sh` that sources those files. Don't do this.

Here's another example. Say we have an `http` directory. From here, we've got two options. If we know _for sure_ that there will be no other networking stuff in the project any time soon (means a couple of years), then it makes sense to keep it simply as `http`. On the other hand, if there is a non-zero chance of implementing things for `smtp` and `dns`, then it does make sense to put it under `net`. Later, everyone will be happier with the structure because it reflects the nature of things and supports the purpose of the files.

At this point, it becomes clear that grouping for files and folders should be done based on its purpose. Even for non-code files.


### Group Non-Code Files by Their Purpose

Group and keep maintenance and routine files together by their purpose, in one or a few directories.

This helps in keeping the root level free from clatter, as well as reduces the time required to find a file when looking at the listing. You simply know where all your scripts reside, so no time and energy is wasted. When a new developer joins the project, they need minimal time to get an idea of what is where. Everyone is a bit happier.

As it was shown previously, group all scripts in the `scripts` directory. If there is plenty of them, and they're specific to the project itself (like startup scripts, init system scripts, cron files, etc), then it might make sense to keep the CI/CD stuff separately. Stay lean though, having `ci/scripts` and `scripts` is not a good sign. What else do you have for CI but scripts? If that's a couple of yaml files, let them stay under the same `ci` directory (see the previous point).

Nowadays many projects use Docker. If a project uses only one `Dockerfile`, and maybe one file for Docker Compose, then it's fine to keep them at the root. If you have multiple Docker files, and also have some scripts for building containers, then keep them grouped in a directory.

At this point, a valid concern may arise: if all those tools are in directories, it's less convenient to type commands in the terminal. This is what the next recommendation helps you with.


### Provide the Makefile

Provide the Makefile for a project to automate and simplify routine and maintenance tasks.

The `make` tool is available on almost any platform one can imagine, even on Windows without WSL. A minimal knowledge of the syntax of a Makefile is enough to produce a handy set of targets that simplify everyday workflows. The tool have existed for more than 44 years now, and it's likely to continue existing for another 40 years. And in the course of its life, the syntax hasn't changed very much. So it makes a lot of sense investing in learning it once, and benefit from it for the rest of your career.

Just to give you an idea how it can be helpful, consider several, rather basic, examples below.

- building a binary

Without `make` and Makefile:

```bash
GOOS=linux GOARCH=amd64 go build -ldflags "-X main.revision=`git rev-parse --short HEAD`" -o bin/my-app ./cmd/my-app
```

With `make`:

```bash
make my-app
```

- building a container and running an app in it

Without `make`:

```bash
docker-compose build --no-cache my-app
docker-compose up -d my-app
```

With `make`:

```bash
make run-my-app
```

And so forth. While being basic, these examples make a huge difference in everyday experience.

To make your and colleagues' lives easier, define all routine tasks as targets in the Makefile. It can be then used not only by developers directly, but also  in your CI/CD scenarios. This makes the configurations clean, straightforward, and easy to understand and maintain.

It's important to give targets in the Makefile meaningful names. A meaningful name reflects the purpose and meaning of the operation it describes. When this is done well, instead of cryptic bash incantations your config for CI may look something like the one below. Pretty nice, isn't it?

```yaml
# snip

script:
  - make test
  - make cover
  - make build
  - make deploy

# snip
```

We're almost done with common suggestions. Last in the list, but not the least in importance, is a piece of advice about storing credentials.


### Store Credentials Somewhere Else

Under no circumstance should you store any sensitive data in a repository, even if it's a private one. Just don't do that.

Don't add ssh keys, files with passwords, private ssl keys, API or any other credentials. It's alright if a repository contains those for development purpose. Anything else is a taboo. Good sleep at night is worth more than deceptive convenience of having production credentials committed to a repository.

What can be used instead? A private bucket in a block storage that contains an encrypted file with credentials might do a good job. A specialised service from your cloud provider of choice might help too. A special service running in your infrastructure that is responsible for storing and providing other services with sensitive data may be even a better option. But once again, do not store any credentials in a repository along with code.

---

All these suggestions are supposed to help and support a team working with a project throughout its lifecycle. Considered and implemented carefully, they form a strong foundation for the project's success.


## Library

We create libraries for re-using in different places, projects, and by other people. The library's code is read more often than it's modified, simply because it's maintained by a limited number of people, whereas the number of users can be much higher. This is especially true for an open-source library. The library's code is consumed by a computer, but a person is the user and who interacts with it.

The layout of a library project should take these factors into account. This means that the layout needs to be optimised for interaction with the user, a person. We seek for answers in documentation, and read the code provided by a library to understand how it works. If a library is well-organised and neat, the process of using it smooth and unobtrusive.

So how can we make this interaction more positive from the layout perspective? If the [Common guidelines](#common) are taken into account and implemented, that's mostly enough. They cover the most important aspects, and can and should be applied to a library project. Additionally, the following will help you in providing a better experience for the users of a library:
- providing a good README file
- keeping the number of files at the top level of a manageable size
- providing examples of using the library
- providing benchmarking information, if applicable

The rest of the section describes these suggestions in detail.


### Provide a Good README File

Provide a well-organised README file with the most important information about the library.

The README file should give answers to the following questions:
- What is it?
Provide a short description of the library. Ideally, one-two sentences.
- What does it do?
Describe the problem the library is solving.
- Why does it exist?
It should be clear why this library exists, especially if it's not the only library solving a particular problem. What does it do differently from others in its area?
- How does it solve the problem?
Give the users an idea on what's going on under the hood, and what to expect from it.

In addition to the above, the README should contain links to related resources like:
- the GoDoc page
- any resources that describe details of algorithms or approaches taken in the library
- articles or other resources that might be related to the topic.

Your goal with the README file is to help your users in answering the most important question about the library: "Does it help me in solving my problem?". Even if a library does solve a hard problem in a great way, if it isn't reflected in the documentation, that might be unnoticed, thus leading to less people using the library. A well-prepared README file helps your users to answer their questions.


### Keep the Top-Level List of a Manageable Size

Keep the root level list of files short and focused. While this sounds very similar to [this guideline](#keep-the-number-of-root-elements-at-minimum), it's worth a mention here.

Strive to keep the number of root elements such that it takes less than a full page height, so the user doesn't need to scroll to get to the README. It's also important when the user interacts with a library from their IDE. An endless list of files visually complicates navigation, and gives an impression of a poor organisation.

Additional recommendations will be given further in the unit about designing a good layout for a package.


### Provide Examples

In the README, provide clear examples of using your library.

A user should not be forced to go to the GoDoc page, or consult with the code on what using the library looks like. Think about possible use cases, and provide appropriate examples that show it.

Pay special attention to _how_ you're showing the use of the library. Write high-quality code as if it was production code. A half, if not more, of the users will copy and paste code from the examples as it is given. So make sure the examples are not misleading, and do not contain things that you wouldn't want to find in your own production codebase.

Follow these suggestions in order to provide your users with good and quality examples:
- handle errors properly instead of omitting them
- avoid globals as much as possible
- avoid using the `init` function
- use regular imports instead of dot-imports
- handle errors properly instead of instead of `panic`-ing, `log.Fatal`-ing.

This is not an exhaustive list, and more detail will be given in the unit about writing good code. But this short list is enough to make examples better, as well as the code that your users write.


### Provide Benchmarking Results

Provide benchmarks whenever possible.

When someone uses your library, it becomes part of their application. For those who do care about performance, it's important. Those who aren't interested in performance, will skip it. But those who are, will appreciate that. Oftentimes, performance is what differentiates one library from another, and the interested users would love to know about it.

Go provides us with all we need to write benchmarks, and to share the results with our users. So help your users in making the decision about the library, write and run benchmarks, and add the results to the README. Then, keep it up to date.

---

The suggestions given above are simple, yet not always easy to follow. Take the time, make sure the project is prepared and organised. Eventually, the users of the library will find it helpful, and they will thank you for having done this for them.


### Examples

Below you can find some examples of well-prepared libraries, and a few that could have been organised better.

#### Good Examples

- [julienschmidt/httprouter](https://github.com/julienschmidt/httprouter)
- [dimfeld/httptreemux](https://github.com/dimfeld/httptreemux)
- [jmoiron/sqlx](https://github.com/jmoiron/sqlx)
- [rs/xid](https://github.com/rs/xid)
- [cryptowatch/cw-sdk-go](https://github.com/cryptowatch/cw-sdk-go)

#### Can Be Improved

- [lib/pq](https://github.com/lib/pq)
While the README lists some features, it doesn't answer many questions, nor it gives us examples of using it, benchmarks, or any other information about why should we use it. The file organisation could also be improved.
- [go-redis/redis](https://github.com/go-redis/redis/tree/v7)
As the number of Redis clients is relatively high, it would be valuable to provide users with comparisons and benchmarks that show the difference from other packages. Also, the structure of the repository can be better as not everything need to live at the root level of the package.


## Single Application

A project (and its repository) for a single application is a further development of the guidelines given so far. Almost, if not everything, from the [Common](#common) and [Library](#library) sections can, and most likely, should be applied when working on a single service from the layout perspective. There are a couple of recommendations that make the experience better.

One distinction between a service and a library is that there is always at least one artefact, a binary. It also means that a project has at least one `main.go` file, which is not needed for a library. The experience of working on a project depends, to some extent, on where those files go.

The following will make interaction with a single-application project more convenient and easier:
- use a separate directory for entry points to the service and satellite tools/applications
- use a special, excluded from Git, directory for outputting build artefacts.

Next couple of sections go in expand the suggestions.


### Use `cmd` Directory for Entry Points

Use `cmd` directory for entry points to your service. Inside the directory, create a directory per each entry point, named after the application. The inner directories should mainly contain only one file, `main.go`.

Even when a project is a single application, oftentimes it has a number of satellite CLI and development tools, or different modes. Instead of a long list of `if` statements, this approach allows for small and focused, yet different entry points.

Create a directory named `cmd` at the top level of the file tree, and move any entry points, i.e. `main.go` files in to appropriate directories inside `cmd`. These second-level directories should be named as you want the binaries to be. Each of those directories is an isolated `main` package, hence a separate entry point to the app, defined in it.


### Use `bin` Directory for Binaries

Use `bin` directory for build artefacts. The directory must be excluded from the source control system. The directory is used in both development and CI processes.

As you remember, there should be no garbage in a repository. A binary that is built during development process should not appear among committed content.

Usually, a binary is excluded by listing its name in the `.gitignore` file. While this gives an immediate result and may work, it doesn't mean there is no a better option. This approach is not scalable. If someone builds the service with a custom name for the binary, the output can be accidentally committed. The same with CI, the process will just put files right at the root level.

A way to improve the situation is to have a separate directory in the project which is:
- excluded from Git
- managed (created and cleaned) automatically
- and used by both developers and build tools.

The special directory is named `bin`, and its place is at the root level. The entire directory is excluded from Git.

Then, since the [Common](#common) recommendations are in effect, you've got the Makefile. Given the previous tip, the targets work like this:
- inputs are taken from the `cmd` directory, e.g. `cmd/my-app`
- binaries are put to the `bin` directory, e.g. `bin/my-app`

Combined together, we get:

```bash
go build -ldflags "-X main.revision=`git rev-parse --short HEAD`" -o bin/my-app ./cmd/my-app
```

If the directory doest not exist, make sure it's created as needed. When you do `make clean`, the target should remove everything from that directory.

All targets in the build or testing pipelines also use this path for outputting and accessing binaries. When running the service locally, you run it from the directory. Nice and tidy: `bin/my-app`. If typing in extra characters is a concern, define a target the Makefile, and use `make run-my-app`. Having this as a target is a scalable way since everyone in the team can use it. If need, you can also create a shell alias as `alias run-my-app='make run-my-app`.

The targets used in CI process also use the `bin` directory for artefacts, and then packaging and deploy steps seek for the artefacts in there. The process is structured and organised.

Having a separate directory for binaries is important because chances of multiple binaries are very high. One popular scenario is when a developer works under macOS, and uses Docker. While local version is going to be built for the Mac, the Docker version is built for Linux, hence there are two different binaries already.

It's likely that sometimes there will be a need to build binaries for many systems locally. Thus, two entries should be present in the `.gitignore` file. Then, if a new binary is introduced later, it will be another two entries. That's already a lot to keep track of, and too many files are at the root level.

Instead, define the `bin`  directory once, add it to the `.gitignore` file, set up all appropriate targets in the Makefile to use the directory, and don't worry about it for the rest of the project's life.

---

Here is how a single-application repository may look like with the two recent suggestions applied:

```bash
├── bin
│   ├── example-agent
│   ├── example-cli
│   ├── example-devsrv
│   └── example-plugins
└── cmd
    ├── example-agent
    │   └── main.go
    ├── example-cli
    │   └── main.go
    ├── example-devsrv
    │   └── main.go
    └── example-plugins
        └── main.go
# snip
```

As you see, the structure is organised in a good order, and easy to work with.

Even more important the `bin` and `cmd` directories are when a project contains code for multiple _different_ services. In this case, it's a monolithic repository, and many people, potentially several teams, are working on it. All previously given recommendations work especially well with a monorepo. That's what we will talk about in the next section.

---

To be continued...
