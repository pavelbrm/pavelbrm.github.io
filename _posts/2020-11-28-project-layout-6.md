---
layout: post
title: "The Style Guide: Project Layout 6 - Summary"
category: Golocron
tags: [golang, m1u1]
---

> When you're a carpenter making a beautiful chest of drawers, you're not going to use a piece of plywood on the back, even though it faces the wall and nobody will ever see it. You'll know it's there, so you're going to use a beautiful piece of wood on the back.
>
> For you to sleep well at night, the aesthetic, the quality, has to be carried all the way through.
>
> – Steve Jobs

This short post concludes the series of drafts for the first unit of the book. It's the summary of what we have discussed recently. Here you can find an almost complete list of the recommendations introduced in previous articles.

<!--more-->

![](/assets/m1u1_6_3.jpg)

## Summary

We began the unit with the excitement of starting something new. However, it's often the case that this excitement fades away quickly, if the project has not been properly cared of.

The guidelines given in this unit are only the very tip of the iceberg of preparing and maintaining a project in a healthy and consistent state. We briefly discussed techniques that apply at the highest, the project level. Project design depends on decisions made at each level. In next units, we go deeper, layer by layer, to learn more about better design choices.

Consistently following the guidelines requires an effort. At times it may seem too much to pay attention to, or too meticulous. However, it pays off in long term, and it always does. One of the best things one can do for their future self is to plan and prepare upfront such that as less as possible changes are required in places which _should be_ set and just work. But that "just work" **is**, in fact, a consequence of these activities which are often invisible when done, and very noticeable when forgotten, ignored, or neglected.

At the end of the unit it is worth summarising all of the suggestions and recommendations given so far, compiled in one list:

1. [In general and overall, strive for a good project layout](https://blog.pavelbrm.com/golocron/2020/08/08/project-layout-intro/#introduction).
2. [For any kind of project](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#common):
   - [provide files required for the project](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#provide-files-required-for-a-project)
   - [keep the number of root elements at minimum](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#keep-the-number-of-root-elements-at-minimum)
   - [keep any level clean and up to date](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#keep-any-level-clean-and-up-to-date)
   - [don't create artificial hierarchy](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#dont-create-artificial-hierarchy)
   - [group non-code files by their purpose](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#group-non-code-files-by-their-purpose)
   - [provide the Makefile](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#provide-the-makefile)
   - [store credentials somewhere else](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#store-credentials-somewhere-else)
3. [For a library](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#library):
   - [provide a good README file](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#provide-a-good-readme-file)
   - [keep the top-level list of objects of a manageable size](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#keep-the-top-level-list-of-a-manageable-size)
   - [provide examples](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#provide-examples)
   - [provide benchmarking information](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#provide-benchmarking-results)
4. [For a single application](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#single-application):
   - [use `cmd` directory for entry points (`main.go` files)](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#use-cmd-directory-for-entry-points)
   - [use `bin` directory for binaries](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/#use-bin-directory-for-binaries)
5. [When working with a monolithic repository, bear in mind the following](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#monolithic-repository):
   - [avoid naively throwing in everything in to one repository](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#naive-monorepo)
   - [service-based monorepos may work, but they are harder to maintain long-term](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#service-based-monorepo)
   - [similarly, entity-focused repositories are especially prone to dependency cycles](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#entity-focused-monorepo)
   - [prefer a structured monolithic repository, and ensure that](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#the-structured-monorepo):
     - [each package is carefully placed in the tree](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#the-layout-of-a-structured-monorepo)
     - [utility code is being maintained as own standard library](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#organise-utility-code-as-own-standard-library)
     - [there is a sandbox for code with breaking changes](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#have-sandbox-for-breaking-changes)
   - [when even more flexibility is required/desired, consider](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#monorepo-additional-chapters):
     - [using nested modules within a single monorepo](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#more-structure-with-nested-modules)
     - [or even using multiple monolithic repositories](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#multiple-monorepos)
       - [some of which are "simple", single-module monorepos](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#multiple-single-module-monorepos)
     - [and finally, there is always the most flexible option with multiple monorepos composed of nested modules](https://blog.pavelbrm.com/golocron/2020/09/21/project-layout-3/#multiple-monorepos-with-nested-modules)
6. [When versioning your Go project, remember of these](https://blog.pavelbrm.com/golocron/2020/10/13/project-layout-4/#versioning-and-go):
   - [the main branch is usually considered to be the source of truth](https://blog.pavelbrm.com/golocron/2020/10/13/project-layout-4/#the-main-branch)
   - [the default branch should be the same as the source of truth](https://blog.pavelbrm.com/golocron/2020/10/13/project-layout-4/#the-default-branch)
   - [avoid the directory-based approach to versioning unless you absolutely must](https://blog.pavelbrm.com/golocron/2020/10/13/project-layout-4/#directory-based-versioning)
   - [similarly, there are few reasons for using the simple branch-based approach](https://blog.pavelbrm.com/golocron/2020/10/13/project-layout-4/#branch-based-versioning)
   - [**use** the branch-based approach with rolling main branch](https://blog.pavelbrm.com/golocron/2020/10/13/project-layout-4/#branch-based-with-rolling-main-branch)
7. [When working on code and then preparing a new version, remember to](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#notes-on-release-notes):
   - [write good commit messages](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#write-good-commit-messages)
   - [use annotated tags](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#use-annotated-tags)
   - [maintain the changelog, keeping in mind that](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#maintain-the-changelog)
     - [changelog is for people](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#changelog-is-for-people)
     - [there must be a record for each version](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#log-each-version)
     - [last version comes first in the list](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#last-version-comes-first)
     - [each release must include the date it's released](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#show-the-date-of-release)
     - [changes should be grouped by the type](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#group-changes-by-type)
     - [the style and wording should be consistent](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#be-consistent)
     - [and the information you add to the changelog is relevant](https://blog.pavelbrm.com/golocron/2020/10/31/project-layout-5/#provide-relevant-information).

An insightful reader might have already noticed that there is something common among the guidelines proposed here. The underlying principles for the suggestions in this and next units are basic and simple. Everyone knows about them, but fewer people do actually follow them in real life. As a reminder and a hint, they're listed below:

- Do what needs to be done, and don't be lazy.
- Anything you do, do it at the best level.
- Especially, when nobody is watching.
- Learn and then follow rules.
- Always do your homework.
- Never stop learning.
  - Regularly update your understanding of the things you already know.
  - Learn something new.

In the next unit, we take a step forward, and introduce guidelines on various aspects of package design in Go.


---

To be continued...
