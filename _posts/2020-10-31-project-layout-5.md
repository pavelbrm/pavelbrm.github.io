---
layout: post
title: "The Style Guide: Project Layout 5 - Notes on Release Notes"
category: Golocron
tags: [golang, m1u1]
---

> What you don't do can be a destructive force.
>
> – Eleanor Roosevelt

A significant part of the release process is letting your users know about what's changed in the software. This post highlights the importance of such commuication, and provides some ideas and guidelines on how to make it more efficient.

<!--more-->

![](/assets/m1u1_5.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Before We Begin

In the previous [draft](https://blog.pavelbrm.com/golocron/2020/10/13/project-layout-4/) we discussed how to prepare and release a new version. But that's only one, and rather boring, part of the release cycle, and it should _just_ work. A more important part is what should happen right after, or even better had happened _before_ you pushed a new tag. There should be a comprehensive, readable, and complete list of changes, explaining what has been changed, if needed - why, and sometimes - how.

So the topic of this conversation is communicating changes in a project to its users. I hope you find it helpful.

- [Notes on Release Notes](#notes-on-release-notes)
- [Write Good Commit Messages](#write-good-commit-messages)
- [Use Annotated Tags](#use-annotated-tags)
- [Maintain the Changelog](#maintain-the-changelog)
  - [Changelog is for People](#changelog-is-for-people)
  - [Log Each Version](#log-each-version)
  - [Last Version Comes First](#last-version-comes-first)
  - [Show The Date of Release](#show-the-date-of-release)
  - [Group Changes by Type](#group-changes-by-type)
  - [Be Consistent](#be-consistent)
  - [Provide Relevant Information](#provide-relevant-information)
- [Conclusion](#conclusion)


## Notes on Release Notes

One more thing to talk about before wrapping up the project layout guidelines, and a follow up for the previous topic, is release notes. We will briefly discuss some ideas for better communicating what's changed in a project.

This is important for both, internal and external audiences. For a private project in an organisation, the internal audience is developers who use the software; the external audience is management and other potentially interested parties. When done well, and everyone knows where to look at, release notes that are published regularly and reflect the work that has been done, serve as a summary and description of the outcomes.

In an open-source project, the internal audience is committers and maintainers; the external audience is users of the project. For the internal audience, often working from all over the world, this helps with keeping everyone in sync with what is going on in the project. For the users, it's the only convenient way for knowing about changes, and it provides information required for planning and deciding on when, and often - how, to perform an update. Of course, in both cases, private and public projects, audiences may, and often do, overlap.

Although the reasoning above just touches on the topic, this is already sufficient to understand the importance of preparing and publishing some form of release notes. This should be done for each and every release. Needless to say that a `MAJOR` version *must* come with a comprehensive description of the changes it contains.

The main guideline of this section is summarised as **provide your project with a changelog**, informing the users of the software about the changes you've made. A few practices help in achieving this:
- being generally disciplined, in the process, keeping things in a good order
- writing good commit messages
- using annotated tags
- and sticking to good practices of maintaining a changelog.

A changelog is not release notes though. A changelog is more on the technical side of a project, while release notes are more on the product, or even marketing side. For software projects whose target audience is developers, the terms may be interchangeable, but not necessarily are.

As this book is more about the technical side of things, we're focusing on maintaining a good changelog, and related activities. Let's look at these a bit closer.


## Write Good Commit Messages

Perhaps there are not so many topics in software development with such a long history of attempts to establish good practices, and make everyone in a team to follow them, as writing good commit messages.

One of the famous and best resources that summarises what needs to be said about writing commit messages, is [this article](https://chris.beams.io/posts/git-commit/). If you haven't seen and read it yet, stop reading this, and don't get back until that article is fully read.

A quick summary on how to do better with commit messages:
1. [Separate subject from the body with a blank line](https://chris.beams.io/posts/git-commit/#separate)
2. [Limit the subject line to 50 characters](https://chris.beams.io/posts/git-commit/#limit-50)
3. [Capitalise the subject line](https://chris.beams.io/posts/git-commit/#capitalize)
4. [Do not end the subject line with a period](https://chris.beams.io/posts/git-commit/#end)
5. [Use the imperative mood in the subject line](https://chris.beams.io/posts/git-commit/#imperative)
6. [Wrap the body at 72 characters](https://chris.beams.io/posts/git-commit/#wrap-72)
7. [Use the body to explain what and why vs. how](https://chris.beams.io/posts/git-commit/#why-not-how).

This is important for many reasons, such as overall order, clean history, transparency and the ease of working with the commit tree. Good commit messages become even more important as it reduces the effort required to gather the list of changes for the changelog.


## Use Annotated Tags

By default, create annotated Git tags.

As you know, Git offers two flavours of tags - lightweight and annotated. A lightweight tag is a pointer to a specific commit. An annotated tag, on the other hand, is a full Git object, which means it contains the following information:
- the checksum
- who has made the tag, when, and their email
- a tagging message (pretty much a commit message, hence same rules apply)
- can be signed with GPG (therefore, verified).

The official [Git documentation recommends](https://git-scm.com/book/en/v2/Git-Basics-Tagging) using annotated tags.

Since each tag carries all identity information with it, this helps in figuring out what was changed, when, where and by whom. Combined with good commit messages, annotated tags provide enough information for preparing the list of changes for the changelog. The worst case scenario would be when in order to create a changelog, the information contained in the repository was not enough, and you had to use multiple sources, including the task tracker.

By writing good commit messages and using annotated tags, you have helped your future self with preparing for writing a good changelog.


## Maintain the Changelog

To keep the audience of a project informed about changes, **maintain the changelog**.

What makes a good changelog? What is important, and requires a special attention? Consider the following suggestions:
- remember that the changelog is for people
- add an entry for each version
- keep records in the descending order, i.e. the latest version comes first
- always include the date of a release
- group changes by their type
- be consistent with the style and wording
- provide only relevant information.

The sub-sections below go into more detail on these suggestions.


### Changelog is for People

First and foremost, **changelogs are** written to be **read by humans**, not machines. Ultimately, the changelog should answer the question: **will my software work if I update this dependency**, or **does the update break anything**?

Having developed and maintained software that was relied upon in everyday work by developers, both internal and external, it’s safe to say that you either only provide good and valuable changelog and/or release notes, or nothing at all. Something in the middle, an automated non-sense or irrelevant information are just noise that no one is interested in.

The changelog is where engineers look at while making decisions about using a library or updating it. Keep in mind that "engineers" means not only software developers that use the project in their environments; the target audience, depending on the popularity and how widespread the project is, also includes:
- system administrators and operations engineers
- SRE and Devops people
- application security analysts and engineers
- and whoever is working with them, such as support and sale teams, lead engineers, product managers, designers, etc.

Given this, make sure that the changelog provides what it is expected to provide  - the information about what's changed, when, how it affects the user, and what to pay a special attention to.


### Log Each Version

Add each version to the changelog.

Although this is self-evident, and does not require further explanation, it's worth reminding that the user should be able to see what has been changed in each particular version of the software. If the software is a library, especially a public one, and is used by other developers in their products, this is a must.


### Last Version Comes First

Always add the latest version at the top of the changelog. In other words, it's similar to a stack data structure, versions in the changelog are listed in a reverse order.

This is important for a few reasons:
- For the first, the information of interest should be accessible as quickly and easily as possible.
- Secondly, in most cases, the user is interested in recent changes rather than in earlier versions of the software.
- And finally, this helps in saving a tiny but important bit of mental energy, as neither the user, nor a maintainer, has to scroll the entire list to get to the point when reading or updating the changelog.


### Show The Date of Release

Provide each version with the date of release. Use the ISO-8601 format, aka `YYYY-MM-DD`.

This is essential for keeping track of changes, and it's hard to tell what's more important - the version number, or the date when it was released. Without either, the information is not complete. The date of release establishes a concrete point in the timeline, and allows for relating to a particular version, and correlating between the date and various events.

The format requirement helps to avoid confusions caused by the variety of regional formats. The ISO-8601 format is well-known, simple and easy to understand. Just stick to it.


### Group Changes by Type

To simplify reading of a changelog, group changes by the type of a change. This is helpful because structured and organised information is easier to understand, as well as to notice something important.

Some of the most common groups, along with the corresponding types of changes, and the relation to Semantic Versioning, are shown in the table below. For Semver, where multiple options are available, they are given in the order of preference. The table shows an approximate relation, just to visualise how types of changes and version numbers correspond.

Group | Type of a Change | Semver
 --- | --- | ---
Added | New Features | `MAJOR`, or `MINOR`
Changed | Improvements | `MINOR`, or `MAJOR`
Fixed | Bug Fixes | `PATCH`, or `MINOR`
Security | Vulnerability Fixes | `MINOR`
Deprecated | Deprecations | `MINOR`
Removed | Removed Features/APIs | `MINOR`, or `MAJOR`

For most use cases, these groups should be enough.

However, a project may support multiple platforms (i.e. operating systems and processor architectures, more on this in Unit 2). In addition to changes common across all supported platforms, something different can be added/updated/fixed for a particular architecture. In such case, two options are available:
- prefix a change that is specific to a particular platform with its name
    - e.g. for an `Added` change for the Mac

```txt
Added:
    ...
    Mac: Notarisation
    ...
Fixed:
    ...
    Linux: High memory usage when working with large files
    ...
```

- alternatively, group platform-specific changes together, after the list of common changes
    - e.g. for a `Fixed` change for Linux, a change should be listed in the `Fixed` sub-list for the `Linux` list, like

```txt
Common:
- Added
    ...
- Updated
    ...
- Fixed
    ...

Mac:
- Added
    - Notarisation
    ...

Linux:
- Fixed
    - High memory usage when working with large files
    ...
```

There are many different ways of grouping, and at some point it might be tempting to add one more category, or a grouping criterion. Resist the temptation, and ask a question "Is this group essential?". Highly likely it's not, and such a change should belong in one of the existing groups. The importance of this is the subject of the [Be Consistent](#be-consistent) guideline.

A special group of changes, however, makes an exception. A natural part of the project and product development process is upcoming changes that are not going to be included in the next one or a couple of releases. These are usually new features, available for canary testing, and while being developed, they aren't included in a release. If this approach is used in your project, keep these changes at the very top of the changelog, before the latest version. Users and developers would benefit from knowing what's coming, and they would be able to plan accordingly, or become early adopters of new features. As a positive side effect, you can receive feedback sooner.


### Be Consistent

Use consistent wording, formatting, and grouping. This is especially important when working in a team of developers. Make sure everyone follows the same visual and writing styles, and the changelog looks nice, and is coherent and consistent. For the reader, it should appear as if it was written by one person.

Specifically, ensure the following:
- all dates are always in the ISO-8601 format, i.e. `YYYY-MM-DD`
- all versions are always in the Semver format, i.e. `vX.Y.Z`
- groups are either all Simple Past verbs, or plural nouns from the table [given here](#group-changes-by-type)
- the list of groups is relatively persistent
- changes are articulated and communicated clearly
- sections with versions are clickable, for ease of navigation
- strive for short and concise descriptions, using, whenever possible, [Plain English](https://en.wikipedia.org/wiki/Plain_English), or the plain version of the language of your choice.


### Provide Relevant Information

Provide only relevant information that is important for the user or the developer. In other words, keep the signal vs. noise ratio high.

For example, here is what should be included in a changelog, in addition the main contents:
- changes in the public API
- increase/decrease in performance
- any implementation-specific changes that may affect the user, especially, in a negative way
- updates to dependencies if they contain any of the following:
    - any negative impact on performance
    - significant increase in the size of the software, aka bloated libraries
    - significant changes in licensing, including dependencies (yes, you're responsible for this too).

Things that are better be left outside of a changelog:
- internal implementation details that don't affect the user/developer in any way
- regular dependency updates
- refactoring without a noticeable change in functionality/performance.

It's worth noting that sometimes changes are so important that it makes sense to provide a description of the actions required from the user. For non-technical users this is normally communicated through release notes and documentation. For developers, the changelog might be a good place to look at first. However, the home for detailed descriptions, explanations, instruction, migration steps, etc is still documentation. A quick summary and a link to the place in the docs is a good way to start.


## Conclusion

What is the most important ingredient in a relationship? Trust. A significant contributor to trust is communication, clear, transparent and honest. As long as involved parties trust each other, the relationships continues to last.

There is a relationship when someone uses software; it's between the user and the developer. The user might be a single person, or a larger group; similarly, there might be a single developer, or an organisation. The important thing is whether there is trust in the relationship.

It is the features and quality of the software what forms the foundation for the trust. If it works, and works well, then users remain loyal. The documentation plays an important role, and is a big part of communication between the developer and user. So far so good.

When is trust lost, assuming there is no betrayal or lie involved? One possible reason is when expectations are not met. The other is, however, when something unexpected happens, regularly, without a notice; when something changes, but neither the changes, nor the reasons for them have been communicated, or communicated poorly.

A changelog is the way for addressing the issue. A well-written changelog communicates what's changed clearly, transparently and honestly. This is why, if you want to build lasting relationships with your users, you need to maintain a changelog.

The ideas and suggestions in this section are based on learnings and experiences gained over many years, and from various sources. A great resource for reference, and where some of the ideas were looked at for inspirations, is [Keep a Changelog](https://keepachangelog.com/en/1.0.0/). Another good place to learn from is real-life examples. The list below includes some that are worth looking at:
- [iced](https://github.com/hecrj/iced/blob/master/CHANGELOG.md)
- [tikv](https://github.com/tikv/tikv/blob/master/CHANGELOG.md)
- [druid](https://github.com/linebender/druid/blob/master/CHANGELOG.md)
- [MeiliSearch](https://github.com/meilisearch/MeiliSearch/blob/master/CHANGELOG.md)
- [docker-ce](https://github.com/docker/docker-ce/blob/master/CHANGELOG.md)
- [keep-a-changelog](https://github.com/olivierlacan/keep-a-changelog/blob/master/CHANGELOG.md).

The ultimate goal is to let users of your software know about the changes you make. If the suggestions given here don't suit, or too resource demanding, focus on communicating changes to the user in a simplest possible form. While the way it's done is important, it's secondary to the fact that it's done at all. Trust is hard to gain, but easy to loose. Make sure that, while making changes to increase trust, the way the changes are communicated support that increase.


---

To be continued...
