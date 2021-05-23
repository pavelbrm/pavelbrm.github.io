---
layout: post
title: "The Style Guide: Package Layout 7 – Summary"
category: Golocron
tags: [golang, m1u2]
---

> It takes something more than intelligence to act intelligently.
>
> – Fyodor Dostoyevsky

This post concludes the second unit of the book. Inside, you'll find a complete list of suggestions about good package layout. Come on in.


<!--more-->

![](/assets/m1u2_7.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Summary

This unit went one layer deeper in the project structure, and discussed guidelines for package design. Its leitmotif is similar to the one of the project layout – mindful, thoughtful and responsible choices do a better job short and long-term, for your peers and you.

Here is a quick recap of the suggestions in this unit:
1. Create a package to improve the design of software, not for the sake of having a package.
2. The `main` package is special, and must be kept as minimal as possible.
3. When creating a new package, make sure to:
    - keep the public API as narrow as possible
    - provide a solution for a task
    - choose a good and meaningful name which
        - sets the right context and namespace
        - is consistent with the environment, and is not confusing
        - is short and concise singular noun
        - is precise and focused, i.e. avoid `common`, `util`, `tools`, etc
        - and is unique.
4. Organise a package properly, in accordance with Go conventions:
    - prefer a flat list of packages by default
    - there is no such thing as a sub-package in Go, organisationally speaking.
5. Keep files in a package in a good order:
    - begin with a single file named as the package
    - organise files by topic, responsibility and behaviour
    - name other files meaningfully
    - avoid creating file per object
    - prefer fewer files of larger size
    - keep methods of a type in the same file
    - avoid ambiguous file names
    - use a separate file for documentation when appropriate.
6. To minimise or reduce the maintenance burden when working with cross-platform code:
    - follow the basic principles of writing cross-platform code
        - the business logic must be platform-agnostic
        - the amount of cross-platform is as minimal as possible
        - significant differences between platforms are compensated by minimal mocking or no-op implementations
    - know the tools available for cross-platform development in Go
        - ways to produce binaries
        - approaches to differentiating code between supported platforms
            - using file names
            - or build tags
            - and how to combine them when needed
    - organise packages containing cross-platform code with change in mind, i.e.:
        - keep platform-dependent code at minimum
        - use simple branching in trivial cases
        - have no platform-specific code in `main`
        - use file suffix in simple cases
        - use build tags when more flexibility is needed
        - strive for platform-independent tests
        - and provide cross-platform tests only when you must.

These guidelines are the continuation of the project level recommendations at a higher resolution. Applied together and in a consistent manner, this sets up a solid foundation for a codebase. Even if you inherited a disorganised codebase, it would be possible to improve its structure by taking small steps towards the better shape. As long as you keep improving it, and continue making right design choices, the codebase eventually reaches, or remains in, a good shape. And then you have to continue, as nothing is static, and is always evolving.

The next step will take us one more layer further – the level of a file in a package. We've made the necessary preparations, and are ready for discussing code organisation within a file.

---

To be continued...
