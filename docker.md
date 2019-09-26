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

