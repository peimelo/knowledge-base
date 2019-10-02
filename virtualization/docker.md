---
description: >-
  Docker, the technology, is a set of tools built around the idea of packaging
  and running software in small, sandboxed environments known as containers.
---

# Docker

## A Brave New World

### Installing Docker

[Installation for macOS](https://docs.docker.com/docker-for-mac/install/)

### Verifying Your Install

```text
$ docker version
```

### Running a Ruby Script Without Ruby Installed

```text
$ docker run ruby:2.6 ruby -e "puts :hello"
$ docker run [OPTIONS] <image> <command>
```

### Cleaning Up After Ourselves

To list the running containers:

```text
$ docker ps
```

To list all containers, including stopped ones:

```text
$ docker ps -a
```

To delete them:

```text
$ docker rm <container id> [<container id2> ...]
```

We can use the `--rm` option, which tells Docker to delete the container after it completes:

```text
$ docker run --rm ruby:2.6 ruby -e "puts :hello"
```

### Generating a New Rails App Without Ruby Installed

```text
$ docker run -i -t --rm -v ${PWD}:/usr/src/app ruby:2.6 bash
```

`-v` this mounts a volume—effectively sharing a portion of our local filesystem with the container. Specifically, `-v ${PWD}:/usr/src/app` says, “Mount our current directory inside the container at `/usr/src/app`”

`-i` interactive session

`-t` terminal inside the container

## Running a Rails App in a Container

### Defining Our First Custom Image

Create a Dockerfile with:

{% code-tabs %}
{% code-tabs-item title="Dockerfile" %}
```text
FROM ruby:2.6
  
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs
RUN apt-get install -yqq yarn

COPY . /usr/src/app/

WORKDIR /usr/src/app

RUN bundle install
RUN yarn install
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Building Our Image

```text
$ docker build [options] path/to/build/directory
$ docker build .
```

The `docker build` command doesn’t output a file; it simply makes the new image available, adding it to the list of images that Docker knows about. We list the images on our system with the following command:

```text
$ docker images
```

### Running a Rails Server with Our Image

Image ID in hand, we can start our Rails app inside a container based on our custom image with the following command. Let’s run it now:

```text
$ docker run -p 3000:3000 123456789999 bin/rails s -b 0.0.0.0
```

## Fine-Tuning Our Rails Image

```text
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              123456789999        8 minutes ago       1.1GB

$ docker tag 123456789999 railsapp

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
railsapp            latest              123456789999        10 minutes ago      1.1GB

$ docker tag railsapp railsapp:1.0

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
railsapp            1.0                 123456789999        11 minutes ago      1.1GB
```

Rather than tagging images after they’ve been built, we can tag them when we build the image using the `-t` option:

```text
$ docker build -t railsapp -t railsapp:1.0 .
```

Having named our image, we’re now able to start our Rails server using the image name, like so:

```text
$ docker run -p 3000:3000 railsapp bin/rails s -b 0.0.0.0

OR

$ docker run -p 3000:3000 railsapp:1.0 bin/rails s -b 0.0.0.0
```

### A Default Command

{% code-tabs %}
{% code-tabs-item title="Dockerfile" %}
```text
FROM ruby:2.6
  
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update -yqq
RUN apt-get install -yqq --no-install-recommends nodejs
RUN apt-get install -yqq yarn

COPY . /usr/src/app/

WORKDIR /usr/src/app

RUN bundle install
RUN yarn install

CMD ["bin/rails", "s", "-b", "0.0.0.0"]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```text
$ docker build -t railsapp .
$ docker run -p 3000:3000 railsapp
```

It’s important to note that the `CMD` instruction just provides a default command—you can specify a different one when you issue the docker run command. For example, to list our Rake tasks, we’d run:

```text
$ docker run --rm railsapp bin/rails -T
```

### Ignoring Unnecessary Files

{% code-tabs %}
{% code-tabs-item title=".dockkerignore" %}
```text
# GIT
.git
.gitignore

# Logs
log/*

# Temp files
tmp/*

# Editor temp files
*.swp
*.swo


```
{% endcode-tabs-item %}
{% endcode-tabs %}

After that, let's rebuild our image:

```text
$ docker build -t railsapp .
```

### Caching Issue 1: Updating Packages

{% code-tabs %}
{% code-tabs-item title="Dockerfile" %}
```text
FROM ruby:2.6

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | \
  apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | \
  tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update -yqq && apt-get install -yqq --no-install-recommends \
  nodejs \
  yarn

COPY . /usr/src/app/

WORKDIR /usr/src/app

RUN bundle install
RUN yarn install

CMD ["bin/rails", "s", "-b", "0.0.0.0"]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Caching Issue 2: Unnecessary Gem and Package Installs

{% code-tabs %}
{% code-tabs-item title="Dockerfile" %}
```text
FROM ruby:2.6.4

LABEL maintainer="peimelo@gmail.com"

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | \
  apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | \
  tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update -yqq && apt-get install -yqq --no-install-recommends \
  nodejs \
  yarn

# Install gems
COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app
RUN bundle install

# Install yarn packages
COPY package.json yarn.lock /usr/src/app/
RUN yarn install

COPY . /usr/src/app/

CMD ["bin/rails", "s", "-b", "0.0.0.0"]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Describing Our App Declaratively with Docker Compose

### Getting Started with Compose

{% code-tabs %}
{% code-tabs-item title="docker-compose.yml" %}
```text
version: '3'

services:

  web:
    build: .
    ports:
      - "3000:3000"

```
{% endcode-tabs-item %}
{% endcode-tabs %}

[Compose Versioning](https://docs.docker.com/compose/compose-file/compose-versioning/)

### Lauching Our App

Add the following line to the top of your `config/boot.rb` file:

{% code-tabs %}
{% code-tabs-item title="config/boot.rb" %}
```text
$stdout.sync = true

```
{% endcode-tabs-item %}
{% endcode-tabs %}

```text
$ docker-compose up

# -d is detaching mode, then the output is hide
$ docker-compose up -d

$ docker-compose stop
```

### Mounting a Local Volume

A mounted local volume represents some filesystem that’s shared between your local machine and the container:

{% code-tabs %}
{% code-tabs-item title="docker-compose.yml" %}
```text
version: '3'

services:

  web:
    build: .
    ports:
      - "3000:3000"
	volumes:
	  - .:/usr/src/app

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Starting and Stopping Services

```text
# what containers are currently running
$ docker-compose ps

# to target a particular service (web, etc):
$ docker-compose start <service_name>
$ docker-compose restart <service_name>
$ docker-compose stop <service_name>

$ docker-compose [start|stop|kill|restart|pause|unpause|rm] \
  <service name>

```

### Viewing the Container Logs

```text
# -f to follow the output
$ docker-compose logs -f web
```

### Rebuilding Our Images

```text
$ docker-compose build web
```

### Clean Up After Ourselves

```text
$ docker container prune
$ docker image prune
$ docker system prune
```

[Image Prune](https://docs.docker.com/engine/reference/commandline/image_prune/)

[Container Prune](https://docs.docker.com/engine/reference/commandline/container_prune/)

[System Prune](https://docs.docker.com/engine/reference/commandline/system_prune/)

## Beyond the App: Adding Redis

### Using docker run

To start a Redis server with docker run, we’d issue the following command:

```text
$ docker run --name redis-container redis
```

{% code-tabs %}
{% code-tabs-item title="docker-compose.yml" %}
```text
version: '3'

services:

  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/usr/src/app
  redis:
    image: redis
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now, let’s start our Redis server:

```text
$ docker-compose up -d redis
$ docker-compose logs redis
```

