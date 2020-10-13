---
layout: post
title: "The Style Guide: Project Layout 4 - Versioning and Go"
category: Golocron
tags: [golang, m1u1]
---

> Order is never observed; it is disorder which attracts attention because it is awkward and intrusive.
>
> – Eliphas Levi

This post covers another important topic in the context of the project layout, the process of creating and assigning new versions in a project. We first learn the theory, and then look at more practical aspects of it. Continue reading, if curious.

<!--more-->

![](/assets/m1u1_4_2.jpg)


## Before We Begin

In this draft, we talk about the task of versioning in a Go project. This is important on its own, but also due to the changes the Go Team introduced with Modules. This question has two sides:
- using Modules for dependency management
- and using it for managing versions of the software you develop and maintain.

The former is partially addressed in [one of the previous articles](https://blog.pavelbrm.com/programming/2020/02/22/migrating-to-go-modules/). The latter is the subject of this material.

As usual, my hopes are that you find something new and interesting here.

- [Versioning and Go](#versioning-and-go)
- [Notes on Terminology](#notes-on-terminology)
    - [The Main Branch](#the-main-branch)
    - [The Default Branch](#the-default-branch)
- [Prerequisites and Assumptions](#prerequisites-and-assumptions)
- [Approaches to Versioning](#approaches-to-versioning)
    - [Review](#review)
    - [Directory-Based Versioning](#directory-based-versioning)
    - [Branch-Based Versioning](#branch-based-versioning)
    - [Branch-Based With Rolling Main Branch](#branch-based-with-rolling-main-branch)
    - [Summary](#summary)
- [Implementing Versioning](#implementing-versioning)
    - [Overview](#overview)
    - [Initial Migration to Versioning](#initial-migration-to-versioning)
    - [Changes Within One Major Version](#changes-within-one-major-version)
    - [Introducing a New Major Version](#introducing-a-new-major-version)
- [Notes on Automation](#notes-on-automation)
- [Conclusion](#conclusion)


## Versioning and Go

The unit won't be complete without considering such an important topic as dependency management, versioning and distribution in the context of the project layout. Thankfully, in Go this is addressed mostly with one of the most significant changes over last several years which is Go Modules.

Although Modules was released more than 2 years ago, many individuals and companies have only started adopting them. Partially this is due to laziness that is often masked under complains about lack of documentation. There is, actually, lots of high quality content on the topic, as Russ Cox published [an extensive and exhaustive set of articles on his blog](https://research.swtch.com/vgo) while Modules were going from the proposal to the final release. Partially this is due to a generally slow adoption process when something new is introduced.

Anyway, over the course of last year, the situation with documentation has significantly improved. One of the best resources so far is [the Go Modules section](https://github.com/golang/go/wiki/Modules) on the official Go Wiki. Yet, developers are often not confident when it comes to either performing a major dependency update, or releasing a major version. Many projects are still using `dep`. And while there is nothing wrong with `dep` or the use of it, the tool is now obsolete, and it will be restricting its user in many ways. So it's better to migrate to Modules, and benefit from what Go has to offer.

This section summarises the learnings and experience gathered since when Modules were first mentioned. If you've already read all of the official materials mentioned above (Russ's posts and all posts in the official Go Wiki and blog), then you probably learn a little from the section. However, there is some practical advice that might be useful. And given the current module is called "The Style Guide", consider everything below as a recommendation.

## Notes on Terminology

This section clarifies the terminology used further around branches in Git and GitHub. This is important because with the introduction of Go Modules, depending on the approach that is taken to versioning, the meaning of branches may change.

When working with Git, and especially with GitHub and the likes, there are two special kinds of branches:
- the main branch that is considered as the most recent/stable branch
- the default branch that is:
    - the default branch in Git
    - the default base branch for new pull requests
    - the default branch shown when the repository is visited on the web
    - in Git, versions prior to `v2.28.0` defaulted to `master`. Starting from `v2.28.0` it allows specifying an alternative name
    - on GitHub, before October 1, 2020 it was `master`. Starting from the day it's `main`.

Knowing this two terms is important, as well as understanding the role and meaning for the workflow.


### The Main Branch

Historically, and until recently, the main branch has been referred to as `master`. From this point and further in the book, this branch is referred to as "the main branch", or simply `main`.

The main branch in a project usually contains the most recent version of code. It’s also widely accepted in the open source community that the main branch is the bleeding edge, containing the latest features, thought might not necessarily stable.

The meaning of the main branch in organisations is slightly different. Often, instead of being the cutting edge, the main branch represents the most recent version that is shippable to the customer and/or deployable to production. Of course, if an organisation follows The Git Flow, then the meaning of the main branch is different. We refer to the main branch in a sense that is closest to The GitHub Flow.

It's fair to say that, for the majority of companies, the main branch is the most recent stable branch that is currently in production.

So from now on, when you see "the main branch" or `main`, and in your case it's still named `master`, don't be confused.


### The Default Branch

The default branch in a repository is the one that is:
- chosen and its contents shown when you visit the project’s homepage
- default base branch for a new pull request.

There are a few notes about the default branch. The first is that with GitHub, we've become accustomed to seeing the most recent version of a project when we visit the repository. Very often, the default and main branches in a project are the same branch.

On the other hand, sometimes the default branch is not necessarily the same as the main branch. This becomes especially important the introduction of Go Modules.

Your users, including you, probably like the experience that is familiar. Getting immediate access to the most recent version without needing to switch the branch when you visit a project page on GitHub saves some time and energy, and prevents from unnecessary distractions. So it would be great if it's possible to keep things simple.

As we shall see, Go Modules introduces ways that can disrupt the existing workflows under certain conditions. We will also learn how to keep things in a good order, such that versions are properly and conveniently managed, and users are happy with quick and easy access and to the most recent code.


## Prerequisites and Assumptions

As we're going to look into some practical aspects of versioning after learning it in theory, it's good to introduce agreements about the environment in which all that works.

For the first, the most important thing here is the version of the programming language. It's worth a reminder that in Go only two most recent versions are officially supported. At the time of writing, the latest stable version is `v1.15`, and the other is `v1.14`. Anything else is unsupported, and should not be in use. If you do, then it's strongly advised to perform an update to the most recent version, for many reasons.

So the first agreement, and a prerequisite, is the version of the language. In everything that is related to versioning and Modules, **the book assumes that you use at least version `v1.14`.**

The second important thing is what is used for versioning and dependency management. Although there are alternative opinions and tools, Go Modules is the standard, de facto and de jure. Moreover, it has already proved to be a convenient and handy tool for both, managing dependencies and managing the release cycle of a software project.

So the second agreement and a prerequisite is about versioning, dependencies and tooling. From the first letter to the last, **the book assumes that you use Go Modules for dependency management and versioning in Go projects.**

The third agreement is implied by the previous, and is about versioning strategy. In modern times, people increment version numbers in any way they like. This kind of freedom is good to some extent. However, in this work, and this is aligned with the official approach Go takes to versioning, **the [Semantic Versioning](https://semver.org/) model is used**. In fact, the way Go manages dependencies with the minimal version selection algorithm is built upon conventions established by Semantic Versioning. Similarly, when and what kind of a change triggers increase in what part of the version, is governed by Semver.

As a quick reminder, Semantic Versioning means that for a version number `MAJOR.MINOR.PATCH` each part is incremented as follows:
- `MAJOR` when a breaking change is introduced
- `MINOR` when a new feature/update is backwards compatible
- `PATCH` when backwards compatible bug fixes are made
- also, additional labels for pre-release and build metadata can be added and incremented to the `MAJOR.MINOR.PATCH` format.

Semver has proven to be a meaningful and convenient approach to managing software version. It works perfectly well with a standard, slowly increasing versions model. It also works well for one approach that has become quite popular in recent years. You, of course, have seen versions like `v2020.08.12`, which can be generalised as `YEAR.MONTH.DAY` or `YEAR.RELEASE_IN_THE_YEAR.FIX`. It's less flexible as compared to the classic model, but still fits nicely into the semantic model.

Let's quickly recap the prerequisites:
- Go version is at least `v1.14`
- Go Modules for dependency management and versioning
- Semantic Versioning is used to decide when and what version is incremented.


## Approaches to Versioning

With modules, there are three main strategies for versioning a project. The official [Go Wiki provides](https://github.com/golang/go/wiki/Modules) some guidelines on how and when to use which. In this section we review what the options are, how each works, and when to use it.

Building a firm understanding of how all this works and how it is represented, is required for working with modules efficiently. For this purpose, there is a quick review of important technical aspects of Modules.

After the review, we consider three approaches to versioning a Go project, namely:
- Directory-Based Versioning
- Branch-Based With Changing Default Branch
- Branch-Based With Rolling Main Branch.


### Review

Getting familiar with the terminology of Modules is helpful in becoming confident when working with it. The bare minimum includes understanding that:
- anything that has not been tagged yet, is implicitly considered as `v0.0.0-YYYYMMDDHHmmSS-first12SymbolsOfCommitHash`
    - e.g. `v0.0.0-20190827140505-75bee3e2ccb6`
- anything that has been tagged with `v2`+ but **is not a module** (i.e. just a tagged release), is implicitly considered as `vX.Y.Z+incompatible`
    - e.g. `v2.2.1+incompatible`
- a correct major version must be set in the `go.mod` file for versioning to work for versions starting from `v2`
    - e.g. `module github.com/awslabs/goformation/v4`
- each major version requires its own `go.mod` file (follows directly from the previous point)
- an appropriate and valid tag is a must for versioning to work properly
- the previous major version must exist in order for the next to work properly on the client
    - i.e. a client fails to use `v2.0.0` if there is no a single release tagged with `v1.y.z`
- backwards compatibility and introduction of breaking changes is governed by **the conventions set by Semantic Versioning**. I.e.:
    - only fixes to stability/correctness/security are allowed in a patch-level release `vx.y.Z`
    - new features that don’t break backwards compatibility are allowed in a minor-level release `vx.Y.x`
    - breaking changes are **only** allowed in a major-version release `vX.y.z`
- unless a version is given explicitly, with `go get`/`go get -u` **updates are performed only within the same major version**, i.e. up to the latest available `MINOR` version.
- `go get github.com/someorg/somepkg` updates the package to its latest version
- `go get -u github.com/someorg/somepkg` updates the package **and all its dependencies** to their latest versions, and this is where the Minimal Version Selection algorithm works.

Keeping these in mind and following the rules is crucial for versioning to:
- work
- work as expected
- work correctly.

From now on, the assumption is made that the reader has familiarised themselves with the basics and theory of Modules.


### Directory-Based Versioning

The directory-based approach is the most conservative option that is available. New major versions live in the corresponding directories, in the same branch.

#### What It Is

With this technique, all versions reside in the same branch, but under different, major version-specific top-level directories. When a new major version, `vN+1` is about to be created, all code from the current version, `vN` is copied into a new directory, `vN+1`. Each version is tagged.


#### How It Works

The first stable version is kept at the root level of a project. All new major versions are housed in directories that are created in the root directory, named after the version they represent - `v2`, `v3`, and so forth.

Essentially, this technique works as follows:
- everything resides in the main branch
- while the project is in active development phase, releases are tagged with `v0.y.z`
- when the project reaches maturity, it’s tagged with `v1.y.z`
- when there is a breaking change, a new directory `v2` is created AND:
    - all the code is COPIED over to the new directory
    - also, the `go.mod` file
    - the `module` statement is changed in the `go.mod` file to reflect the new version, i.e:
        - `module github.com/myorg/mylib/v2`
    - when needed, a new version is tagged with `v2.y.z`.


#### When To Use It

The main benefit of this option is backwards compatibility with pre-Modules versions of Go. In other words, it's recommended for the projects that have backwards compatibility with old versions of Go as one of the main goals.

The Go Team has backported a basic support for Modules to `1.9.7+` and `1.10.3+`, and improved support in `1.11`, so these versions are able to consume modules with versions `v2+`.


#### Weak Points

Although the method is the most conservative and allows for the widest range of supported versions of Go, it has some obvious and hidden downsides.

Firstly, it opens up a possibility for misusing Modules and breaking the client of a library. The major downside of the method is that it’s too tempting to begin sharing code between the versions. If there is a need to share code between versions, pull it off to a separate **module** (i.e. library).

A second downside is pollution of the root directory if the versioning policy is aggressive, and new major versions are released quite often. For example, [go-redis/redis](https://github.com/go-redis/redis) has already got `v8`. If they followed this path (thankfully they don't), they would have ended up with something like below plus all the files they have:
```bash
v2
v3
v4
v5
v6
v7
v8
```

The only real strong reason for using the major directory approach is backwards compatibility with old, officially unsupported versions of Go. Unless it's a main goal of your project, using this method is not recommended.


### Branch-Based Versioning

A more convenient and the intended approach to versioning is based on branches. This approach fully utilises the Go modules functionality that is baked into versions `v1.12`+ (although `v1.11` was the first to support modules, it’s was more like a preview).


#### What It Is

Each major version is kept in its own branch. When working on patches and minor updates for a version, a working branch is based off of the corresponding home branch for that major version. When a breaking change is introduced, a new major branch is created, and is tagged accordingly.


#### How It Works

Before going into detail on how this method works, a couple of things are worth noting:
- strictly speaking, a new branch is not required, as long as a change can be introduced without breaking the existing code
- by default, the Go Team assumes that you use the main branch to house `v1.y.z`.

The way this technique works can be summarised as:
- everything resides in the main branch
- while the project is in active development phase, releases are tagged with `v0.y.z`
- when the project reaches maturity, it’s tagged with `v1.y.z`
- when a breaking change is about to be introduced, a major new branch `v2` is created AND:
    - the `module` statement is changed in the `go.mod` file to reflect the new version
        - i.e. `module github.com/myorg/mylib/v2`
    - when needed, a new version is tagged with `v2.y.z`
    - all changes to `v2` are targeted towards the `v2` branch. I.e.:
        - the base branch for any PR that touches `v2` code is `v2`
        - effectively, the `v2` branch is now a second main branch in the project, for anything related to this version
- when there is a need to introduce a fix or backport a change to `v1`, it's done as it was done before:
    - the base branch for any PR that touches `v1` code is the old main branch (`main`, `master` or something else).


#### When To Use It

In general, this way is what the Go Team has planned as the intended use of Modules, and what it is designed to work together the best with. It has mostly advantages, including:
- nice separation of concerns
- clear boundaries between incompatible versions
- ease of use
- ease of automation.

However, these guidelines advise against using this method, due to its downsides, which are not as obvious at the first glance, and in favour of the third option.

If, after reading about the rolling main branch technique, that approach does not seem appropriate, then default to this, simple major-branch based way.


#### Weak Points

From a purely technical perspective, this approach has only one, rather minor, downside which is just a new way of working. At the same time, there is a bit bigger question about the changed semantics and perception of the two branch terms introduced in the [Notes on Terminology](#notes-on-terminology).

In this context, the following two questions arise:
- what branch do I use as the main branch?
    - i.e. what is my current, most recent shippable/deployable version, or the branch containing experimental changes?
- what branch do I use as the default branch? I.e.:
    - what is the branch to show by default when my repository is visited?
    - what is the branch new pull requests should use as the base branch?

With this technique, the answer to the first question is unclear. By the classic meaning, the bleeding edge should be pointed to the branch that has the highest `vX`, or to the branch that is even untagged. By the meaning employed by organisations, it should be the branch that is currently deployed in production. Again, it should be the most recent `vX`. But the community seem to still consider the main branch that is the home for `v1` as a main branch.

A similar question is about the default branch. It’s reasonable to say that when you visit the homepage of a project, it’s natural to expect the documentation and code shown to be up to date. Therefore, the default branch for a repository should be changed each time when the major version is incremented, which means a new major branch is created. If the latest version in a project is `v3`, then the default branch in the repository, for convenience of the users and developers, should be set to this branch.

Besides being technically beneficial, this approach has a philosophical downside - the meaning of the main branch of a repository is changed, and now there is no permanent home for the latest and greatest changes. For other languages, this still holds.

A better option, which has the same technical advantages, but also does not have the methodological downsides of the this one, is a slight modification of major branches and a rolling main branch.


### Branch-Based With Rolling Main Branch

It would be great to have the questions, left by the previous option above open, answered. It would be also great to keep the meaning of the main branch. As it turns out, there is a way to address all of these.

This approach is an extension to the previously described method, with one, rather significant difference.


#### What It Is

In essence, the main branch always represents the most recent version, whatever it is. When a new major version, `vN`, is introduced, the previous major version, `vN-1`, is kept in the appropriate major branch, `vN-1`, for `N >= 2`. All new development continues going in to the main branch. Hence, everything remains as it should be, in a good order.


#### How It Works

A project starts with the main branch and no versions. At some point, the first version is tagged in the main branch, and it remains here. When it’s time to work on a new major version, prior to making any breaking changes, a stable version-specific branch is created, `v1`. This branch is then used for all subsequent minor and patch releases, they are based off and merged into this branch, when making changes to `v1` code. As of this point, the main branch is the home for the new version, and is tagged as minor and patch releases are prepared. When it’s time to release another major version, the process is repeated, keeping a snapshot of the current stable version in v2`.

This approach ensures the following:
- the main branch remains the most recent and stable version (as you merge only code that is proved working)
- the main branch continues to be the source of truth
- the default branch and the main branch are the same.

This is how it works:
- everything resides in the main branch
- while the project is in active development phase, releases are tagged with `v0.y.z`
- when the project reaches maturity, it’s tagged with `v1.y.z`
- right before working on a breaking change, create a snapshot of the stable code in `v1`
    - and continue using it for updates and fixes to this version
- a new, potentially, incompatible, next version is now located in the main branch, AND
    - the `module` statement is changed in the `go.mod` file to reflect the new version, i.e `module github.com/myorg/mylib/v2`
    - when needed, a new version is tagged with `v2.y.z`
- when it's time for the new version, right before working on a breaking change, create a snapshot of the stable code in `v2`
    - and continue using it for updates and fixes to this version
- a new and, potentially, incompatible, next version is now housed in the main branch, AND
    - the `module` statement is changed in the `go.mod` file to reflect the new version, i.e `module github.com/myorg/mylib/v3`
    - when needed, a new version is tagged with `v3.y.z`
- repeat.

This model allows an easy to understand, simple to work with and clean solution. The questions about the meaning and roles of the main and default branches are answered.


#### When To Use It

The guideline is to prefer this method whenever possible and **by default**. Only use the alternatives when (not if) there is no way this approach can work.


#### Weak Points

As this technique is an extension of the major-branch approach, and it has addressed the weak points left since, it's hard to find a reasonable downside.

Of course, someone could argue that:
- it does not support the pre-Modules versions of Go
- there is some added process into when to do what.

However, the reality is such that:
- when you need support for old versions, use [the major version directory](#directory-based-versioning) technique
- without a process, there is no order.


### Summary

As we saw, for a new project, it's important to make a right decision at the beginning, as it significantly simplifies things going forward. Nonetheless, even changing the approach for an existing project from one to another is surely doable, it just requires a little more effort. What is important, however, that the method used for versioning, is convenient, and is not limiting when looking into the future. Pro-actively preventing from accumulating legacy, or actively working on reducing the amount if it exists, is always worth the effort.

The following table combines together the facts that are likely to be involved when deciding on the approach to versioning.

Name | How | Next | Default | Main | Go | When
--- | --- | --- | --- | --- | --- | ---
Major Directory | dir<br>`vN` | dir<br>`vN+1` | `main` | `main` | `v1.9.7+`<br> `v1.10.3+`<br>`v1.11+` | Support for old versions
Major Branch<br>`v1` in `main` | branch<br>`vN` | branch<br>`vN+1` | latest `vN` | latest `vN` | `v1.11+` | When the major branch with rolling `main` does not suit
Major Branch<br>rolling `main` | branch<br>`vN-1` | `main` | `main` | `main` | `v1.12+` | By default, the preferred choice

So choose wisely, experiment, implement and move on. In general, stick to the third option. :-)


## Implementing Versioning

Having reviewed the theory in the preceding sections, let's now turn to the practical side of the subject. Specifically, there are three basic questions about versioning:
- What does the initial migration to versioning look like for a stable project?
- What does the process look like within the same major version?
- What does the process look like when a new major version is introduced, assuming versioning has already been implemented?

This list is, of course, not exhaustive, and barely scratches the surface of all possible cases. For more information, see [the Modules page](https://github.com/golang/go/wiki/Modules) on the official Go Wiki. This section aims to provide the reader with guidelines on how to achieve versioning. If a situation you're in is more of an edge case, and is not described here, then it's likely to be covered by the Go Team.


### Overview

One last pre-flight check before moving on to the practical advice. The procedures and steps described further rely on certain assumptions, and will not work if some preconditions aren't met. Here we briefly remind what those conditions are.

The following is considered as a given, and further is implied to be true:
- Semantic Versioning is used, and strictly followed
- Go Modules is the default
    - any library/project that has not been migrated to Modules yet, should be migrated before proceeding
- No backwards compatibility with pre-Modules versions of Go, including:
    - 1.9.7+
    - 1.10.3+
    - sometimes, 1.11
- No backwards compatibility with projects/libraries those dependencies are managed with `dep` or anything else but Modules.

With this in mind, let’s move on to concrete steps towards versioning.


### Initial Migration to Versioning

Initial migration to versioning may be required in several cases. For example, a library might have been already stable for a while, but has not been versioned yet. This is quite often the case for libraries developed by companies and used internally. The same may as well apply to an open-source package.

You may also want to perform the initial transition to the Go way of versioning even if your project already uses versions in its simplest form, i.e. just tags.

The steps below assume the first case. However, the procedure will also work for the second scenario, with the only difference being the starting point. The instructions assume that the current state of a library is stable, and the intent is to migrate and version it properly, *without breaking potential clients that might the project*. So it's cautious and has steps that are aimed at providing the strongest guarantee.

This is how the initial migration may look like for a previously non-versioned project to versioning:
- pull the most recent version of the main branch
- create a tag `v0.1.0` in the main branch
- push the tag
    - at this point, all clients that have a pseudo-version `v0.0.0-etc` will be able to get this version (if don't use vendoring and are running `go get`, `go build` or any command that implies downloading modules)
    - *if migrating a project that already uses tags, then increment the `MINOR` version*
- optionally, update clients explicitly, if needed, but this is superficial, due to the point above
    - at this point, we have to fix the first stable version, as there is no such a version as `v0`, hence there is no way yet to introduce a breaking change, as we need to landmark the stable version
- create a new branch off of the main branch, `v1`
- checkout to `v1`
- push the branch to GitHub
    - from now on, this is the new home for all further changes related to the initial version
    - mark the branch as `protected` to avoid deleting it by accident
- create a `v1.0.0` in the main branch
- push the tag
    - at this point, clients that previously had a pseudo-version `v0.0.0-etc` and used autoupdates, were migrated to `v0.1.0`, and continue using it
    - clients that haven’t attempted updating, or use vendoring, remain on a pseudo-version `v0.0.0-etc`, and will continue using it
    - even if someone updates a client, as the `v1.0.0` points to the same stable code, so nothing breaks
- optionally, update clients explicitly, if needed
    - at this point, any client that has been migrated to `v1.0.0`, will not start using any other major version unless explicitly told to do so using `go get` with an explicit tag or `latest`, and import of the versioned module
- checkout out back to the main branch
- branch off of the main branch to make a change to the `go.mod` file
- increment the version of the module in `go.mod`, i.e.:
    - `module github.com/myorg/mylib/v2`
- commit, push
- open a PR targeted to the main branch
- pass review, merge the PR
- pull the updated main branch
- create a tag with the incremented major version, `v2.0.0`
- push the tag
    - don’t worry about clients. Unless explicitly updated, they remain on the previous stable version, `v1.y.z`
- from now on, you’re free to make any breaking changes
- from now on, follow the processes described in sections below:
    - when need to make a minor or patch change, see [Changes Within One Major Version](#changes-within-one-major-version)
    - when need to create a new major version, see [Introducing a New Major Version](#introducing-a-new-major-version)
- celebrate success.

Although this process may seem a bit complicated, it's rather not, and is required only once for a module. By doing this way we can be sure the users of a package won't be accidentally affected.


### Changes Within One Major Version

Assuming that the initial transition has been done, making a new `MINOR`- or `PATCH`-level release within the same major version is straightforward. On average, it looks like this:
- branch off of the major version’s home branch
- make changes
- commit, push, repeat
- open a PR targeted at the major version’s home branch, pass review, merge the PR
- pull the updated major version branch
- depending on the level of change (`MINOR` or `PATCH`), figure out the last version for that level
- create a tag with the incremented version for the level
- push the tag.


### Introducing a New Major Version

Introducing a new major version involves a little more steps as compared to minor/patch. However, on average, this should probably happen not too often, a couple of times a year. The frequency of this procedure depends on the aggressiveness of the release cycle.

So, **assuming that the project has already used another major version before**, this is is what it looks like:
- suppose the current version is `v2`, and a new major version, `v3`, should be introduced
- before any changes, create a branch named after **the current major version**, based off of the most recent version of `main`
    - in this case, it's `v2`
- push the branch to GitHub (or the service you use, obviously)
    - additionally, if the functionality of the service allows so, mark the branch as `protected` to guard against an accidental force-push or removal, as this branch is, from this moment, essentially, the home for the previous major version for as long as that version will be maintained
- figure out the last minor version for this major version, `v2`
    - let's suppose it's `v2.3.0`
- from the new home branch of this version, the `v2` branch, create a new tag with the incremented minor version
    - in this case, it's `v2.4.0`
- push the tag
- branch off of the `main` branch to make a change to the `go.mod` file
    - this should be a regular, short-lived development branch
- increment the version of the module in `go.mod`
    - i.e. `module github.com/myorg/mylib/v3`
- commit, push
- open a PR targeted at the main branch
- pass review, merge the PR
- pull the updated main branch
- create a tag with the incremented major version, `v3.0.0`
- push the tag
- make coffee.


## Notes on Automation

For a developer, thinking of automating a repeatable, prone to human errors, or just boring process is natural.

However, to better understand the process, it's worth practicing it to a point when you know what the steps are, what *exactly* they do, and why a certain step is needed. Only after having practiced it enough, when you fully understand the task, attempt automating it. Learning the process by hand gives a sense of what is going on, and helps in understanding of what to do if something goes wrong, or if it does not work as expected. Automation based on a solid experience is more likely to be solid and robust. Try doing this earlier, and the flaws in your knowledge leak into the implementation, making it flawed too, in some way or the other. **This is a general principle of automation**:

> Whenever the legal moves of a formal system are fully determined by algorithms, then that system can be automated.
>
> – John Haugeland, Artificial Intelligence: The very idea (1985)

In the context of versioning, there are some steps that can potentially be automated, although not all of them seem to require it. For example, the initial transition to Modules highly depends on each particular project. Attempts to generalise it would require making lots of assumptions and guesses about the state of a project before conventing it into a module. As this process is required only once ~~in a blue Moon~~ per project, there is little to zero value in automating it beyond what the Go toolset already offers.

Potentially, the task of introducing a new major version might be a good candidate for future automation. The procedure described in the corresponding section is straightforward, and comes down to:
- determining the current `MAJOR` version, `vN`
- creating a major version branch for keeping the state of this version, `vN`
- pushing the branch
- determining the current `MINOR` version, `vN.M.P`
- switching to the branch
- tagging it with the next `MINOR` version, `vN.M+1.0`
- pushing the tag
- switching back to the main branch
- branching off of it
- increasing the `MAJOR` version number in the `go.mod` file, `vN+1`
- pushing, merging, pulling the main branch again
- tagging it with the incremented version, `vN+1.0.0`
- pushing the tag.

On the other hand, creating a new major version is an important process, a significant step in the release cycle, and it is non-linear by its nature. It might include some extra steps requiring human interaction, approvals, or anything else. As long as a tool is able to *reliably* do the work, and *properly* handle *potential* failures, it might be useful. Otherwise, the world is already full of sub-optimal and poor programs, and it won't benefit from one more piece of software, "hacked" over night.

Another part that can be automated is introduction of a new `MINOR` or `PATCH` release within the same `MAJOR` version. This process is mostly the same as with other languages, with the only difference being that if an update is being introduced to a major version that is not the current major version, a tool must take into account that a new tag must should be created from the corresponding major branch.

Some steps can definitely be automated though. We first need to seek for the most likely sources of errors. An obvious candidate, and the most boring thing to do, is figuring out the previous version tag and creating a new tag with a proper version. It's a simplest step, but its simplicity is deceiving, as messing it up once could cause issues to the users of a project. That's why having a command that shows the current and next versions would be helpful.

Stick to the KISS principle and UNIX philosophy. There is a set of powerful tools which are available at our disposal, without the need to reinvent the wheel. Some of these include:
- `make`
- `git`
- `awk`
- `sed`.

With some thought and testing, a library can be provided with a minimal set of `PHONY` targets in the `Makefile` that simplifies calculation of a version number. For instance:
- `make next-minor`
    - returns the next minor version for the current version (based on the branch, existing tags and `go.mod`)
- similarly, `make next-patch`
- `make release-minor`
    - in addition to properly calculating a version, it creates and pushes a tag
- similarly, make `release-patch`.

Even such a minimal set of tools is enough to simplify everyday (in reality, weekly) maintenance of a library to not worry about the process much, if at all.

## Conclusion

This section has shown that choosing and implementing a good approach to versioning is important for the lifecycle of a project, regardless of its size.

A versioning strategy has effect at various levels, ranging from what branch developers start off and merge into everyday, all the way up to what end users of the software see by default when they're visiting the homepage of the project's repository. A good strategy, as all good things, reduces the mental load and saves time, whereas a poor strategy confuses and requires extra steps to get something which should be accessed easily.

A good strategy also works both short- and long-term. In short-term, it allows for simple and straightforward introduction of a new major version. Long-term, it allows maintaining multiple major versions easily for as long as you should as a responsible developer. A good approach is also universal, as it does not depend much on a particular technology.

The third option given here, the Major Version Branch with Rolling Main Branch, looks like a candidate for a good one. It does not actually depend on anything specific to Go. It works for other technologies as well as it does with Go. It's good short- and long-term, and it does keep the conventions that the community is accustomed to, along with keeping the meaning of the main and default branches. It addresses the needs of versioning without raising new issues. This is a good sign.


---

PS. Hi Joseph! 👋😉

To be continued...
