---
layout: post
title: "Hot loading a ReactJS app in Django Part 3/3"
description: "Hot reloading a ReactJS app in Django!"
date: 2017-05-04 16:39:18
comments: true
keywords: "reactjs, react django, python"
category: reactjs
tags:
- reactjs
- django
- python
---

In part two, we setup the link between `Django` and `ReactJS` with the help of  `Webpack`. We need to run **three commands** just to quickly reload a server and view UI changes. In this guide, we will setup `react-hot-loader` to allow live changes to be reflected on our `Django` application that is showing the `react` components.

If needed, please revisit [part two](http://www.botzeta.com/post/10/). You can also checkout the part3 branch [here](https://github.com/Zaccc123/react-django) to continue.

### Step 1: Update webpack config
We will need to update web pack config to use `react-hot-loader`.

Update the `entry` to this:

```js
entry:[
	'react-hot-loader/patch',
	'webpack-dev-server/client?http://localhost:3000/',
	'webpack/hot/only-dev-server',
	'./assets/static/js/index',
],
```

`output` should be updated to:

```js
output: {
  path: path.resolve('./apps/static/bundles/'),
  filename: '[name]-[hash].js',
  publicPath: 'http://localhost:3000/static/bundles/',
},
```

Then finally `plugins` to:

```js
plugins: [
	new webpack.HotModuleReplacementPlugin(),
	new webpack.NoEmitOnErrorsPlugin(),
	new BundleTracker({filename: './webpack-stats.json'}),
],
```

What we are doing here are updating the config of `webpack` to cater for `react-hot-loader`. These are just a bunch of settings to follow. You can visit [webpack](https://webpack.js.org/guides/)  and [react-hot-loader](https://github.com/gaearon/react-hot-loader) to learn more. What you need to know is, during development, instead of going to usual directory to read static files, we are now going to `localhost:3000`.

Since we are using `react-hot-loader v3` we need to update `.babelrc`:

```js
{
  "presets": ["react"],
  "plugins": ["react-hot-loader/babel"]
}
```

### Step 2: Watch for changes
Now instead of recompiling whenever we make a change, we will run a server to do it.

Create a `server.js` file in root directory.

```js
var webpack = require('webpack')
var WebpackDevServer = require('webpack-dev-server')
var config = require('./webpack.config')

new WebpackDevServer(webpack(config), {
  publicPath: config.output.publicPath,
  hot: true,
  inline: true,
  historyApiFallback: true,
  headers: {
    'Access-Control-Allow-Origin': '*',
  },
}).listen(3000, '0.0.0.0', function (err, result) {
  if (err) {
    console.log(err)
  }
  console.log('Listening at 0.0.0.0:3000')
})
```

Once that is completed, we can run the server with either of the command below:

```bash
$ node server.js		#either this or
$ npm run watch 		#where we setup this in part 1
```

### Step 3: Update the `index.js`
Now, because we are using `react-hot-loader`, we need to update `index.js` and move any component that could possibility change to another file.

Update `index.js` to this:

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { AppContainer } from 'react-hot-loader';
import App from './App'

const render = Component => {
  ReactDOM.render(
    <AppContainer>
      <Component />
    </AppContainer>,
    document.getElementById('app')
  )
}
render(App)
if (module.hot) {
  module.hot.accept('./App', () => { render(App) })
}
```

The above are the exact configuration we take from the github [docs](https://github.com/gaearon/react-hot-loader/tree/master/docs#webpack-2). Next, we will need to create a new `App.js` that we are importing here.

```js
import React from 'react';

const App = () => {
    return (
      <div>
      <h1>Hello World!</h1>
    </div>
    );
}
export default App;
```

Thats all we need. Now `ReactJS` component live inside `Django` and with the help of `webpack` and `react-hot-loader` we can have a better development environment.

## See hot loader in action:
Just run `npm run watch` and another `python manage.py runserver`.
Keep the two session running while coding. This is what we will see:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/11/hot-loading.gif" width="800">

### Use in Production
In production, we does not need to use `react-hot-loader`. Therefore we should have a separate file called `wepack.prod.config.js`. This is what i have inside it:

```js
var path = require("path")
var webpack = require('webpack')
var BundleTracker = require('webpack-bundle-tracker')

module.exports = {
  context: __dirname,
  entry: [
      './assets/static/js/index'
  ],
  output: {
      path: path.resolve('./assets/static/bundles/'),
      filename: '[name]-[hash].js',
  },
  plugins: [
    new BundleTracker({filename: './webpack.prod-stats.json'}),
    new webpack.DefinePlugin({
      'process.env': {
        'NODE_ENV': JSON.stringify('production')
    }}),
    new webpack.optimize.OccurrenceOrderPlugin(),
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
  }
}
```

Basically the different is there is no `react-hot-loader` config and we also added a few `plugin` to remove debug code.

To run on production, ensure that the step of `npm run build-prod` is run before `collectstatic` when in production.

### References
Most of the content that I have here are gather from various sources when trying to implement `ReactJS` into my `Django` project.

- [Using Webpack transparently with Django + hot reloading React components as a bonus - Owais Lone](http://owaislone.org/blog/webpack-plus-reactjs-and-django/)
- [Set up a Django + ReactJS project with Webpack manager · GitHub](https://gist.github.com/Belgabor/130e7770575e74581b67597fcb61717e)
- [Webpack2 Getting Started](https://webpack.js.org/guides/get-started/)
- [GitHub - ezhome/django-webpack-loader: Transparently use webpack with django](https://github.com/ezhome/django-webpack-loader)

### Where to go from here
`ReactJS` is very much just another way to render view. It is not another `MVC` framework. In order to show is full potential, it need to be somehow connected with a backend services. Here, I am directly rendering it with a `Django` backend. However there are a lot more different setup, like [using it with redux](http://redux.js.org/docs/basics/UsageWithReact.html) or [flux](https://facebook.github.io/flux/).

## Live Production Example!
In the very same blog, the [App Page](https://www.botzeta.com/app/) are build with the same setup that I described here.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/11/app-page.png" width="800">
