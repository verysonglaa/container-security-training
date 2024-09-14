---
title: "1.1 Images"
weight: 1
sectionnumber: 1.1
---

## Docker images

You can search for images available on [Docker Hub](https://hub.docker.com) by clicking the **Explore** link or by typing `mariadb` into the search field: <https://hub.docker.com/search/?q=mariadb&type=image>

You will get a list of results and the first hit will probably be the official image: <https://hub.docker.com/_/mariadb>

This page contains instructions on how to pull the image. Let's pull a certain version of mariadb:

```bash
docker pull mariadb:11.5
```

{{% alert title="Note" color="primary" %}}
Care about security! Check the images before you run them.
{{% /alert %}}

* Is it an [official image](https://docs.docker.com/docker-hub/official_images/)?
Official Images are a good starting point, please read [here](https://docs.docker.com/docker-hub/official_images/) why.

* What is installed in the image?
  * Read the Dockerfile that was used to build the image
  * Check the base image
  * Check the vulnerabilitis of this image. Does it affect your application?
  * Check the dependencies of the image.
  * Compare your images Digest to the sha256 value shown on dockerhub.

After an image has been downloaded, you may then run a container using the downloaded image with the sub-command `run`. If an image has not been downloaded when Docker is executed with the sub-command `run`, the Docker client will first download the image, then run a container using it:

```bash
docker run hello-world:linux
```

{{% alert title="Note" color="primary" %}}
Here we use the `linux` tag of the hello-world image instead of using `latest` again.
{{% /alert %}}

To see the images that have been downloaded to your computer type:

```bash
docker images
```

The output should look similar to the following:

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mariadb             11.5                58730544b81b        2 weeks ago         397MB
hello-world         latest              1815c82652c0        2 months ago        1.84kB
hello-world         linux               1815c82652c0        2 months ago        1.84kB
```

The hello world container you ran in the previous lab is an example of a container that runs and exits, after emitting a test message. Containers, however, can be much more useful than that, and they can be interactive. After all, they are similar to virtual machines, only more resource-friendly.

As an example, let's run a container using the downloaded image of MariaDB. The combination of the -i and -t switches gives you interactive shell access to the container:

```bash
docker run -it mariadb:11.5
```

An error has popped up!

```bash
2022-08-09 08:19:21+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-08-09 08:19:21+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-08-09 08:19:21+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-08-09 08:19:21+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
        You need to specify one of MARIADB_ROOT_PASSWORD, MARIADB_ALLOW_EMPTY_ROOT_PASSWORD and MARIADB_RANDOM_ROOT_PASSWORD
```

{{% details title="🤔 Why do I get an error? Is this a bug in the image?" %}}
Everything is fine, to run this image there is some configuration needed. Read the following excerpt carefully.

```bash
error: database is uninitialized and password option is not specified
        You need to specify one of MARIADB_ROOT_PASSWORD, MARIADB_ALLOW_EMPTY_ROOT_PASSWORD and MARIADB_RANDOM_ROOT_PASSWORD
```

We will add the configuration later.
{{% /details %}}

{{% details title="🤔 What's an image?" %}}

Think of an image like a blueprint of what will be in a container when it runs.

* An image is a collection of files + some metadata (or in technical terms: those files form the root filesystem of a container)
* Images are made of layers, conceptually stacked on top of each other
* Each layer can add, change or remove files
* Images can share layers to optimize disk usage, transfer times and memory use
* You build these images using Dockerfiles (in later labs)
* Images are immutable, you cannot change them after creation
{{% /details %}}

{{% details title="🤔 What's the difference between a container and an image?" %}}
When you run an image, it becomes a container.

* An image is a read-only filesystem
* A container is an encapsulated set of processes running in a read-write copy of that filesystem
* To optimize container boot time, copy-on-write is used instead of regular copy
* docker run starts a container from a given image
{{% /details %}}
