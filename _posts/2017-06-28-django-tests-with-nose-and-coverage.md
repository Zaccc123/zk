---
layout: post
title: "Django - tests with nose and coverage"
description: "Setup test coverage using django-nose"
date: 2017-06-28 16:39:18
comments: true
keywords: "django, nose, tdd, unit-test, django-test, nose-test"
category: django
tags:
- django
- python
---

The following guide would help us run test using `django-nose`. It give us more option to run test either by apps, modules or even just individual tests. We will also include `coverage` to give us a test report locally whenever test is run.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/17/coverage.png" width="800">

First, we installed the required package and update requirements.txt.

```bash
$ pip install django-nose
$ pip install coverage
$ pip freeze > requirements.txt
```

Next, we need to update `settings.py` to setup some configuration required by `django-nose`.

```python
INSTALLED_APPS = (
...
'django_nose', # Append to INSTALLED_APPS
...
)

TEST_RUNNER = 'django_nose.NoseTestSuiteRunner'
NOSE_ARGS = [
    '--cover-erase',
    '--cover-package=MY_APP', # Change `MY_APP` to your `app` name
]
```

Next, we will setup `coverage` to display a report whenever `python manage.py test` is run. In `manage.py` file, update it with this:

```python
if is_testing:
    import coverage
    cov = coverage.coverage(source=['app'], omit=['*/tests/*'])
    cov.set_option('report:show_missing', True)
    cov.erase()
    cov.start()
# Add this 5 line above

execute_from_command_line(sys.argv)

# and add this 4 line below
if is_testing:
    cov.stop()
    cov.save()
    cov.report()
```

The code here is asking `coverage` to report missing line that is not tested. The reason we are doing this manually instead of using a config file is because of a bug [here](https://github.com/django-nose/django-nose/issues/180).

By updating it to the above, whenever `python manage.py test` is run, we should see a nice report generated like this:

```bash
...................
--------------------------------------------------------------------
Ran 19 tests in 3.564s


OK
Destroying test database for alias 'default'...
Name                                     Stmts   Miss  Cover   Missing
--------------------------------------------------------------------
blog/__init__.py                                 0      0   100%
blog/admin.py                                   11      0   100%
blog/apps.py                                     3      3     0%   1-5
blog/forms.py                                    6      0   100%
blog/migrations/0001_initial.py                  9      0   100%
blog/migrations/0002_postimage.py                7      0   100%
blog/migrations/0003_auto_20170409_1343.py       5      0   100%
blog/migrations/0004_auto_20170410_2116.py       5      0   100%
blog/migrations/__init__.py                      0      0   100%
blog/models.py                                  31      3    90%   9, 37, 45
blog/tests.py                                  127      0   100%
blog/urls.py                                     3      0   100%
blog/views.py                                   65      2    97%   15, 58
--------------------------------------------------------------------
TOTAL                                          272      8   96%
```

If we are using CI, we could also generate a `html` or `xunittest` report. To get a `html` report, update the `manage.py` file to include this:

```python
cov.save()
cov.html_report(directory='covhtml')  # add this line
cov.report()
```

For generating a `xunittest` file, we have to update the `NOSE_ARGS` in settings.py to this:

```python
NOSE_ARGS = [
    '--cover-erase',
    '--cover-package=create_tasksapp',
    '--with-xunit', # Add this and the following line
    '--xunit-file=xunittest.xml',  # xunittest.xml could be any name
]
```

For more configuration, visit the following page:

- [django-nose](https://nose.readthedocs.io/en/latest/usage.html#configuration)
- [coverage](http://coverage.readthedocs.io/en/coverage-4.1/config.html)
