---
title: "2.2 Privileges"
weight: 22
sectionnumber: 2.2
---


Before exploring different options to minimize container privileges, it's important to address a fundamental yet frequently overlooked practice: keeping your software up to date. Regular updates are crucial for protecting against known container escape vulnerabilities, such as [Leaky Vessels](https://snyk.io/blog/cve-2024-21626-runc-process-cwd-container-breakout/), which often allow attackers to gain root access to the host. This means both the host system and Docker itself must be consistently updated, including the host kernel and Docker Engine.

Since containers share the host's kernel, a vulnerable kernel exposes all containers to risk. For instance, the [Dirty COW kernel privilege escalation exploit](https://github.com/scumjr/dirtycow-vdso), even if run inside a highly isolated container, would still lead to root access on a vulnerable host.

# User Management in Docker

We learnt that if nothing else is configured u user with a container runs as root. Configuring the container to use an unprivileged user is the best way to prevent privilege escalation attacks. This can be accomplished in three different ways as follows:

First during runtime using -u option of docker run command, check the differences

```bash
docker run alpine id
docker run -u guest alpine id
```

Note that the users we are running as must exist in the /etc/passwd of the Docker container. Otherwise, the command will fail as it fails to resolve the username to a user entry in the /etc/passwd file.

A second way is to do it during build time. Simply add user in Dockerfile and use it

```
FROM alpine
RUN groupadd -r myuser && useradd -r -g myuser myuser
#    <HERE DO WHAT YOU HAVE TO DO AS A ROOT USER LIKE INSTALLING PACKAGES ETC.>
USER myuser
```

Finally you can enable user namespace support (--userns-remap=default) in Docker daemon, this requires to start the Docker Deamon with an different option.
This is an option for applications that need to be executed inside the container with the UID 0 (root). For these cases, using the --user parameter to specify a user with less privileges would not be possible. It makes use of the Linux USER namespace to re-map the root user within the container to a less-privileged user in the host machine. In this way, the container will be running as root, but that root is mapped to an user that has no privileges on the host.
User namespaces are not enabled by default and require to modify the start parametes for the docker deamon. More on that topic in the [official documentation](https://docs.docker.com/engine/security/userns-remap/).
