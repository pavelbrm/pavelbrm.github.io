---
layout: post
title: "The Style Guide: Project Layout 3 – Monorepos"
category: Golocron
tags: [golang, m1u1]
---

> There's a big difference between knowing the name of something and knowing something.
>
> – Richard P. Feynman

This post features another draft of [the book](https://blog.pavelbrm.com/programming/2020/06/27/about-software-development-with-golang/) [I've been working on](https://blog.pavelbrm.com/golocron/2020/08/08/project-layout-intro/) [for several months already](https://blog.pavelbrm.com/golocron/2020/08/22/project-layout-2/). This time around we take project layout further and talk about monolithic repositories.

<!--more-->

![](/assets/m1u1_3.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Before We Begin

Having established a base line with recommendations on organising library and single-application projects, we're ready to get a little bit deeper and look at how to better organise monolithic repositories.

As we're currently still at the project level, most of advice given below is language-agnostic, with a slight hint of Go when it's necessary and beneficial.

Hope you find something interesting and useful.

- [Monolithic Repository](#monolithic-repository)
- [Naive Monorepo](#naive-monorepo)
- [Service-Based Monorepo](#service-based-monorepo)
- [Entity-Focused Monorepo](#entity-focused-monorepo)
- [The Structured Monorepo](#the-structured-monorepo)
- [The Layout of a Structured Monorepo](#the-layout-of-a-structured-monorepo)
  - [Use Appropriate Position at the File Tree](#use-appropriate-position-at-the-file-tree)
  - [Organise Utility Code as Own Standard Library](#organise-utility-code-as-own-standard-library)
    - [Utility Code](#utility-code)
    - [Details and Discussion](#details-and-discussion)
    - [In Practice](#in-practice)
  - [Have Sandbox for Breaking Changes](#have-sandbox-for-breaking-changes)
    - [Details and Discussion](#details-and-discussion)
- [Monorepo: Additional Chapters](#monorepo-additional-chapters)
  - [Open Questions](#open-questions)
  - [More Structure with Nested Modules](#more-structure-with-nested-modules)
  - [Multiple Monorepos](#multiple-monorepos)
    - [Multiple Single-Module Monorepos](#multiple-single-module-monorepos)
    - [Architectural Monorepos](#architectural-monorepos)
    - [Technology-Specific Monorepos](#technology-specific-monorepos)
    - [Multiple Monorepos with Nested Modules](#multiple-monorepos-with-nested-modules)
- [Closing Notes](#closing-notes)


## Monolithic Repository

_Warning:_ Do not confuse a monolithic repository with a monolithic application. They're absolutely different things.

Unless you're in one of the [FAANG](https://en.wikipedia.org/wiki/Big_Tech) or similarly-sized companies, or just don't like the idea, there are probably few technical reasons, if any, against using the [monorepo](https://en.wikipedia.org/wiki/Monorepo) approach to organising a project that has multiple moving parts. And Go projects can benefit quite well from it.

Moreover, teams are likely to benefit from this technique as it has several strong supporting arguments like:
- simplified workflows for CI/CD and testing
- reduced burden with managing internal dependencies
- significantly less chores with permissions/access
- unified tooling
- easier to package/ship and so forth.

Of course, there are some limitations and downsides, but they are not in effect until after at least a couple of hundreds of thousands SLOCs have been committed in a couple of millions of commits.

A valid concern may be access control, but that's only partially true. If the security policies at a company do make sense and implemented and respected well, there is little to worry about. Experience of companies for which security **is** the biggest asset, suggests that the attitude and mindset are what matter. When everyone is accountable for what they're doing, there is no need in introducing artificial barriers.

The decision on whether to use a monorepo or not is upon you, the reader. The goal of this section is not to make you use this approach, although it makes sense rather often than not. This work covers various aspects of software development, is based on experience, and provides practical advice and guidelines. Several services will be introduced in the third module (it's quite far from this point), and the materials *do use* a monolithic repository.

With that in mind, further we talk about how to structure a monorepo in a way that makes the experience of using it better. As there are several ways to approach organising a monorepo, and to understand what makes a good one, it's worth getting familiar with some of them. Finally, one reasonable approach will be introduced, and we'll learn more about it.

There is no official classification for monolithic repositories. Yet some of common traits can be derived after investigating and working with different projects. The experience shows that most of the time a monorepo is of one of the following types:
- naive
- based on services
- focused on entities.

Let's have a quick look at these and consider their pros and cons. Then we introduce an alternative, the structured approach.


## Naive Monorepo

A naive monorepo is exactly what its name says. It's what usually happens when a team decides to use a monorepo, without taking the time to learn and putting a good thought into how to make it work:
- a new repository is created
- then each of the existing services is copied over to the new repo, as is
- done
- alternatively, everything can be moved into one of the existing repositories, but this doesn't do much of a difference.

As a result, the team got just a rather large directory with lots of semi-independent parts, lots of duplicated scripts, files, words, whatnot. One of the immediate benefits _seems to be_ that now something from service A can be imported by service B, but this will quickly become troublesome, especially if the original repositories did not follow the suggestions (such as) listed above, which is highly likely.

This approach has many downsides. Besides being messy and containing many duplications, it is the shortest path to creating dependency cycles. This can often lead to either a poor hierarchy and structure that is created just for the sake of breaking a cycle, or even to concluding that the monorepo was a mistake, and should be avoided. Neither of the those leads to productive and efficient collaboration.

It's hard to justify following this way. Instead, plan the migration, and then execute the plan iteratively. You need to come up with a good structure that suits well and will serve your needs for years. Don't rush, think, plan, try, and only then execute. Not the other way around.

After having realised that this way is not better than it was before the move, the team may decide to organise the repository somehow. One of obvious ways is to use a service as a boundary and splitting criterion. This is how a naive monorepo may evolve to a service-based one.


## Service-Based Monorepo

The service-based approach is a slight improvement of naive. The main difference is that some of the duplicated components among services are unified, e.g. CI and building routines, but the codebase continues using services as boundaries for packages. Put simply, each folder at the root level contains a service along with everything that's in the service's scope - data types, business and transport logic, etc. When a service needs something that's already implemented somewhere, it just imports that. New functionality is being developed within the boundaries of a service.

While it might work for some time, such a repository still has exactly the same major downside as naive - it's too easy to end up with a dependency cycle, more so when you try to re-use some code with business logic. Also, there isn't much of an order, since data, logic and utility code are spread across the entire codebase.

A few other serious downsides enter the stage at this point, caused by importing different parts of various services:
- increased sizes of binaries
- increased compilation times
- not always clear what to do with tests.

As the project evolves, it might seem natural to think of grouping code based on entities it belongs to. Here is how a monorepo may transform into entity-focused.


## Entity-Focused Monorepo

The entity-focused technique is organising code around a particular entity. Packages are often created for different units of the business domain of a project, such as `user`, `photo`, `library` and so forth. Developers add logic into appropriate packages, and then use it in services.

This approach is a bit better than the previous two. It allows for working on services' part separately from the business logic, if implemented correctly.

Still, there are two potential problems:
- different levels of representation and responsibility could be mixed together, such as data types, methods for accessing storage, business logic and transport details
- a major risk in creating cycles, specifically at three levels:
    - entity - when several entities depend on each other
    - business logic - when a business process depends on the logic of several entities
    - transport and representation - when representations of several entities/processes depend on each other.

The second issue comes from a fact that it's rare for entities, business logic and their representations to be independent from each other, i.e.:
- entities often aggregate or are parts of other entities
- business logic for one process depends or is included into another process
- representation for one area of the domain requires other parts.

Is there a solution to address these and the problems described above? How to organise a repository in a way which reduces the dependency cycle risk to a minimum (or better to zero)? How to organise a repository in a way when it's easy to re-use entities and logic in different services? How to make developers happier, and services better organised?

The first problem is to be addressed by making better package and architectural design decisions. The former is the subject for Unit 2 in this module, the latter is the topic for Module 3. Some suggestions will be given shortly, in the very next section.

There is no ultimate solution, of course. But there is an option which, if implemented carefully and everyone respects the process, can help to achieve better efficiency and maintainability.


## The Structured Monorepo

The structured approach is based on grouping code by responsibilities, and levels where objects play their roles. In other words, things are put together by what they do and where they belong:
- data layer (models)
- database layer (repositories)
- business processes (controllers)
- transport representations (handlers) and so on.

By doing this way, we avoid problems described above, and get some additional benefits:
- at the model level, any model can safely relate to another
- at the business process level, any process is free to do the following, without the risk of introducing a cycle:
    - use any model or combine multiple
    - include any other business process, or be included in other business process, as a step
    - interact with database representations for the models it works with
- similarly, at the transport level, any service can use or combine various business processes, and more.

The insightful reader might have already noticed that this has many similarities with properly designing and structuring a good single service. Moreover, if a single service has followed this way, adding another service wouldn't even require to do anything, since the project, hence the repo, is already prepared to accommodate as many applications as needed.


## The Layout of a Structured Monorepo

Everything that has been discussed about different layouts so far comes together and applies to a monolithic repository. Taken into account, implemented and followed carefully, the practices establish the foundation for a good monorepo.

This is what a project may look like at this point:
- the documentation is provided and up to date
- the list of elements at any level is of a reasonable size
- all maintenance scripts and other non-code files are organised
- the entry points to services are located in the `cmd` directory
- the binaries automatically go and picked up from the `bin` directory
- code that implements the project's data and logic is grouped by responsibilities and roles.

A few questions inevitably arise while working on a reasonably large project within one or a couple of teams:
- Where do we put code that is meant to be used by many services?
- Where should utility code go?
- How to gradually and safely introduce something new or a breaking change?

There is no simple and direct answer. It's also where good planning and thinking should be done, as well as some exceptions. With that in mind, let's consider the following suggestions that help to keep things at the right places:
- use appropriate position at the file tree to reflect the importance of a package
- organise utility code as own small standard library
- keep breaking changes in sandbox.


### Use Appropriate Position at the File Tree

Place packages appropriately in the file tree of a monolithic repository to reflect the importance and nature of a package.

What does it mean and what properties can we use to determine a right placement of a package? There's no hard set of rules. The following heuristics can help in understanding where a package should be placed:
- An approximate position of a package in the dependency graph.
A package that is imported by many other different packages should be located closer to the root of the hierarchy. A rarely imported package is most likely an implementation detail, and should go deeper in the tree.
- The frequency of use.
The position of a package is in direct proportion to of how frequently the package is used.
- The importance of a package.
Something that is unique, and provides and implements an important piece of functionality should be placed closer to the top.
- The level of abstraction and role.
The higher the abstraction, the higher the level at which a package should be placed.

For example, a package with code for converting internal errors from their internal representation to the external, and which is used by the most of the packages that implement application functionality, should be placed at the top level of the structure.

Another example is the packages that _are_ the definitions of the business logic  - packages with models, controllers, and the database layer.

On the other hand, a set of middleware handlers for an API should be located deeper in the tree, as it's used only in the context of API, and only by an instance of API. Similarly, routines for data validation, migrations, etc are better to be placed at the second or third level of the tree.

More on this to come in later units covering package design and architecture of a service.


### Organise Utility Code as Own Standard Library

Organise, treat, and maintain all utility code as a small private extension to the standard library. If possible, consider releasing it open source.

This recommendation sounds controversial, and needs further explanations. To understand it better, we first need to clarify what differentiates utility code from other, and then go deeper into how to apply it in real life.

#### Utility Code

Utility code is code that implements technical details of a process, and is **independent** from the business logic of a project. The independence from the business logic is the crucial part that distinguishes utility code from any other.

The following traits are common for utility code:
- it's independent from any other code besides the standard library and/or itself
- most of the methods operate on built-in types, or on the types from the standard library
- provides common and often used routines
- it can be extracted as a separate library
- it can be open sourced
- provides methods that do not exist in the standard library.

Here are some examples of code that can be part of a private extension to the standard library:
- managing the lifecycle of a process, including proper signal handling, graceful and forced shutdown
- archiving/compressing directories
- extended configuration and methods for the `http.Client`, such as custom transports, file downloading, etc
- handling multipart uploads
- advanced strings manipulation
- some standard and generic data structures and algorithms, such as queues, graphs, lists, and so forth
- concurrency patterns
- file tree traversing utilities and so forth.

Having clarified what utility code is and what is not, we can discuss what to do with it.


#### Details and Discussion

To begin with, it's worth reminding one of the most often repeated mantras in the Go community:

> Prefer duplication over the wrong abstraction.
>
> Sandi Metz, and many gophers.

This is true, and this section should not be considered as something opposite. The author is one of those who respects and follows this advice.

Nonetheless, many rules do have exceptions. It's more about finding what works. What is good for a small project/service, can be a poor choice when applied to multiple services. What's good at a small scale, can significantly complicate things at a larger, and vice-versa.

The definition of utility code given above implicitly prohibits building additional abstractions on top of the standard library code. It can be only considered as an extension.

Duplication works well for a small service, when a small team maintains a couple of medium-sized services. In other words, duplication suits well when it's used moderately and rarely, and in isolation.

Things are different in a project that is supported by one large or several teams, when it's a monorepo, when the number of services grows. The duplication approach is not scalable. Employing it at a larger scale leads to _unnecessary_ duplications, mess in codebase, lack of guarantees, and makes the project prone to errors. The quality of testing decreases.

One of the biggest strengths of Go as a programming language, is that there is mostly one way for accomplishing a task. It becomes almost a requirement when working with many services. There should be _only_ one way to download a file, zip or unzip a directory, parse an authentication header and so forth. Failing to acknowledge this can turn out to be a major source of obscure and nasty bugs at later stages of a project.

So when a project is a monorepo, and has more than one service, the following is true about utility code:
- it will inevitably occur
- it should be reliable and trustworthy
- it should be tested
- it should be maintained
- it should be standardised.

One of the ways to provide these guarantees is to have such code in one place, and being conscious about and responsible for it.


#### In Practice

The guideline can be simply employed like this:
- at the root level, have the `lib` directory as home for utility code
- put packages inside `lib`
- for packages those names clash with ones from the standard library, add a prefix, for example, `x`, `xstrings`
- alternatively, keep names for packages as is, but have an agreement on how a custom package should be named when imported along with a package from the standard library with the same name. Do not use custom names for the standard packages, ever.

This approach also works especially well when implementations of some routines are different between the platforms your project supports.

As a result, the file tree may look like this:
```bash
├── bin
├── cmd
└── lib
    ├── process
    │   ├── process.go
    │   ├── process_darwin.go
    │   ├── process_linux.go
    │   ├── process_posix_test.go
    │   ├── process_windows.go
    │   └── process_windows_test.go
    ├── files
    │   ├── files_posix.go
    │   ├── files_test.go
    │   └── files_windows.go
    ├── http
    │   ├── http.go
    │   └── http_test.go
    ├── os
    │   ├── os.go
    │   ├── os_posix_test.go
    │   └── os_windows_test.go
    └── strings
        ├── strings.go
        └── strings_test.go
```

When applied and followed carefully, this advice helps to consolidate and maintain low level code, providing one way for accomplishing a particular task, giving more guarantees about the quality and correctness of the implementation, and reducing the maintenance cost.


### Have Sandbox for Breaking Changes

Another important aspect that is different when working with a monolithic repository, is introduction of breaking changes, or entirely new code that is not proved to be stable. Within a monorepo, a piece of code is relied upon potentially in hundreds of places, so it's better to test a change in isolation, and only then use elsewhere. How do we do about it in a monorepo?

In a monorepo, have a special place for adding potentially unstable code. An `experimental` or `unstable` directory at the root level would be a good choice. Then, inside that directory, follow similar structure as if it were the root level.

#### Details and Discussion

In a classic scenario, dependency management tools usually solve this problem. A dependency that is used in many places, is updated, and the change is then gradually introduced. Go modules and/or vendoring are handy tools here, and this is one of the main reasons for their existence.

However, these tools are not available anymore for addressing this problem as all code is in a single repository. At least, not directly. It's impossible to vendor a part of a repository, or use a separate import path for a package that is part of a large module.

A solution to this problem has existed for many years, and is in use by various kinds of software, from private projects to the Linux kernel, and many established Linux distributions and software. A common name for the solution is "experimental".

What do we mean by "experimental"?

Of course, there are different stages in release processes, such as development, alpha and beta versions, release candidates and so on. Somewhere between development and beta there is usually an experimental branch, or a stage. After reaching some level of stability, the project transitions to next stage, usually testing, or beta. This is followed by many projects, yet it's not exactly what current advice is about.

This guideline is about having experimental code included in the stable version, and made available for conscious use. If the reader have ever configured and compiled a Linux kernel, they would recall the `EXPERIMENTAL` label for drivers that are included in a stable release, but are still in active development, and included for use as is, without any guarantees.

Similarly, even the most stable version of Debian and many other Linux distributions have an `experimental` section in their repositories, which contain early releases of software. Such sources are turned off by default, but the user is free to use it.

So here's what to do in a monorepo. When introducing a breaking change or something entirely new, and such code is not guaranteed to work or work correctly (hence it's not for general use), consider this:
- add a package for new code, if it doesn't exist yet, to the `experimental` or `unstable` directory
- add new code to the package
- use it, test it, change it, prove it works
- once confirmed working:
    - for new code, move it to the stable tree
    - for changes to existing code, depending on the situation
        - move changes to the stable tree and make necessary adjustments
        - alternatively, use type aliases to redirect uses from old code to new
            - test and prove it works
            - move changes to the stable tree.

An important note about the process is that the `experimental`/`unstable` tree must be kept in a good order at all times. As with utility code, without discipline, it's too easy to let these places become junk yards. Keep them clear from clutter by making sure everyone on the team is following conventions and the boy scout rule, move working code to the stable tree, eliminate unused and incorrect code.


## Monorepo: Additional Chapters

The approach to maintaining a monolithic repository outlined above is a good starting point, and works quite well in most cases. At the same time, larger organisations with a larger number of teams and services, may want or need more flexibility when working with a monorepo. Let's think of potential reasons for that, and how to address the needs.


### Open Questions

What questions are left with no answers by the structured monorepo? Some of them include:
- What if we need to maintain both, older and newer versions of a package?
For example, this could easily be the case when a large number of services work with the same model, say a core business unit. You have to update the model, you tried to find a way to make a backward and forward-compatible change, but it seems to be either impossible or too resource demanding. Had this model been in a separate module, a new major version with a breaking change could have done the job. But with a monorepo, everything is versioned as a whole, or not versioned at all.
- What if some packages are changed too often, such that it's hard for other teams to keep their services up to date with the changes? Or, how to allow quick and frequent changes in code that is relied upon by other code?
While this should not be normally the case in a well-designed system, this happens. There might be places in a codebase which are depended upon by hot execution paths, and for some reason changes to them are made quite often. If several teams are forced to perform routine updates caused by the work of another team, this can quickly become a major bottleneck, a source of anxiety and even lead to a (rather wrong) conclusion that monorepos don't work.
- What if infrastructure and utility code changes often enough to bother other teams with the need to keep their services up to date?
Similarly to the previous question, this normally should not happen. Yet, sometimes this happens, especially when the initially written low-level code is not optimal (someone quickly prototyped, and forgot to finalise the work by taking the draft to a solution that is done well), and at some point it needs to be refactored. In such case, there *will* be breaking changes, and having everything in a single repository dictates the need to adjust all depending services at the same time.
- What if all of the above is the case, the number of people working on a project is large, as well as the codebase, which is slightly aged and has got a fair amount of legacy in it?

These are just a few questions that you may have, either because it's your case, or you know someone who is in a similar situation, or for any other reason. There are, of course, many other potential questions. Our goal here is not to find a silver bullet that is the ultimate answer to these and all other questions. Instead, we're trying to find out something to start with, where you can begin experimenting and working out your own way.

Further we consider two additional options that you can employ to help in maintaining a healthy monolithic repository as well as to keep your teams happy and efficient, as much as it is possible in this context. It's advised to use them only when the structured approach is not enough, and it has either already limited you in some way, or the analysis shows some evidence that it might happen soon, and other changes have been tried to improve the situation.

As with everything in life, any flexibility comes at a cost of increased complexity and responsibility, and you have to keep the latter two in balance. Abusing the complexity, or neglecting the responsibilities are two of the most common reasons for any failure, especially and very often it is the case in software development.

Having put an extra stress on the importance of being cautious with added complexity, let's look at ways for achieving additional flexibility. The first one offers the use of nested Go modules, and the second is a bit more radical, but still an option for situations when changes happen too often, and are too breaking. And finally, we look at the combination of the two.

_NOTE: In the context of the next section we use the term "nested modules" for a Go module that is located not at the root of a repository, potentially having siblings in neighbour directories. We deliberately refrain from using the term "submodule" to avoid confusing it with a Git submodule. Git submodules are not considered in and are outside of the scope of the book._


### More Structure with Nested Modules

Consider converting the top-level packages into nested modules when need more fine grained control and flexibility over packages' lifecycle.

This is possible because Go modules support nested modules, which are sometimes referred to as submodules.

As you know, a Go module is identified by the `go.mod` file. Each module must have this file, and it's location defines the root of the module. While it is the most often use case, and it is strongly advised to have a single module per repository located at the root, it's possible to maintain multiple modules under the same repository. It's also possible to version nested modules independently using Git tags of a slightly different form.

In general, the technique works as follows:
- some of the top-level packages are separate modules
- each such package, i.e. module, is versioned separately
- each module can import another module, as if it was just a package
- following the conventions set by semantic versioning is mandatory, i.e. each breaking and incompatible change requires a new major version of such module
This allows the use of a separate import path by the code that is interested in the newest version, while the code that is yet to be updated continues using the older version.

Now let's look at an example. Here, we have a monolithic repository called `monorepo` owned by the `myorg` account. In the repo, we've got two packages - `model` and `controller`. Everything up until now has been as a single module, defined at the root of the repository. Our task is to convert the two packages into nested modules, and to version them independently. The execution plan consists of three steps, for each package:
- initialise a module
- commit and push changes to the remote origin
- version the new module by creating and pushing a tag, paying special attention to the name of the tag, it's `module_name/vX.Y.Z`, not just `vX.Y.X`.

The final directory structure looks like this:
```bash
.
├── controller
│   ├── controller.go
│   └── go.mod
├── model
│   ├── go.mod
│   └── model.go
└── go.mod
```

Let's go look at each step in detail for each of the two packages, starting with `controller`. For simplicity, we omit creating a separate branch and pull request:
- first, convert the `controller` package into module
```bash
cd controller
go mod init github.com/myorg/monorepo/controller
```

- now commit and push the change
```bash
git add go.mod
git commit -m "Convert controller package to module"
git push
```

- next, version the module by creating a tag
```bash
git tag -a controller/v0.1.0 -m "Convert controller to module"
git push --tags
# or
git push -u origin controller/v0.1.0
cd ..
```

Now we handle the `model` package, assuming it's version is somewhat bigger:
- convert it into module:
```bash
cd model
go mod init github.com/myorg/monorepo/model
```

- commit & push
```bash
git add go.mod
git commit -m "Convert model package to module"
git push
```

- finally, version the module
```bash
git tag -a model/v0.4.1 -m "Convert model to module"
git push --tags
# or
git push -u origin model/v0.4.1
cd ..
```

And that is it.

This approach allows for versioning packages within the monorepo independently. The biggest advantage is that you can now easily introduce a breaking change under a new major version, starting from `v2.0.0`. A detailed conversation about versioning Go modules is the subject for next section, right after we finish with monolithic repositories.

All above is, of course, good, but what if you're larger than Google, and one monorepo is too limiting for your needs, or for some other reason the options discussed so far offer not enough flexibility? Well, would two or more monolithic repos help you?


### Multiple Monorepos

In a perfect world, and for some teams, a single monorepo works absolutely well. But what if your reality is somewhat less than ideal, and one monolithic repository is not enough? If a single repository is too limiting or restrictive, that is probably not a sign that it does not work. Perhaps you just need opt for two or more monorepos, each carrying unique responsibilities.

We leave aside some obvious and legit cases where a large organisation has several monolithic repositories as a matter of separation of concerns. For example, one company acquires another, and the two companies under the same roof continue development in two monorepos. Legal concerns might be involved too, so this is an absolutely natural reason for multiple monorepos.

Similarly, we leave aside situations where a company starts experimenting in a new field, or a new project. For example, it's natural for a game studio to develop each game in a separate repository.

Nonetheless, there are questions that some may want to ask. What if, for example, the infrastructure-specific code is evolving too quickly? Or what if CI/CD workflows for client and service code are too different? What if services in the monorepo use so different technologies that it somehow resulted to incompatible workflows? Or, what if one team wants to follow GitHub Flow, and the other is willing to follow Git Flow? What if one team is simply not willing to adapt the practice?

The questions above, although sounding real, are a subject to the overall engineering policy and culture at an organisation. Normally, an organisation has some sort of shared vision where everyone is aligned with and agreed on the foundational principles, approaches and tooling. So if those sorts of questions are present within your organisation, and everyone is doing whatever they want, that's probably a topic for a separate conversation. Here we focus on the technical aspects of dealing with a problem where one monorepo is not enough. And while the author has failed to find an example of a real reason for this (besides a few mentioned earlier and the likes), it should not prevent us from exploring what's available.

Below we briefly explore several available combinations for consideration should you need this. It's obvious that all options that are mentioned, as well as anything that you come up with, have pros and cons, and here are a few common. Unless given explicitly, a common advantage is increased flexibility, either technical or organisational/legal, which comes at a cost of the downside in a form of exponentially increased complexity, and increased cost of maintenance. Instead of reducing entropy and chaos, they facilitate it.

_NOTE: Bear in mind that the options explored further won't work for all teams, and finding a silver bullet is not the goal of the exploration. It should be clear that no one is enforcing anything upon the reader. Some teams consist mostly of full stack developers, while others prefer specialised engineers. Therefore, it's obvious what is good for one case, might not work for another._


#### Multiple Single-Module Monorepos

A simplest option, if applying this adjective is even possible once we're here, is to have a few monorepos that follow the structured approach described in detail earlier in the unit.

It's relatively straightforward, and enables for easier separation of concerns. This option might suit for cases when code needs to be split for both, legal and technical reasons, when alternatives are somewhat less optimal.

It might also be helpful when there is a strong desire or need in splitting utility code from business logic code. In such situation, the monorepo with utility code becomes an official in-house extension to the standard library, and the monorepo with business logic and services is focused only on the company's business domain.


#### Architectural Monorepos

Another option is splitting all code into few layers, and put each layer into a separate monolithic repository.

A large organisation may have a huge project, or many projects, each of them have at the very least three major components: infrastructure, client-side and services. In such case, it might make sense to organise code as follows:
- all infrastructure and operations code lives in one monorepo
- the second monorepo is devoted to client code
    - it's also possible to go further and split web clients from mobile, if needed
- finally, the code that is responsible for handling and persisting data, i.e. services, goes into its own repository.

With this setup, all major components are split by their nature, and this might be beneficial for the teams, and processes.


#### Technology-Specific Monorepos

An organisation may have hundreds of thousands lines of code written using one technology stack, i.e. language, and then switch to another, producing another hundreds of thousands of lines of code, or millions. And while this is not necessarily a goal on its own, it might be beneficial to split code into monolithic repositories focused on a specific technology.

This option may work quite well when a company is transitioning from one stack to another, or works with several in parallel. Go code stands out in that it's much easier to support compared to other languages, besides maybe a case when you have no code at all. So it might be reasonable to put all Go code in a separate repository, and say C++ code in another, and then Javascript in the third.


#### Multiple Monorepos with Nested Modules

Potentially the highest degree of flexibility, along with complexity and chaos, can be achieved by employing multiple monolithic repositories with multiple independently versioned modules in them.

It's strongly advised against using this option, but it's still available. If your organisation is ready to pay the price of this flexibility, or if there is a real need in it (which is highly unlikely), then this is technically possible, and, with very high levels of personal responsibility of each individual and collective responsibility of the engineering department as a whole, might work. Depends on you.


## Closing Notes

Maintaining a monolithic repository that several teams are working with, is not an easy task. The experience shows that this is not only possible, but is also highly beneficial when approached consciously and organised, when the vision is shared by everyone involved in the process, when the culture is strong, and people understand the importance of keeping things in a good order.

The techniques described in this section are not new, unique, silver bullet, nor a panacea. But they have proved to be helpful for projects of various sizes, in small and larger teams. Give it a go, experiment, adjust to your needs, and find what works and what does not.


---

To be continued...
