---
title: "1.3 Dockerfile"
weight: 4
sectionnumber: 1.4
---

## Dockerfile

Docker can build container images by reading the instructions on how to build the image from a so-called Dockerfile or more generally, Containerfile.
The basic docs on how Dockerfiles work can be found at <https://docs.docker.com/engine/reference/builder/>.

## Write your first Dockerfile

Before we extend our php image we are going to have a more general look at how to build a container image.
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

If you only want to trigger a partial rebuild, e.g. run `apt-get update` to install the latest updates,
you can use the following pattern:

```Dockerfile
ENV REFRESHED_AT 2020-03-13
RUN apt-get update
```

If you update the value of `REFRESHED_AT` it will invalidate the Docker build cache of that and all
the following steps, thus installing the latest updates.

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

```bash
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

After we got to grips with the image building basics, we now want to include the source code of our frontend app in an already-built container image. To achieve this we will create a Dockerfile.

The base image is our `php:8-apache` image which we used before. The `ADD` command allows us to add files from our current directory to the Docker image.
We use this command to add the application source code into the image.

{{% alert title="Note" color="primary" %}}
Use `.dockerignore` to exclude files from the Docker context being added to the container. It works the same as `.gitignore`: <https://docs.docker.com/engine/reference/builder/#dockerignore-file>
{{% /alert %}}

In the directory containing the subdirectory `php-app` create a Dockerfile with the following content:

```Dockerfile
FROM php:8-apache

# Copies the php source code to the correct location
ADD ./php-app/ /var/www/html/

# Install additional php extension
RUN docker-php-ext-install mysqli
```

### Build the php-app image

{{% alert title="Note" color="primary" %}}
Stop and delete the running `php-app` container first. Leave the database container running.
{{% /alert %}}

Now build the image:

```bash
docker build -t php-app .
```

### Run the php-app container

After a succesful build, run it:

```bash
docker run -d --network container-basics-training --name php-app -p8080:80 php-app
```

Now open a browser and navigate to <http://localhost:8080/db.php> (or in the webshell use `curl http://localhost:8080/db.php`).
You should get a response saying "Connected successfully".

## Optional lab

Configuration should always be separate from the source code, so the database connection details must not be inside the php file `db.php`.
Fix the code in the db.php file. According to the continuous delivery principles, we don't want usernames and passwords in our source code. Use the PHP global variable `$_ENV["<environment variable name>"]` to read environment variables inside the container. Challenge yourself, this time the code is hidden. Try to find the solution before looking at it.

{{% details title="Show me the solution" %}}
Replace the line

```php
$password = "venkman";
```

with

```php
$password = $_ENV["password"];
```

and run the container by passing the necessary env var:

```bash
docker run -d --name apache-php -e password=venkman -v $(pwd)/php-app:/var/www/html php:8-apache
```

{{% /details %}}
