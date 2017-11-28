---
layout: post
title: "Displaying ReactJS app in Django Part 2/3"
description: "Displaying ReactJS component in Django with template"
date: 2017-05-01 16:39:18
comments: true
keywords: "reactjs, react django, python"
category: reactjs
tags:
- reactjs
- django
- python
---

After part one, we already have a `ReactJS` app that is living inside `Django`. In this part two, we will focus on setting up `Django` to display a `ReactJS` app using `webpack`.

If you have followed from [part one](https://www.botzeta.com/post/9/), use the same project. If not, to get a starter project from part two, you can checkout the part2 branch from [here](https://github.com/Zaccc123/react-django).

### Step 1: Install `django-wepack-loader`
In order to do this, we need to use `django-webpack-loader`. **(Ensure that the virtualenv is running)**  Do this:

```bash
(react_django)$ pip install django-webpack-loader
```

[django-webpack-loader](https://github.com/ezhome/django-webpack-loader) would allow `Django` to use the generated bundle that is output by `webpack-bundle-tracker`.

### Step 2: Update `settings.py`
Add `webpack-loader` to `INSTALLED_APPS`:

```py
INSTALLED_APPS = [
	...
	'webpack_loader',
]
```

Include the following in `settings.py`:

```py
WEBPACK_LOADER = {
    'DEFAULT': {
        'BUNDLE_DIR_NAME': 'bundles/',
        'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats.json'),
    }
}

PROJECT_ROOT = os.path.dirname(os.path.abspath(__file__))
STATIC_ROOT = os.path.join(PROJECT_ROOT, 'staticfiles')
STATIC_URL = '/static/'
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'assets/static'),
)
```

From the above, we tell `Django` we would like to use `webpack_loader` in the `INSTALLED_APPS` and setup the required config in `WEPACK_LOADER`.

Adding the `STATICFILES_DIRS` to point to `assets/static` is to ensure that when `python manage.py collectstatic` is run, it would fetch `staticfiles` from that given directory.

### Step 3: Start the real coding
With all the above setup, we can finally start the real coding. You can skip this step and go to **Step 4** if you are adding it to a existing project.

```bash
$ python manage.py startapp helloworld
```

Update `settings.py` to include `helloworld`.

```py
INSTALLED_APPS = [
	...
	'webpack_loader',
	'helloworld',
]
```

Create folder directory and a `index.html` in the newly created `helloword` app. `index.html` should be in: `./helloworld/templates/helloworld/index.html`

In `views.py` do this

```py
def index(request):
    return render(request, 'helloworld/index.html')
```

Then update `react_django/urls.py` to

```py
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'', include('helloworld.urls')),
]
```

Also create a `urls.py` in `helloworld`. It should look this like inside `helloworld/urls.py`.

```py
from django.conf.urls import url
from helloworld import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```

The above are just getting the browser to display `index.html` page when `localhost:8000` is visited. You should be quite familiar with this already.

### Step 4: Display ReactJS in Django
Inside `index.html` or inside any `.html` file if you are adding to a existing project, add the following line:

{% raw %}
1. `{% load render_bundle from webpack_loader %}` at the top
2. `{% render_bundle 'main' %}` inside a `<div id=“app”>` tag

So if a new `index.html`, it should do like this:

```html
{% load render_bundle from webpack_loader %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Hello World!</title>
  </head>
  <body>
    <div id="app"></div>
    {% render_bundle 'main' %}
  </body>
</html>
```
{% endraw %}

The `<div id="app">` could be any other given id. We just need to ensure that it matches the one we set later in `index.js`.

### Step 5: Start coding in React
Inside `index.js`, do this:

```js
var React = require('react')
var ReactDOM = require('react-dom')

const App = () => <h1>Hello world</h1>;

ReactDOM.render(<App/>, document.getElementById('app'))
```

The above are written using `EcmaScript6` syntax. Most `ReactJS` tutorial online are using this modern javascript syntax as well. We need to tell `babel` that we are writing in `react` so that they will apply the correct loader.

In root directory, create a `.babelrc` and add the following line:

```
{ "presets": ["react"], }
```

### Step 6: See React in action!
Now with all this completed, in order to view the `ReactJS` component in `Django`, we need to do the following:

```bash
$ npm run build
$ python manage.py collectstatic
$ python manage.py runserver
```

Visiting `localhost:8000`, we should see this:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/10/sample.png" width="800">

You could see that `Hello world` is being displayed by a file `main-[hashes].js`. This show that `ReactJS` is now working on our `Django` powered site!

## Summary
What we have did so far is creating a simple `ReactJS` that live together with our `Django` project. The linking between `Django` and `ReactJS` is make possible by `webpack`.

However, it seems too troublesome to run **three commands** just to quickly reload a server and view UI changes. This isn’t the kind `ReactJS` that we have heard of. In [Part Three](http://www.botzeta.com/post/11/), we will setup `react-hot-loader` to work in our project and also explain how to use it in production.
