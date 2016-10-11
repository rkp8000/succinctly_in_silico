---
layout: post
title: "the scientific stack"
description: "enumeration of the minimum computational tools that a scientist should have"
comments: true
---

One of the key factors in developing a smooth scientific workflow is asking what software is most worthwhile to have around when writing scientific code. I've spent a fair bit of time thinking about this lately, and in this post I’ll share some of my conclusions.

Before I go into the details, though, I’d like to remark that technically you only need to know the basic syntax for a single programming language to build anything you want, since all code, no matter how complex, is fundamentally composed of a large number of simple components. This reductionist take on writing code was my perspective early on and in my experience embodies that of many beginning scientific coders. After following this principle for some time, however, I found that the codebase I was developing inexorably waxed a more and more rickety construction. Making small changes to one part made seemingly unrelated parts completely fail, debugging became a nightmare that could ruin several days of work, and most importantly, even when my code didn’t crash I found that I simply didn't trust the results.

Fortunately for those of us wishing to avoid the danger of an unsound codebase, there’s a ton of freely available code out there that’s already been written for you and much of which claims to be very useful should you take the time to learn how to use it. However, if you’re like me you probably find it hard to decide when taking the time to sit down and learn to use a new piece of software will actually be beneficial to your research in the long run. Further, even if you do decide to put in the effort, only some tools will actually prove their worth; many others might take a long time to learn but still end up finding little application to your project.

To help others expedite this process, below I’ve collected together a list of the core tools that I think are truly more than worth it to learn how to incorporate into your workflow. That is, it’s my belief that each item in this so-called *scientific stack* (in allusion to the more commercially recognized *web stack*) will not only genuinely save you time in the long run (that is, the upfront learning cost is worth it purely in terms of total hours spent coding) but will also help make your analyses cleaner, more robust and testable, and ultimately more reproducible and extendable.

In addition to listing core technologies that I think are worthwhile having, I’ve also included my recommendation for a specific package or library for each, making sure to include only freely available software. Though the list may seem a bit daunting at first, a lot of the first steps are already done for you, since many of the tools come bundled together. 

| Core technology | Recommended software | Description |
|:--------------- |:----------------:| -----------:|
| basic programming language | Python (specifically the [Anaconda][anaconda] installation) | where you do the vast majority of your programming |
|----
| version control system | [git][git] (included with Mac OSX and Linux) | software that tracks the changes you make to your code base |
|----
| service for backing up and sharing code | [GitHub][github] | important components of the scientific process |
|----
| programming sandbox | [Jupyter Notebooks][jupyter] (included with Anaconda) | where you "mess around" when learning how to use new libraries, etc. |
|----
| IDE | [PyCharm][pycharm] | the "integrated development environment", a powerful tool built to accelerate the actual writing of production code |
|----
| code testing library | [pytest][pytest] (included with Anaconda) | framework for automatically testing custom-written functions to ensure they work properly and don't change their behavior when you change their implementation |
|----
| numerical library | numpy (included with Anaconda) | routines for array manipulation and standard advanced mathematical operations | 
|----
| statistics library | scipy.stats (included with Anaconda) | routines for performing standard statistical tests |
|----
| data analysis library | pandas (included with Anaconda) | basic routines for manipulating data and summarizing it in useful ways |
|----
| visualization library | matplotlib (included with Anaconda), seaborn, bokeh | routines for quickly displaying nice-looking plots |
|----
| package manager | conda (included with Anaconda), pip | software allowing you to download and install new packages |
|----
||||
{: rules="groups"}

[anaconda]: https://www.continuum.io/why-anaconda
[git]: https://git-scm.com
[github]: https://github.com
[jupyter]: http://jupyter.org
[pycharm]: https://www.jetbrains.com/pycharm/
[pytest]: http://doc.pytest.org/en/latest/ 

And if you're working with reasonably sized datasets:

| Core technology | Recommended software | Description |
|:----------- |:-----------:| ---------:|
| database management system | [postgreSQL][postgres] | software for organizing, storing, and querying data |
|----
| database toolkit | SQLalchemy (included with Anaconda) | routines for interacting with database using primary programming language |
|----
| database GUI | [PgAdmin][pgadmin], [phpPgadmin][phppgadmin] | software for graphically browsing your data | 
|----
||||
{: rules="groups"}

[postgres]: http://www.postgresql.org
[pgadmin]: https://www.pgadmin.org
[phppgadmin]: http://phppgadmin.sourceforge.net/doku.php

Here I’ve only included tools that I think every scientific coder will benefit from, but of course there are many other extremely useful packages for specialized purposes (such as symbolic mathematics, network analysis, or efficient parallel computing, to name a few). Many of these I’m not very familiar with, but I’d love to hear your thoughts and recommendations if you have any. In future posts I’ll go into more detail about how to make the most of some of the specific packages I’ve listed above.
