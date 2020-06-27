---
layout: post
title: About Software Development with Golang
category: Programming
tags: [golang]
---


> To unleash your full potential you will have to do much of the study and research on your own. It will be... perilous.
>
> – Darth Bane

This post gives some detial about my experiences with Golang, and announces a project I've been working on recently.

<!--more-->

![](/assets/books.jpg)


## Where Did It All Start

At some point in the past I worked on a monitoring solution for infrastructure and applications. Think of it as an alternative to Datadog.

The initial version was implemented as a set of plugins and tools that we built around [Sensu](https://sensu.io/). At that time the Sensu company didn't even exist, and the project was based on Ruby and bash. Our project also was a mixture of some Ruby, Nodejs and bash. It was enough to build a proof of concept.

As our project evolved, we migrated everything to Python, mostly because it _seemed_ easier to distribute, retrieve the information about the underlying systems, and so forth. It also seemed to be a good choice since the vast majority of system tools uses it, it's shipped by default in most Linux distributions, and is familiar to sysadmins and developers.

While the solution worked at the beginning, we started facing new challenges. One of them was poor performance of the backend services. A SaaS-fashioned monitoring solution has to handle lots of data, and one of the goals with infrastructure monitoring is ensuring data is of a high precision, and is available as soon as possible for further processing and the users. It became apparent that this can hardly be achieved. Given quite a large number of solutions that had existed that time, the chances of offering great experience to the user were negative.

Another challenge was distirbution of the agent. Supporting even a small subset of the most popular Linux distributions requires maintaining and _properly packaging_ software in tens of different combinations. Several Ubuntus, then a couple of Debians, add some CentOSes, and we've got half a dozen of different versions of python, not to mention the availability of dependencies, dealing with init systems and so on.

Development and testing process are also important. It was clear that supporting such a project will quickly become a nightmare, so I needed to find a better solution to address the these and some other issues.


## First Look at Golang

After researching potential solutions to the problems I decided to try Golang. Thanks to the architecture of the agent, I could easily use the language to implement a small piece of functionality - a plugin, for example, for obtaining statistics from an external service.

Implementing the plugin took less than a day, and pretty soon a fully functional version was in use. The experience suprised me. Having almost zero knowledge about the language, I was able to make the solution work, and it proven to be a stable and performant solution.

That experience convinced me in two things. The first is that it's worth investing in mastering Golang, as it's an amazing tool for developing performant and resilient solutions. Much more convenient, peformant and stable than with other tools and languages. The second is that migrating the project to Golang would solve the existing and prevent from having many potential problems.

So I commited to learning Golang in a way that I find meaniningful. In short, this means that you get a firm foundation by:
- thoroughly studying high-quality materials (i.e. good books)
- practise as you learn
- read tons of code on the Internet
- search for good examples
- search for bad examples
- write tons of code
- and lastly, you never stop learning once started.

What is the most important when it comes to learning? Getting high-quality materials. At the beginning, there was no good materials at all. Then books started being published, and these days the amount is slightly overwhelming. Same was with articles on the Internet. There is a lot of noise, and you have to find the signal.

The vast majority of the materials fall into two categories - intoductions to the language, and howtos that teach you not necessarily optimal ways of writing software in general and using the language in particular.


## Finding The Signal

So I kept searching, learning and working my ass off to build a solid understanding and skill. When you've got such a goal, self-discipline is required. You read about something for the second, third or even fourth time in hopes to learn something new about it. You write thousands of lines of code just to throw it away a couple of months later, and rewrite again, developing a better understanding of the language.

Fost-forward to these days. As you remember, once started, never stop learning. By the end of 2019th, I've read from cover to cover 8 books and over 200 articles about Golang. In addition to that, I've read tens of thousands of lines of code, and seen the beauty and the ugliness of using the language. Several tens of thousands of code were written in addition to what I do at work, and mostly throwed away as drafts and sketches.

One conclusion that I ended up with is that all those materials don't give a clean understanding of how to develop maintainable and efficient applications and services using Golang. This is especially true for a reader who is searching for a guidence rather than explorations. When someone needs a practical advice on how to actually do the work, there _is_ no immediate answer. In order for one to form a view and good taste, they would need to basically go through a similar path as I've went. But that's a very long and resource demanding path, and not everyone is ready or able to invest resources into this process.

Another problem is the quality of materials, examples, and projects in the wild. Oftentimes you're taught or shown sub-optimal ways of using the language. Many, too many projects are written recklessly, so you don't really want to use that software. Many libraries are selfish, and let down your application quite easily without giving you a way to adjust behaviour. So learning by example is, in fact, oftentimes harmful, as there are too many examples of how it's better not to write software.

This is something that concerns me. It doesn't have to be like this.


## Introducing Golocron

It all starts with a name.

The name `Golocron` consists of two parts, one obviously being `Go`. The other though has nothing to do with task scheduling via `cron`. The second part comes from the word `Holocron`, which is the name of a small [device](https://starwars.fandom.com/wiki/Holocron/Legends) from the Star Wars Universe.

By combining the two words together, we get the folloing meaning: something that  contains knowledge about different ways of using Golang, and is available for those who searched for it. Now, add a bit of detail to make it self-explaining, and we get "Golocron - Software Development With Golang".

But what does this name indentify? What is that "something"? Well, it's a book.

I'm writing a book on using Golang to develop applications and services that are:
- simple (but not primitive or rigid)
- pleasure to work with
- easy to maintain
- easy to understand
- and most importantly, ready for change.

The book won't teach you the language. There are a few books about the language, and if you haven't read a couple yet, it's the right time. These two are good:
- [Programming in Go](https://www.amazon.com/Programming-Go-Creating-Applications-Developers/dp/0321774639) by **Mark Summerfield**
- [The Go Programming Language](https://www.amazon.com/Programming-Language-Addison-Wesley-Professional-Computing/dp/0134190440) by **Alan A. A. Donovan, Brian W. Kernighan**

So instead of writing another book about the language, I assume that the reader has some experience with Golang _and_ in software engineering. So what is this book going to be about?


## The Signal

The book will contain at least three modules; each module has several units; each unit is compraised of multiple sections. Modules group larger topics, units are focusing on specifics of a topic, and sections give details.

The main goal and material of the book is the third module, and its draft name is "The Clean Approach". This module is dedicated to application design and architecture, software engineering practices and guidelines on how to write maintainable and changeable software. This module covers:
- desiging for CHANGE
- the abstaction principle
- keeping responsibilities clear
- the need of encasulation of what varies
- the service structure, including handlers, controllers, repositories, models, and many more
- testing.

But in order to get there we need some preparations. The first two modules will prepare us by introducing some important principles and establishing conventions.

The first module is called, and in fact it is, The Style Guide. While there are a couple of style guides, my experience shows that there is still a big room for improvement. The Style Guide covers the following things:
- the project layout
- the package layout
- the file layout
- tips on writing good code
- and the actual guidelines.

The second module is called Gound Work, and it tells you about writing good, correct and efficient underlying code for your future service. This includes, but not limited to:
- process management and life cycle of a service
- working with web and explains how to stay lean, and why you don't need a framework
- working with RPC, and why you don't need bloated libraries such as gRPC
- writing middlewares
- working with data storages
- the error flow.

Additionally, the book comes with a set of fully functioning services showing the ideas and guidelines live.

Of course, the actual contents may change as I progress, but these are the core things that the book will exist for.


## The Process

I've never done anything like this before, and haven't taken a course on writing books. So the actual process is going to be discovered. It's going to be a journey with ups and downs rather than a highly structured and strict process.

At the same time, having no plan is equal to having a plan to fail, and the latter isn't something I'm aiming for, even though ready to. Here is what the plan looks like at a high level:
- write a draft of The Style Guide
- in the background, continue gathering the information for the third module, The Clean Approach
- write a draft of the Ground Work module
- draft The Clean Approach
- compile everything together
- iteratively improve the draft
- officially release the first version.

This is going to take some time, in fact, a lot of time as it's a long process. To keep you interested and updated, and to give you something what you can already start using and hopefully benefiting from, I will publish drafts as they're ready as articles in this blog. As soon as a new section is ready, it will appear here.

Last 12 months I've been working on the code materials for the book. While a lot has been already done, there is still some work left to do. The materials will be open-sourced when it's time, most likely, when the most of the book is ready.

The book itself is already in progress. I've been working on The Style Guide for several weeks already. Some of the materials are ready to meet its first readers.

Once ready, the book will be published as an open-source book.

I wish the process was moving faster, and am doing everything to keep it going. I work on the book only in my spare time. Usually, during a week I work on the code materials, and write the actual content on weekends. Given the current speed, hope to finish the draft for the first module in next 3 - 4 months. Then there will be some work on the pre-draft metrials for the third module.

The current and most likely a very wrong estimation is that it will take about 1.5 years to get it to its finish. So, let's see.


## Conclusion

It's inspiring and frightening at the same time. It's scary to announce something publically. It's double scary because I'm not that fluent in English as I wanted to be. I don't know what it will wind up with, and also not so sure if I'm able to get it to its end.

What I do know though is that I will do my best. And my hopes are this work helps someone to get efficient and productive in writing software in general, and with using Golang, and they find it helpful.

Stay tuned in, and see you next time!
