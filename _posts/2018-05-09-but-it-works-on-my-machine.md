---
layout: post
title: "But, it works on my machine."
description: "With the current trend and emerging technology, there are many ways to prevent this from happening."
date: 2018-05-09 16:39:18
comments: true
keywords: "ci, docker, agile"
category: ci
tags:
- ci
- docker
- agile
---

<img src="{{ '/images/posts/it-work-on-my-machine.png' | prepend: site.baseurl }}" alt="But, it work on my machine">

This is a big problem if you or anyone in your team still said it. With the current trend and emerging technology, there are many ways to prevent this from happening.

No matter how good a developer is, he or she will still make mistake. There are many common reasons why a particular change work on a developer machine but fail on others. Some common reasons are:

* Different machine and system environment
* Missing dependencies
* Hardcoded directory or path in the code

## Working Together and Shipping Early

With the shift in methodology towards agile development, cross-functional team work together closely and ship/deploy software early.

Instead of product team vs infrastructure team, when the cross-functional team works together it is easier to spot the problem and fix the issue. People also tend to get less defensive when anyone on the team is allowed to just jump in and fix any issue.
When we deploy our changes early and frequently, we can avoid deployment hell. Any failure could be spotted immediately and apply fix before it gets worst.

## Continuous Integration

The day where you need to mess with tedious and complicated Jenkins set up to run your own CI Server is long gone. With cloud base CI company, like [CircleCI](https://circleci.com) and [TravisCI](https://travis-ci.org) anyone can setup continuous integration with just a simple yaml config file.

[CircleCI](https://circleci.com) - provide a free instance for both private and public repo. It supports many languages and is used by large companies like Facebook, Spotify, and Kickstarter.

[TravisCI](https://travis-ci.org) - provide a free plan for public repo and required a paid plan for private projects. It is extremely popular among open source community.

By having a separate machine to run the changes in our codebase, we can find out problem early. A red flag on a separate test machine is always less offensive than being challenged by someone else. Even without tests, we should still ensure the project work as it is and compile with all the required dependencies.

## Docker Container

We can easily address the issue of different machine and system environment, by setting up our development project with [Docker](https://www.docker.com).
A really good article that compares the benefit of development with and without docker can be read here.

In short, any new developer who join the team can easily start working on the project just by having Docker installed in their machine. The onboarding time is lesser, and we no longer have to worry about different OS, system version or package version because everything should be specified in the `docker-compose.yml` file.

## In Conclusion

With emerging technology and trend in development, there is no excuse for any team to still have the developer who said that "it works on machine". If it didn't work on the CI machine, you did not complete your job. The priority is to fix it before moving to other issues.

If you are interested, in a separate [article](https://medium.freecodecamp.org/how-to-set-up-continuous-integration-and-deployment-for-your-react-app-d09ae4525250), I setup a [demo react project](https://github.com/) that has a continuous integration and deployment setup with [CircleCI](https://circleci.com), [CodeClimate](http://codeclimate.com) and [Heroku](https://heroku.com).

---

## Thanks for reading!

I enjoy reading and writing about tech and product especially related to boosting the productivity of developers. You can say hello to me on my [Twitter](https://twitter.com/Zaccc123).
