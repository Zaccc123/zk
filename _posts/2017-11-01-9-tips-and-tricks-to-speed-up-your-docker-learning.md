---
layout: post
title: "9 Tips and Tricks to speed up your Docker learning"
description: "Setup test coverage using django-nose"
date: 2017-11-01 16:39:18
comments: true
keywords: "dockers, tips, trick"
category: docker
tags:
- docker
- tips
---

Ever since I started using Docker in my development and production project, I really fell in love with it. When I was learning Docker(still learning), understanding the syntax and what each of it does, is kind of overwhelming at the start.

The following are things I wish I knew earlier when venturing into the learning of Docker. Most of it is going to be useful for beginners. If you already have lots of experience in Docker, this article might not be applicable.


## 9. Bash Completion
Refer to `--help` or using a cheatsheet is certainly useful. But sometime you know what you want, just can’t remember the exact command. Auto completion can certainly save you a hell lot of time.

`Docker CLI` actually have command line completion. If you are using `bash` or `zsh`, follow this guide [here](https://docs.docker.com/compose/completion/).

<iframe src="https://giphy.com/embed/3ohjUXSutaYMdsGVCo" width="520" height="80" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>


## 8. Editor highlight and linter
Once we venture into `docker-compose.yml` file, we suddenly have more keyword to remember other than just `FROM`, `RUN` and `WORKDIR`.

No matter which editor/ide you are using, having a linter and code highlight is going to prove useful here. You don’t want to spend an hour figuring out why your file doesn't work only to find out that is a typo(We all been there).

Atom:
- [language-docker](https://atom.io/packages/language-docker)
- https://atom.io/packages/linter-docker

VSCode
- [Docker - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)

Sublime
- [Dockerfile Syntax Highlighting - Packages - Package Control](https://packagecontrol.io/packages/Dockerfile%20Syntax%20Highlighting)


## 7. `Key: Value` format for compose file
The other thing i struggle with `docker-compose.yml` file the most is not understanding when to use :

```
key: value
```

or

```
key:
  - value_one
  - value_two
```

So the trick is actually for `key` that are pural, they usually expect a list of `values`. For example:

```
volumes:
  - /var/www/html/modules
  - /var/www/html/profiles
ports:
  - 8080:80
depends_on:
  - postgres_db
```

But not:

```
image: "postgres:9.6"
restart: always
container_name: postgres_db
```

## 6. Shorthand to remove container
While learning and playing with container, very often we need to delete it after use. I often do a `docker container ls -a` to list all container and copy the `CONTAINER_ID`  and paste it in `docker rm CONTAINER_ID` to remove it.

BUT, actually we could just type the first few characters (even one character) of the `CONTAINER ID` and it will work as well.

```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              
  STATUS              PORTS                    NAMES
967da8178908        side:development     "bash ./wait-for-i..."   About a minute ago   
  Up About a minute   0.0.0.0:8000->8000/tcp   side_app
8f1b7b22601f        postgres:9.5        "docker-entrypoint..."   About a minute ago   
  Up About a minute   0.0.0.0:5432->5432/tcp   side_db
$ docker rm 9
9
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             
  STATUS                      PORTS               NAMES
8f1b7b22601f        postgres:9.5        "docker-entrypoint..."   26 seconds ago      
  Exited (0) 10 seconds ago                       side_db

```

It even protect you from accidentally delete multiple container when they start with the same character:

```
$ docker rm 2
Error response from daemon: Multiple IDs found with provided prefix:
  2e684870456cd4cbb3a5a052fddb589cec55b4cf7e44b6c0662478ae77e1d6dd
```

## 5. `.dockerignore` should not be ignored!
Many time we start learning something by referring to tutorials online and testing it on our own side project. Somehow from the tutorials that I tried, none really explain or even use `.dockerignore`. Maybe is not in the scope of the tutorial, but in my opinion, it is really important to know it exists and what it does.

`.dockerignore`  file allows us to define rules and exceptions for files and folder to be excluded from the build context. Many time in our `Dockerfile`, we have `ADD` or `COPY`. With `.dockerignore`, it will first look into the rules and exclude whatever that is being defined.

The advantages are:
- Lean Docker Image
    - We usually want to exclude the `.git` folder from the final docker image.
- Hiding of Secret / Private Keys
    - We always want to exclude sensitive information like `aws` keys or secret keys in `.env`.
- Prevent Cache Invalidation
    - We usually have a `COPY . .` to copy the source code, here, we like to exclude the `.logs` file or test results like the `coverage.xml` that is frequently changed and not directly related to our source code. In this case, Docker can reuse the same cache if no changes are done and it saves us time from rebuilding it.

To understand more, please read up [here](https://docs.docker.com/engine/reference/builder/#dockerignore-file).

## 4. use a env file when there it require many variable
This is more of a personal preference than which is more "right".

In `docker-compose.yml` many time we specific the environment like this:

```
db:
  environment:
    - POSTGRES_USER =example
```

Personally, I feel that the list get really long especially when there is multiple containers defined in its own `environment`. I prefer to use a `.env` file instead. So in the `docker-compose.yml` it will look like this:

```
db:
	image: "postgres:9.5"
	env_file:
	  - postgres.env
  app:
    build: .
	env_file:
	  - app.env
```

Then in the respective `file.env`, it look something like this:

```
POSTGRES_DB=exampledb
POSTGRES_USER=example
POSTGRES_PASSWORD=qwe123qwe123
POSTGRES_HOST=examplehost
POSTGRES_PORT=5432
```

For the precedence of which environment variable is being taken when multiple environments are declared can be read up [here](https://docs.docker.com/compose/environment-variables/#the-env-file).

TLDR:
`environment` in compose file > `env_file` > `shell environment`

## 3. docker logs -f
Once you start getting the hang of using docker especially with compose command, you tend to run it with the `-d` detach flag. Once in awhile, the docker container is running but does not work like expected. I have this issue with a container running postgres.  I would usually just kill the container and start the container again.

It usually work, but you will not be able to fully understand what goes wrong. If cases like this happen again, use `docker logs -f CONTAINER_ID` to find out what exactly happen.

I manage to fix my problem with by using a `wait-for-it.sh` [script](https://github.com/vishnubob/wait-for-it)  after looking at the logs. Somehow, whenever I run without the `-d` flag the error just don’t happen.

So, the next time when a container does not work as expected, even if you are on detached mode, you can still look at the logs and debug it.

## 2. Subscribe and follow courses online
Nothing work better than following a expert courses online. However, before diving straight into courses, I would suggest you to mess around with docker yourself first. I notice that i learn a lot faster when messing around myself on my side project. Referring to officially documentation and other tutorial / article that i googled online.

Once I feel like is time to get serious about Docker, I look for popular courses online from Udemy.

There are a few by different Docker Captain, I recommend the following:
[Docker Mastery: The Complete Toolset From a Docker Captain | Udemy](https://www.udemy.com/docker-mastery/)

I purchased it when there is a discount and must say that it is really good. Bret is a docker captain and always keep his course material up to date. He actively replies to any question posted in Udemy.

## 1. Follow and speak with Docker Captain
Lastly, follow devops related topic, docker and docker captain in Twitter. Get invite and join their slack channels. Most of the time, I learn a lot just by what they shared and by the question posted by others.

Bret Fisher have a slack channel that is pretty active here: chat.dockermastery.com
