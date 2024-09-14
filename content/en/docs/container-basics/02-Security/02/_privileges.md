---
title: "2.2 Privileges"
weight: 22
sectionnumber: 2.2
---


Before exploring different options to minimize container privileges, it's important to address a fundamental yet frequently overlooked practice: keeping your software up to date. Regular updates are crucial for protecting against known container escape vulnerabilities, such as [Leaky Vessels](https://snyk.io/blog/cve-2024-21626-runc-process-cwd-container-breakout/), which often allow attackers to gain root access to the host. This means both the host system and Docker itself must be consistently updated, including the host kernel and Docker Engine.

Since containers share the host's kernel, a vulnerable kernel exposes all containers to risk. For instance, the [Dirty COW kernel privilege escalation exploit](https://github.com/scumjr/dirtycow-vdso), even if run inside a highly isolated container, would still lead to root access on a vulnerable host.

## User Management in Docker

We learnt that if nothing else is configured u user with a container runs as root. Configuring the container to use an unprivileged user is the best way to prevent privilege escalation attacks. This can be accomplished in three different ways as follows:

First during runtime using -u option of docker run command, check the differences

```bash
docker run alpine id
docker run -u guest alpine id
```

Note that the users we are running as must exist in the /etc/passwd of the Docker container. Otherwise, the command will fail as it fails to resolve the username to a user entry in the /etc/passwd file. As an alternative you can run it using an arbitrary uid. In all cases the user must have the necessary rights to execute the binaries or read the files needed in the container. For `alpine` this works because most binaries are set to read/execute for everyone (755).

A second way is to set it in the image. Simply add user in Dockerfile and use it

```
FROM alpine
RUN groupadd -r myuser && useradd -r -g myuser myuser
#    <HERE DO WHAT YOU HAVE TO DO AS A ROOT USER LIKE INSTALLING PACKAGES ETC.>
USER myuser
```

Let us apply those two ways to our frontend container as well. By checking the process on your local host we see that we started this container, despite our current knowledge, with root privileges:

```bash
docker top frontend
```

Which shows an output like similiar to this:

```
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                19086               19067               1                   22:45               ?                   00:00:00            /usr/local/bin/python3.9 /usr/local/bin/flask run --host=0.0.0.0 --port=5000
```

Let us stop and start the container with an user which does not exist on the host:

```bash
docker stop frontend
docker rm frontend
docker run -u 1001 --name frontend -e username=peter -e password=venkman -e servername=$ip container-lab-frontend:v1.0
docker top frontend
```

Ok, this is fine but even better would be to have it as an default already in the Dockerfile. Please change the Dockerfile of the frontend application to use an new user and build it with a tag of v2.0. Try do do it on you own before checking the solution.

{{% details title="I'm lost, show me the solution" %}}

Change your Dockerfile to match the content below:

```bash
# Use an official Python runtime as a parent image
FROM python:3.12-slim

# Set the working directory
WORKDIR /app

# Install required packages
RUN pip install flask mysql-connector-python

# Create a non-root user and group
RUN groupadd -r appuser && useradd -r -g appuser appuser

# For the installation we were root, now switch to the non-root user
USER appuser

# Copy the current directory contents into the container at /app
COPY . /app

# Set environment variable for Flask
ENV FLASK_APP=app.py

# Expose port 5000 for the Flask app
EXPOSE 5000

# Define the default command to run the app with Flask
CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
```

Now build it using the tag `v2.0` make sure you are in the frontend directory:

```bash
docker build -t container-lab-frontend:v2.0 .
```

Now we stop the currently running container and start our new one, hopefully we still have $ip saved in our shell:

```bash
docker stop frontend
docker rm frontend
docker run --name frontend -e username=peter -e password=venkman -e servername=$ip container-lab-frontend:v2.0
docker top frontend
```

{{% /details %}}

Both ways to change to user are fine. But what, if need to run an image which requires root privileges inside the container?
This is where the third option comes into play. It makes use of the Linux USER namespace to re-map the root user within the container to a less-privileged user in the host machine.

In this way, the container will be running as root, but that root is mapped to an user that has no privileges on the host.
User namespaces are not enabled by default and require to modify the start parametes for the docker deamon.

User namespaces are not enabled by default for Docker and require to modify the start parametes for the docker deamon. More on that topic in the [official documentation](https://docs.docker.com/engine/security/userns-remap/).

