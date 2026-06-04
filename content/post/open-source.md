---
author: cunhazera
title: "Giving back: relevant open source contributions and where to start"
date: 2026-06-04
description: A starting point for open source contribution
---






You probably already noticed it, but most libraries and tools that you use to build your applications are open source. Which means that someone started a project and decided to make it collaborative, receiving feedback from others, allowing them to take part in the project's evolution through ideas and code. And most people don't make their living with open source as a full-time job, so committers are very likely using their free time to help shape a library they use.
That being said, a good portion of your applications points to someone else's work, someone who was trying to fix something for themselves or for the community, even when they didn't have to do so.

There's a quote by Yegor Bugayenko that always stood out to me:
> They give you something, and you’re not giving anything back.¹

This makes perfect sense and gives a good reason to consider contributing: giving back. You've been benefiting from open source more than you know. From your IDE to the database you run in production, there's a lot of effort to build useful and stable tools other devs can use. Getting involved with projects is an opportunity to actively help the community and somehow return the favor from those who dedicated their time and careers to help people they never met. Especially when you realize that besides the funny part of solving real problems, contributors also need to deal with hundreds of feature requests of all sorts of "here's what you should do next" and "I think this is a good idea, just because". Hands-on contributors who are willing to roll up their sleeves always take over those who just make requests but never take part in discussions, reviews or even coding itself.

But I know how it feels: you want to contribute, you really want to, but you don't feel like you have enough technical knowledge to work on the issues on Github, maybe you can't even submit one of your own. And your first question is probably: how do I get started? What is the first step?

I've seen many developers who actively maintain Java libraries on GitHub talking about how to get involved with open source projects lately. And the truth is: there's no easy way. There's no step-by-step guide that will teach you how to make impact in some project. A few days ago, I saw a message from [Rodolfo Mendes](https://www.linkedin.com/in/rodolfo-mendes/), sharing a question on [reddit](https://www.reddit.com/r/SpringAIDev/comments/1tgw2y4/). Someone asked the same question, trying to understand what to do in order to contribute in Java AI ecosystem. Here's part of my response:
> [it's hard to contribute to non-existent problems (it's a clickable link)](https://www.reddit.com/r/SpringAIDev/comments/1tgw2y4/comment/oo0wyac/)

Contributions don't happen in a vacuum, with people who stomp on a repository and start coding. They require real engagement, empirical experience, and a real problem to be solved.

I say that from my own experience. I used to see open source contribution as some abstract thing, as if I would be able to simply go into a project and code some random solution in a codebase I've never seen before. And that's just not how it works (not for most contributions, I think).

When I look back at all my GitHub issues, all of them were related to something I was using heavily. It started small: I was building an SDK that parsed Java files and kept running into edge cases, so I ended up opening a few issues at the [JavaParser](https://github.com/javaparser/javaparser) repo. Some of them were just questions, others became features or bug fixes.

From there, it became a pattern. I started testing Quarkus in 2019, the year it was released, and found a particular [database operation](https://github.com/quarkusio/quarkus/issues/7884) I wished was part of the Panache spec. After lots of back and forth, it was eventually reopened after someone came up with a convincing argument. Later, during a talk about Jakarta Data, I mentioned to the audience that you could not use type-safe ordering for projections — and that observation became a [new issue](https://github.com/jakartaee/data/issues/1197) in the repo, which ended up being accepted.

What all these cases have in common? All of them started from a personal experience while I was actually using these tools. If your motivation in getting started with open source is just to say you committed to a hyped project, you will very likely end up going from project to project without doing anything useful to the world. As I said before: it's all about giving back. If you're not interested in improving a project, or if you just want to increase your market value by creating pointless issues, please save yourself (and the project maintainers) some time. These libraries impact millions of applications and developers in the industry, and repository owners are not in the mood to _mess around and find out_ with your selfish "contributions".

# My current contribution: Open J Proxy (OJP)

That same principle is what led me to [Open J Proxy](https://github.com/Open-J-Proxy/ojp) — a database-level library that provides backpressure, observability, load balancing, and more performance-related resources so you can better handle database connections. I decided to get involved for two main reasons:
 - I truly believe in this project: scaling your database and handling several concurrent connections is hard, especially when you have a distributed monolith with different services that share the same database²
 - Friendly and engaged community: the project leaders welcome everyone who's willing to invest some time to help with the existing tickets. From refactoring to expanding the core functionalities, there's space to learn, discuss, and push code into the repo.

And when I say that you're welcome, I really mean it! Take a look at the [project issues](https://github.com/Open-J-Proxy/ojp/issues): some of them are labeled as "good first issue". That's probably what you want to work on before moving to greater things. If you're just getting started there's something we need: documentation!

Documenting things is valuable for new users and also new contributors during their first use-case with the project. There was a [PR merged recently](https://github.com/Open-J-Proxy/ojp/pull/542) that improved our e-book diagrams. None of the contributors thought of it, but someone picked up and created a PR after identifying some area that needed improvement.

Bottom line, if you believe in what we're doing with OJP, feel free to request your new features or bug fixes! There's an ebook in progress, distributed transaction support, logging, and lots of other things that make this project unique! That's an exciting tool, with a net of engineers that are willing to deal with your questions with respect and transparency.

Open source is not easy, but it might be fun and a good way to learn, help core committers, and eventually build a network of engineers interested in the same things you are.
<br>

<br>

<br>

¹ https://www.yegor256.com/2015/12/22/why-dont-you-contribute-to-open-source.html

² I'm not saying it's good, I'm simply stating a fact
