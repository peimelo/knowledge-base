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
docker run -p 3000:3000 1234567890 bin/rails s -b 0.0.0.0
```

## Fine-Tuning Our Rails Image



