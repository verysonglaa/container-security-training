---
title: "2.8 Container Runtime Security"
weight: 28
sectionnumber: 2.8
---

Do the following commands and outputs carefully:

```bash
user1@localhost$ whoami
user1@localhost$ head -1 /etc/shadow
```

Now do this:

```bash
docker run -v /etc:/host alpine sh -c 'whoami;head -1 /host/shadow'
```

Why can you suddenly read stuff which you shouldn't be allowed to?
Are containers not restriced? The answer lies in the architecture of Docker:

![Docker Architecture](../docker-architecture.png)

By default the Docker Deamon runs in root mode without user namespaces enabled.
This means we can mount anything from the host into our container and change it there too using the root user inside our container.

It gets worse, just read the exerpt below as we don't have sudo installed in our webshell:

```
user2@localhost$ whoami
user2
user2@localhost$ sudo su
Sorry, user user2 may not rund sudo on localhost.
user2@localhost$ docker run --rm -it -v /etc/sudoers.d:/host/etc/sudoers.d alpine sh
/ # echo 'user2 ALL=(ALL) NOPASSWD:ALL' > /host/etc/sudoers.d/user2
/ # exit
user2@localhost$ sudo su
root@localhost$ whoami
root
```

To ensure that a user running a container doesn't gain root access to your host, you need to run the container engine and the containerized process as a non-root user. This provides multiple layers of security between the service (httpd, MySQL, etc.) and the privileged resources in the operating system.

Running the container engine as a non-root user, is one layer of defense, while running the process in the container as a different non-root user offers yet another layer of defense.

Here is an except from the [docs](https://docs.docker.com/engine/security/rootless/):
Rootless mode executes the Docker daemon and containers inside a user namespace. This is very similar to userns-remap mode, except that with userns-remap mode, the daemon itself is running with root privileges, whereas in rootless mode, both the daemon and the container are running without root privileges.

Running rootless mode in Docker comes with a set of limitations, most notably that not all storage drivers are allowed and AppArmor is not supported.

Another solution would be to switch from Docker to [Podman](https://podman.io).

While Docker relies on a client-server model, Podman employs a daemonless architecture. With Podman's approach, users manage containers directly, eliminating the need for a continuous daemon process in the background.

Compared to Docker is has stronger default security settings, features like rootless containers, user namespaces, and seccomp profiles are enabled by default. On the image below we see a comparisation between Docker and Podman architecture

![Podman Architecture, src: https://dev.to/arafetki](../podman-architecture.jpeg)

## Recap

We saw how to run containers and how to secure them avoiding root, dropping capabilites, mounting filesystems readonly and using Linux Security Modules such as seccomp. However it is important to say that because of the architecture of docker anyone who can start container has more privileges on the host. It is still important to secure the Host Operating system and maybe to run a deamonless container technology like `podname`. What we did not touch are things like network security and monitoring. More an that in the Kubernetes Security lab.
