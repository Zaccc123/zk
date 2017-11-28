---
layout: post
title: "Guide on using ReactJS with Django Part 1/3"
description: "Setup ReactJS component with a Django project."
date: 2017-04-29 16:39:18
comments: true
keywords: "reactjs, react django, python"
category: reactjs
tags:
- reactjs
- django
- python
---

This is a step by step guide on setting up `ReactJS` to work with a `Django` backend. It can be use on a existing project or a brand new project.

At the end of this three-part guide, you will be able to write frontend in `ReactJS ` and still have a comfortable and familiar `Django` backend to work with by part two. At part three, we would make `ReactJS` more awesome by using the enabling hot reload!

## Assumption
You should already have a strong understanding in `Django`. This guide is only going to show how to setup `ReactJS` with `Django`. Limited explanation on other topic is given to keep this guide short.

## Getting Started
`git clone` the following [project](https://github.com/Zaccc123/react-django) and checkout to `start` branch to follow along.

### Step 1: Create a virtualenv and install pip
Navigate to the clone project and do this:

```bash
$ cd react-django
$ virtualenv react_django_env
(react_django_env)$ source react_django_env/bin/activate
(react_django_env)$ pip install -r requirements.txt
```

If you use `virtualenvwrapper`, do this:

```bash
$ cd react-django
$ mkvirtualenv react_django_env
(react_django_env)$ pip install -r requirements.txt
```

This steps only installed `django v1.11`,  `psycopg2` for postgres and `dotenv` to read our environment variable from `.env` file later.

### Step 2: Create a postgres user and database and do migration

```psql
$ psql
# CREATE USER reactuser;
# CREATE DATABASE reactdjango OWNER reactuser;
# \q
$ python manage.py migrate
```

We have create a postgres user: `reactuser` and database: `reactdjango`. At this stage, migration should run successfully.

### Step 3: Setup ReactJS, webpack with npm

```
$ npm init
$ npm install --save-dev react react-dom react-hot-loader@next babel-cli babel-loader babel-preset-react
    webpack webpack-bundle-tracker webpack-dev-server
```

If everything is good, you should see this in the tail of `package.json`:

```
"devDependencies": {
  "babel-cli": "^6.24.1",
  "babel-loader": "^7.0.0",
  "babel-preset-react": "^6.24.1",
  "react": "^15.5.4",
  "react-dom": "^15.5.4",
  "react-hot-loader": "^3.0.0-beta.6",
  "webpack": "^2.4.1",
  "webpack-bundle-tracker": "^0.2.0",
"webpack-dev-server": "^2.4.5"
}
```

All the `babel` related package would allow us to write in modern javascript syntax. `webpack` related package would help us bundle and serve the static file when needed. `react`, `react-dom` is needed for `ReactJS` and `react-hot-loader` is something we will use in part two of this guide.

### Step 4: Add some scripts for easy development later

In `package.json` add this.

```
"scripts": {
  "build": "webpack --config webpack.config.js --progress --colors",
  "build-prod": "webpack --config webpack.prod.config.js --progress --colors",
  "watch": "node server.js"
}
```

### Step 5: Create web pack and related config

```bash
$ mkdir -p assets/static/js
$ touch assets/static/js/index.js
$ touch webpack.config.js
```

Inside `webpack.config.js`, do this:

```js
var path = require("path")
var webpack = require('webpack')
var BundleTracker = require('webpack-bundle-tracker')

module.exports = {
  context: __dirname,
  entry:[
    './assets/static/js/index',
	],
  output: {
      path: path.resolve('./assets/static/bundles/'),
      filename: '[name]-[hash].js',
  },
  plugins: [
    new BundleTracker({filename: './webpack-stats.json'}),
  ],
  module: {
    loaders: [
      { test: /\.jsx?$/, exclude: /node_modules/, loaders: ['babel-loader'], },
    ],
  },
  resolve: {
    modules: [
          'node_modules',
          'bower_components'
    ],
    extensions: ['.js', '.jsx']
  },
}
```

To check everything is working, compile the bundle by doing this:

```bash
$ ./node_modules/.bin/webpack --config webpack.config.js
```

In the editor, we should see a `assets/static/bundles/main-[hashes].js`

At this part, we already have a `ReactJS` app that is living inside `Django`. In [part two](https://www.botzeta.com/post/10/), the guide will focus on setting up `Django` to display a `ReactJS` using `webpack`.
