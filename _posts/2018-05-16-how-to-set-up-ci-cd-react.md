---
layout: post
title: "How to set up continuous integration and deployment for your React app"
description: "Setting up a React development environment can be confusing and daunting to newcomers."
date: 2018-05-16 16:39:18
comments: true
keywords: "ci, cd, react"
category: ci
tags:
- ci
- cd
- react
---

<img src="{{ '/images/posts/github-check.png' | prepend: site.baseurl }}" alt="Github Status Check">

Setting up a **React** development environment can be confusing and daunting to newcomers. You might have heard developers talking about how different packages like babel, **Webpack** and so on, are needed as well (but this is debatable).

With React getting more popular, there are a few boilerplate projects that aim to help developers create a suitable React development environment. [create-react-app](https://github.com/facebook/create-react-app/) is one of the most popular starter templates.

It aims to allow developers to create a react app with zero build configuration.

Developers no longer have to worry about how `webpack` should be setup, what should be configured with `babel` to use `es6`, or which `linter` and `test` package to use. Everything will just work out of the box. **Yes, it is so easy!**

For developers who need to manage the underlying configuration, it has a `npm run eject` that allows them to mess with the configuration and do what they couldn't do previously. The only thing to note is that once `eject` is run, it cannot be reversed.

## Development Stack for React

I wrote the following guide to help developers build a **Continuous Integration and Continuous Deployment stack for their React app**. We will be using [CircleCI](https://circleci.com), [CodeClimate](http://codeclimate.com), and [Heroku](https://heroku.com). If you do not have an account at any of the services above, head over to sign up - they're FREE!

At the end, we will have a React app in a [Github Repo](https://github.com/Zaccc123/awesome-cicd-react) that will automatically deploy any changes on `master` branch to [Heroku](https://heroku.com) after all tests pass. Here is a sample of the deployed [React website](https://awesome-cicd-react.herokuapp.com/).

## Let's Start!

The first step is to follow the [create-react-app](https://github.com/facebookincubator/create-react-app) guide to create a new React app. Do this:

```bash
$ npm install -g create-react-app
$ create-react-app my-react-app
$ cd my-react-app/
$ npm start
```

Then the browser should automatically open a page at http://localhost:3000/. If you see a Welcome to React page running, everything is good.

## CircleCI Setup

Next, we need to add a little configuration to setup [CircleCI](https://circleci.com) for our project. Create a `.circleci` folder and a `config.yml` in that directory and add the following:

```yml
version: 2
jobs:
  build:
    docker:
      - image: circleci/node:6
    steps:
      - checkout
      - restore_cache: # special step to restore the dependency cache
          key: dependency-cache-{{ checksum "package.json" }}
      - run:
          name: Setup Dependencies
          command: npm install
      - run:
          name: Setup Code Climate test-reporter
          command: |
            curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ./cc-test-reporter
            chmod +x ./cc-test-reporter
      - save_cache: # special step to save the dependency cache
          key: dependency-cache-{{ checksum "package.json" }}
          paths:
            - ./node_modules
      - run: # run tests
          name: Run Test and Coverage
          command: |
            ./cc-test-reporter before-build
            npm test -- --coverage
            ./cc-test-reporter after-build --exit-code $?
```

This setup is for [CircleCI 2.0](https://circleci.com). They are sunsetting [CircleCI 1.0](https://circleci.com) on August 31, 2018.

The build step sets up a node:6 with a Docker image. It requires v6 or higher to work.

In steps, we first check out the project, restore from the cache if any, then install dependencies. We also install cc-test-reporter, a tool provided by [CodeClimate](http://codeclimate.com) to send a coverage report.

We then run the test between the before-build and after-build commands according to [CodeClimate docs](https://docs.codeclimate.com/docs/configuring-test-coverage). This notifies [CodeClimate](http://codeclimate.com) of the pending report and when completed, it either sends the report or a failure status.

## Setup Git

Create a repo in [Github](https://github.com) and do the following:

```bash
$ git init
$ git remote add origin git@github.com:username/new-repo-here
$ git add .
$ git commit -m "first commit"
$ git push -u origin master
```

This will push the project that we've created into GitHub.

## Build and Test the Project

Head over to [CircleCI](https://circleci.com), sign in, and build the newly created project. At the end of the build, you should see a failure on the `Run Test and Coverage`.

<img src="{{ '/images/posts/circleci.png' | prepend: site.baseurl }}" alt="CircleCI Result">

## Setup CodeClimate

The above failure is because we are not authorized to post a report to [CodeClimate](http://codeclimate.com) yet. So, now, head over to [CodeClimate](http://codeclimate.com), sign in and build the created GitHub project. We should get this at the end of the analysis:

<img src="{{ '/images/posts/code-climate.png' | prepend: site.baseurl }}" alt="codeclimate analyse">

In order to fix the [CircleCI](https://circleci.com) issue and use `Test Coverage` feedback, we will need the Test Reporter ID. This can be retrieved at the Settings > Test Coverage tab. Copy the Test Reporter ID without sharing it with anyone.

In [CircleCI](https://circleci.com), navigate to `Project > Settings > Environment variable` and add `CC_TEST_REPORTER_ID` with the copied `Test Reporter ID`.

<img src="{{ '/images/posts/circleci-env.png' | prepend: site.baseurl }}" alt="circleci environment">

## Heroku Deployment Setup

In order to deploy React on [Heroku](https://heroku.com) , we will use a [buildpack](https://github.com/mars/create-react-app-buildpack). Do the following:

```bash
$ heroku create REPLACE_APP_NAME_HERE - buildpack https://github.com/mars/create-react-app-buildpack.git
$ git push heroku master
$ heroku open
```

We pushed the latest `master` branch to `heroku` with `git push heroku master`. A webpage will be open at the end showing the **Welcome to React** page.

Next, we will have to navigate to the newly create app in [Heroku](https://heroku.com) Dashboard to setup automated deployment. Do the following on the dashboard:

* Go to Deploy tab and Connect to the correct GitHub repo.
* Enable Automatic deployment and check Wait for CI to pass before deploy.

<img src="{{ '/images/posts/heroku-cd.png' | prepend: site.baseurl }}" alt="enable automatic deployment">

## We are done!

With a few steps, we have a fully automated continuous integration and deployment suite ready. Now with every commit that is pushed to GitHub, it will send a trigger to [CircleCI](https://circleci.com) and [CodeClimate](http://codeclimate.com). Once the test has passed, if it was on the master branch, it will also be automatically deployed to [Heroku](https://heroku.com).
View the sample repo [here](https://github.com/Zaccc123/awesome-cicd-react) and the deployed website [here](https://awesome-cicd-react.herokuapp.com/)!

## Conclusion

This is an update of my previous [post](https://medium.com/@Zaccc123/https-medium-com-zaccc123-continuous-integration-and-deployment-setup-for-react-app-7b5f4bd76cdd) almost a year ago. The use of [CircleCI](https://circleci.com) has been updated to `2.0`, and we also use the updated `cc-test-reporter` by [CodeClimate](http://codeclimate.com). If you are interested in the migration, you can look at the [pull request](https://github.com/Zaccc123/awesome-cicd-react/pull/3).

---

## Thanks for reading!

I enjoy reading and writing about tech and products especially related to boosting the productivity of developers. You can say hello to me on my [Twitter](https://twitter.com/Zaccc123).
