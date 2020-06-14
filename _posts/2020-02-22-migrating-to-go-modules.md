---
layout: post
title: Migrating to Go Modules
category: Programming
tags: [golang, modules]
---


This article aims to describe the process of migration to Go Modules in different cases. While it's virtually not possible to cover all scenarios, we'll try to consider the ones that are likely to occur in an average environment.

The article covers the initial migration of services and libraries. It does not tell you how to implement versioning, or how to release versions above `v0`. For that, there is plethora of articles on the Internet. Here we're focusing on how to get from `dep` to `go get`.

<!--more-->

_The article uses a personalised communication style to ease the conversation and focus on the topic._

![](/assets/library.jpg)

## Why

Go Modules arrived a year and a half ago with `go1.11`. It's not possible to describe the reasons behind the decision better than Russ Cox did it in his series of posts. If you haven't read them yet, it is probably worth reading. It gives you the full picture of why and how the team has arrived to that decision:
- [The Proposal](https://blog.golang.org/versioning-proposal)
- [The original blog post series](https://research.swtch.com/vgo)

A short summary is:
- versioned imports - support for multiple versions of a dependency;
- minimal version selection - a stricter version selection algorithm that provides a safer way for updating dependencies
- checksums and stricter verification for dependencies
- improved approach to vendoring (outside of a repository)
- overall, simplified dependency management
- the official and standard way to work with libraries and dependencies.

With the upcoming release of `go1.14`, and after having been being in use for 1.5 years, modules are officially considered [ready for production use](https://github.com/golang/go/wiki/Modules#go-111-modules).

These days, more and more projects and libraries migrate to modules. While the Go team has been working hard on keeping compatibility between modules and other dependency management tools (mostly `dep`), there are cases when it's impossible for a project, which is not using modules, to use a library that does use them.

On the other hand, `dep` hasn't been updated since [Jul 1st 2019](https://github.com/golang/dep/releases/tag/v0.5.4), which is a clear sign that the tool is not going to be actively maintained further.

To summarise, the reasons for migrating to modules are:
- they are _the way_ to manange dependencies in Golang
- they simplify the dependency management workflow (mostly)
- to use some libraries that do use modules (in some cases it's a must, read further)
- `dep` is most likely to stay without meaningful maintenance and eventually will be sunsetted.

So further in the article we assume that the decision on migration has been made. Now you need to understand how urgent is it.


## When

Go Modules guarantee backwards compatibility, in most cases. A non-modules project/library can use a module-based one, and vice-versa.

There is one caveat which comes from one of the most exciting features of Go Modules. Here it is - if a dependency you're trying to add **uses Go Modules** and **is versioned**, and your project or library **is a non-modules one**, it won't work. ~~Deal with it.~~ This is true regardless of whether it's a direct or indirect dependency (unless it's vendored in an intermediary library).

For example, if you try to use this package `github.com/vmihailenco/msgpack/v4/codes`, or one of your dependencies uses it, `dep` vomits something close to:

```bash
Solving failure: No versions of github.com/vmihailenco/msgpack met constraints:
	v4.3.7: Could not introduce github.com/vmihailenco/msgpack@v4.3.7, as its subpackage github.com/vmihailenco/msgpack/v4/codes is missing. (Package is required by (root).)
	v4.3.6: Could not introduce github.com/vmihailenco/msgpack@v4.3.6, as its subpackage github.com/vmihailenco/msgpack/v4/codes is missing. (Package is required by (root).)

...skip...

	master: Could not introduce github.com/vmihailenco/msgpack@master, as its subpackage github.com/vmihailenco/msgpack/v4/codes is missing. (Package is required by (root).)
	v2: Could not introduce github.com/vmihailenco/msgpack@v2, as its subpackage github.com/vmihailenco/msgpack/v4/codes is missing. (Package is required by (root).)
```

In this case, there are two options available - reconsider the need of adding such a dependency (which you should always consider, regardless of the topic of the article), or update your project/library to support the modules.

To conclude:
- There is no urgency with migration _yet_. The Go Team starts encouraging migration for the majority of users with the release of `go1.14` (this month).
- You must migrate to modules if one of your direct or indirect dependencies uses both modules and a versioned import path (with `V2` and above).

So take your time, plan and execute depending on the load, but do not delay it for too long.

If you feel like it's too hard to do the migration, just think for a while about the fact that Kubernetes, one of the largest Golang projects to date, finished their migration to modules in April 2019.

Now as we agreed that it's worth migrating, and have known and planned when to do that, let's learn how to do that. The rest of the article focuses on the technical side of the topic.


## How

Alright, it's time to do some real work.

Just before we get started, here is what we need to bear in mind before, during and after the migration:
- You may probably want to reconsider approach to vendoring.

    While Go Modules is extremely convenient, the classic approach to vendoring has become a bit complicated. Why? Because Go Modules provides another (and I think, a better) approach to dependencies. It drastically reduces the need in vendored dependencies. And even if you do still need them, it has something to offer. If that doesn't suit your needs, then well, you'll struggle.
- You may probably want or need to update the IDE/code editor.

    If Goland is the IDE of choice, carry on and don't worry. If you're already on [gopls](https://github.com/golang/tools/tree/master/gopls) or [bingo](https://github.com/saibing/bingo) plus some code editor like VS Code/Vim/whatever, don't worry and move on. Otherwise, the tooling needs to be upgraded. The set of tools many of us got used to (like `gocode` and the guys) **do not support** modules, and thus - no autocompletion, go to definition and all of these convenient tools;
- There is some chore work involved with amending any build scripts you use.
- If you're still using Golang below `1.13`, upgrading to it is a must.\

    Make sure `go version` gives a version of `1.13.x`. Upgrading to the latest version is outside of the scope of this article.

Some of the questions above will be covered later. For now, let's get started, and go from the easiest case to a more advanced one.


### Prerequisites

All of the steps below assume the prerequisites are met:
1. Make sure to be in the directory of the project.
2. `go version` is something like `1.13+`.

During the migration process it's important to keep in mind that breaking things  while updating to a new version of a tool is, at least, undesired. So it's your responsibility to make sure that nothing is broken or degraded due to the update.

All following steps will assume that you:
- are working in a separate branch
- will remember to update your build scripts
- performing a thorough testing at each step of the migration.


### Updating a Non-Vendored Library or Project

The easiest and the most straightforward case out of all.

To start using modules, simply do the following:
- initialise the module with `go mod init github.com/account_name/repo_name`
- make sure that your imports and dependency definitions match with `go mod tidy`
- check if it works `go test -count=1 ./... -v`
- when confirmed working, remove the `dep` files - `rm -f Gopkg.toml Gopkg.lock`


### Updating a Vendored Library or Project AND Dropping Vendor

Whether to vendor or not is discussed further in the article. This section assumes that the decision has been made to drop the support for vendoring.

Technically, steps are the same as in the previous case, with an extra step to remove the `vendor` directory:
- initialise the module with `go mod init github.com/account_name/repo_name`
- make sure that your imports and dependency definitions match with `go mod tidy`
- check if it works `go test -count=1 ./... -v`
- when confirmed working, remove the `dep` files - `rm -f Gopkg.toml Gopkg.lock`
- remove the dependencies `rm -fr ./vendor`

It's your responsibility to make sure that all the dependencies are as close to the vendored by versions as possible. If it's not possible to achieve reliability and you're unsure or concerned with the change, err on the side of keeping `vendor` for a while until the problem is solved. If it's your case, read on the following section.


### Updating a Vendored Library or Project AND Continue Vendoring

While Go Modules are awesome and convenient, if vendored dependencies are left behind, they are somewhat less convenient when it comes to vendoring. But until you have decided to let them go, let's update the project to use modules, and vendoring.

Initial steps are similar, the difference comes into play when we attempt to build the project.

Firstly, initialise the module with `go mod init github.com/account_name/repo_name`.

Secondly, you need to explicitly vendor your dependencies by `go mod vendor`. This will copy the dependencies from the module cache to the `vendor` directory.

Lastly, the building process is changed. To build, run `go build -mod=vendor`. This is important, otherwise `go` ignores `vendor`.

Once you've achieved some level of certainty that the build is reliable and stable, finish ~~him~~ the process with `rm -f Gopkg.toml Gopkg.lock`.


### Updating Build Scripts

For libraries/projects that don't use vendoring, besides the prerequisites, there should be nothing to change in build scripts.

For projects that DO use vendoring, there might be different issues, depending on how the build process is configured, and what steps it consists of.

Most common problems are:
- `go get /some/pkg` fails or a binary cannot be found:
    - firstly, read [the](https://blog.golang.org/using-go-modules) [docs](https://blog.golang.org/migrating-to-go-modules)
    - and then use the `go install /some/pkg` instead to use the binary
- builds are failing because of missing packages:
    - set `GOFLAGS="-mod=vendor"`
    - or add the `-mod=vendor` argument to the `go build` command wherever it's used in the scripts
    - DO NOT add `export GOFLAGS="-mod=vendor"` to your local bash profile, unless ALL of the projects you work with use vendoring.

Once everything works as expected, and has been tested well, we are done with migrating to modules. It's time to see how to add a dependency.


### Adding Dependencies

As with building, for non-vendored libraries and projects, adding a package is easy:
- if you're IDE/code editor is properly set up, it's enough to just add it to imports
- otherwise, add a package to imports first
- and `go get /path/to/package`
- check with `go build` if it works.


With vendored services there's a little more work involved:
- import the dependency
- `go get /the/dependency`
- **and then vendor it explicitly by** `go mod vendor`
- and then build with `go build -mod=vendor` to see if it works

Without explicitly vendoring the dependency, it won't be added to the `vendor` directory, causing errors when building and testing.


### Thoughts on Vendoring

Personally I think vendoring should not be in use with the introduction of modules. Mostly because modules solve the problem of keeping builds reproducible.

Some developers are concerned with potential lost of a dependency if it has been abandoned or deprecated. Well, in that case there are a few solutions to the problem available:
- either do not use the library which looks like to be sunsetted soon
- or consider forking it
- or consider alternatives.

Also, general common sense and cautious when introducing a new dependency are good to apply. I usually evaluate a library before introducing it to a codebase. Pay special attention to:
- the quality of code
- the behaviour of the library (be suspicious of selfish libraries which silently `panic` or, even worse, do `os.Exit(1)` or `log.Fatal` under the hood)
- possible issues
- recent activity
- the core developer.

If you do see the need in a sort of vendoring, then it is worth considering using a Go Module Proxy. You may need to vendor dependencies through a proxy in a company. This is worth a separate discussion, but a rough and high-level idea/approach may sound like this:
- have a proxy close to your CI environment, so the pipeline saves some time on downloading dependencies
- have a proxy close to your office, so developers don't waste time on waiting for download to  complete.

---

That's it. If you've made this far, thank you for your attention, and my hopes that this has been useful for you.

**Happy coding!**
