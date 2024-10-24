---
title: "1.4 Dockerfile"
weight: 4
sectionnumber: 1.4
---

## Dockerfile

Docker can build container images by reading the instructions on how to build the image from a so-called Dockerfile or more generally, Containerfile.
The basic docs on how Dockerfiles work can be found at <https://docs.docker.com/engine/reference/builder/>.

## Write your first Dockerfile

Let us have a general look at how to build a container image.
For that, create a new directory with an empty Dockerfile in there.

```bash
mkdir myfirstimage
cd myfirstimage
```

Create a new File with the name `Dockerfile` and add the following content to that `Dockerfile` using your editor of choice:

```Dockerfile
FROM ubuntu
RUN apt-get update && \
    apt-get install -y figlet && \
    apt-get clean
```

* `FROM` indicates the base image for our build
* Each `RUN` line will be executed by Docker during the build
* Our RUN commands must be non-interactive (no input can be provided to Docker during the build)
* Check <https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/> for further best practices on how to write Dockerfiles.

## Build the image

Just run:

```bash
docker build -t myfirstimage .
```

* `-t` indicates the tag to apply to the image
* `.` indicates the location of the build context (which we will talk more about later, but is basically the directory where our Dockerfile is located)

Please note that the tag can be omitted in most Docker commands and instructions. In that case, the tag defaults to `latest`. Besides being the default tag there's nothing special about `latest`. Despite its name, it does not necessarily identify the latest version of an image.  
Depending on the build system it can point to the last image pushed, to the last image built from some branch, or to some old image. It can even not exist at all.  
Because of this, you must never use the `latest` tag in production, always use a specific image version.  
Also see: <https://medium.com/@mccode/the-misunderstood-docker-tag-latest-af3babfd6375>

### What happens when we build the image

The output of the Docker build looks like this:

```
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM ubuntu
 ---> ea4c82dcd15a
Step 2/2 : RUN apt-get update &&     apt-get install -y figlet &&     apt-get clean
 ---> b3c08112fd1c
Successfully built b3c08112fd1c
Successfully tagged myfirstimage:latest
```

### Sending the build context to Docker

```
Sending build context to Docker daemon 84.48 kB
...
```

* The build context is the `.` directory given to docker build
* It is sent (as an archive) to the Docker daemon by the Docker client
* This allows you to use a remote machine to build using local files
* Be careful (or patient) if that directory is big and your connection is slow

### Inspecting step execution

```
...
Step 1/2 : FROM ubuntu
 ---> ea4c82dcd15a
Step 2/2 : RUN apt-get update &&     apt-get install -y figlet &&     apt-get clean
 ---> b3c08112fd1c
Successfully built b3c08112fd1c
Successfully tagged myfirstimage:latest
```

* A container (ea4c82dcd15a) is created from the base image
  * The base image will be pulled, if it was not pulled before
* The `RUN` command is executed in this container
* The container is committed into an image (b3c08112fd1c)
* The build container (ea4c82dcd15a) is removed
* The output of this step will be the base image for the next one
* ...

### The caching system

If you run the same build again, it will be instantaneous.
Why?

* After each build step, Docker takes a snapshot
* Before executing a step, Docker checks if it has already built the same sequence
* Docker uses the exact strings defined in your Dockerfile:
  * `RUN apt-get install figlet cowsay` is different from
  * `RUN apt-get install cowsay figlet`
  * `RUN apt-get update` is not re-executed when the mirrors are updated
* All steps after a modified step are re-executed since the filesystem it's based on may have changed

You can force a rebuild with docker build --no-cache ...

### Run it

Now run your image

```bash
docker run -ti myfirstimage
```

You'll find yourself inside a Bash shell in the container, execute

```bash
figlet hello
```

and you will see the following output:

```
root@00f0766080ed:/# figlet hello
 _          _ _
| |__   ___| | | ___
| '_ \ / _ \ | |/ _ \
| | | |  __/ | | (_) |
|_| |_|\___|_|_|\___/

root@00f0766080ed:/#
```

exit the container by executing:

```bash
exit
```

## The CMD instruction in Dockerfile

With the `CMD` instruction in the Dockerfile, we can define the command that is executed when a container is started.

{{% details title="🤔 Can you find out which CMD instruction the ubuntu image has?" %}}
You did find yourself in a shell, so the instruction must either be `/usr/bin/bash` or `/usr/bin/sh`.
{{% /details %}}

Modify the previously created Dockerfile as follows:

```Dockerfile
FROM ubuntu
RUN apt-get update && \
    apt-get install -y figlet && \
    apt-get clean

CMD ["figlet", "hello"]
```

Build the image with:

```bash
docker build -t myfirstimagecmd .
```

And run it:

```bash
docker run -ti myfirstimagecmd
```

It directly executes the defined command and prints out

```bash
 _          _ _
| |__   ___| | | ___
| '_ \ / _ \ | |/ _ \
| | | |  __/ | | (_) |
|_| |_|\___|_|_|\___/

```

Check out <https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact> for more information.

## Frontend app image build

Now we are familiar with the image building process we have a more detailed look at our frontend image. You can find the source code [here](https://github.com/songlaa/container-lab-fronted).

We see that the developer has become quite lazy and has not updated the python to the latest version, he did not even care to create sensible tags. So let us do it ourselves using the tag `v1.0`

Check out the repository

```bash
cd /home/project
git clone https://github.com/songlaa/container-lab-fronted
cd container-lab-fronted
```

You should have the necessary knowledge now to update and rebuild the image locally. Delete the currently running container and start a new one with updated python.

{{% details title="😳 I'm lost, show me the solution" %}}

First of all we need to check for the latest python base image. You could do this in dockerhub:

```bash
grep FROM Dockerfile
```

We see that currently we use version 3.9 of python, a look at [https://hub.docker.com/_/python](https://hub.docker.com/_/python) shows us that that the most recent version, at the time of writing is 3.12.

Replace the from line with this new value

```bash
FROM python:3.12-slim
```

And then we build it using tag `v1.0`

```bash
docker build -t container-lab-frontend:v1.0 .
docker images
```

Finally we kill the currently running container and start our new one, hopefully we still have $ip saved in our shell:

```bash
docker stop frontend
docker rm frontend
docker run -d --name frontend -e username=peter -e password=venkman -e servername=$ip container-lab-frontend:v1.0
```

{{% /details %}}


{{% details title="🤔 What did we update by rebuilding the image?" %}}
We did not only update python to a recent version but also the modules in python!
Generally you should build & deploy very often to avoid configuration drift and keep your software up to date!
A common solution to update your dependencies is [https://docs.renovatebot.com/](https://docs.renovatebot.com/)
{{% /details %}}
