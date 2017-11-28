---
layout: post
title: "Continuous Integration and Deployment setup for React App"
description: "Setup a CI and CD pipeline for React App"
date: 2017-05-28 16:39:18
comments: true
keywords: "reactjs, react, ci, cd"
category: reactjs
tags:
- reactjs
- circleci
- cd
---

Setting up a `React` development environment can be confusing and daunting to newcomers. You often heard developers saying different packages like `babel`, `webpack`, `es6` and etc. are needed (not entirely true) as well. With `React` getting more popular, there are a fews boilerplate project aim to help developer create a suitable `React` development environment.

[create-react-app](https://github.com/facebookincubator/create-react-app) is one of the most popular starter template. The aims is to allow developers to create `React` app with zero build configuration. Developers no longer have to worry about how should `webpack` be setup, what should be configured with `babel` to use `es6`, which `linter` and `test` package to use. Everything will just work out of the box. **Yes**, is so easy!

For developers who needed to touch the underlying configuration, it have a `npm run eject` that will allow developers to mess with the configuration and do what couldn’t be done previously. Only thing to note is, once `eject` is run, it cannot be reversed.

## Development Stack for `React`
The following guide aim to help developers build a **Continuous Integration and Continuous Deployment** stack for `React` app. We will be using [CircleCI](https://circleci.com), [CodeClimate](https://codeclimate.com) and [Heroku](https://heroku.com). If you do not have a account at any of the service above, head over to sign up one, is FREE!.

### Sample Project
At the end, we would have a React app in [Github Repo](https://github.com/Zaccc123/awesome-cicd-react) that will automatically deploy any changes on `master` branch to [Heroku](https://heroku.com) after all tests passes. [Here](https://awesome-cicd-react.herokuapp.com) is a sample of the deployed `React` website. Visual feedback will also provide on `pull request` like this:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/12/pull-request.png" width="800">


### Lets Start!
First step is to follows the [create-react-app](https://github.com/facebookincubator/create-react-app) guide to create a new app. Do this:

```bash
$ npm install -g create-react-app

$ create-react-app my-react-app
$ cd my-react-app/
$ npm start
```

Then browser should automatically open a page at [http://localhost:3000/](http://localhost:3000/). If you see a **Welcome to React** page running, everything is good.

### CircleCI Setup
Next, we need to add a little configuration to setup [CircleCI](https://circleci.com) for our project. Create a `circle.yml` in the root directory and add the following:

```yaml
machine:
  node:
    version: 6.0.0
test:
  override:
    - npm test -- --coverage
  post:
    - npm install -g codeclimate-test-reporter
    - codeclimate-test-reporter < coverage/lcov.info
```

We have to use `node v6` because [create-react-app](https://github.com/facebookincubator/create-react-app) only works with `v6` and above. We override the test to run the option with `npm test — —coverage` to gather coverage report. At the end, we install `codeclimate-test-reporter` to send the coverage report located at `coverage/lcov.info` to [CodeClimate](https://codeclimate.com).

### Setup Git
Create a repo in [Github](https://github.com/) and do the following:

```bash
$ git init
$ git remote add origin git@github.com:username/new-repo-here
$ git add .
$ git commit -m "first commit"
$ git push -u origin master
```
This would push the project that we had created into the github.

### Build and Test the Project
Head over to [CircleCI](https://circleci.com), sign in and build the newly created project. At the end of the build, you should see a all green like this:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/12/ci.png" width="800">

### Setup CodeClimate
Now, head over to [CodeClimate](https://codeclimate.com), sign in and build the created github project. We should get this at the end of analyse:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/12/code-climate.png" width="800">

In order to use `Test Coverage` feedback, we will also need to copy the `Test Reporter ID` from `Settings > Test Coverage` and add it into CircleCI environment variable.

In CircleCI, navigate to `Project > Settings > Environment variable` and add `CODECLIMATE_REPO_TOKEN` with copied id.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/12/env.png" width="800">

### Heroku Deployment Setup
In order to deploy `React` on [Heroku](https://heroku.com) , we will use a [buildpack](https://github.com/mars/create-react-app-buildpack). Do the following:

```bash
$ heroku create REPLACE_APP_NAME_HERE --buildpack https://github.com/mars/create-react-app-buildpack.git
$ git push heroku master
$ heroku open
```

We pushed the latest `master` branch to `heroku` with `git push heroku master`. A webpage will be open at the end showing the **Welcome to React** page.

Next, we will have to navigate to the newly create app in [Heroku Dashboard](https://dashboard.heroku.com/apps)  to setup automated deployment. Do the following on the dashboard:

- Go to **Deploy** tab and **Connect** to the correct github repo.
- **Enable** Automatic deployment and **check** `Wait for CI to pass before deploy`

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/12/cd.png" width="800">

## We are done!
With a few setup, we have a fully automated continuous integration and deployment suite ready. Now with every commit that is pushed to [GitHub](https://github.com), it will send a trigger to [CircleCI](https://circleci.com) and [CodeClimate](https://codeclimate.com). Once the test passed, if it was on the master branch, it will also be automatically deployed to [Heroku](https://heroku.com).

View the sample repo [here](https://github.com/Zaccc123/awesome-cicd-react) and the sample deployed website [here](https://awesome-cicd-react.herokuapp.com) !
