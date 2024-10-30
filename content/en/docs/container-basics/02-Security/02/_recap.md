---
title: "Recap"
weight: 29
sectionnumber: 2.9
---

We saw how to run containers and how to secure them avoiding root, dropping capabilites, mounting filesystems readonly and using Linux Security Modules such as seccomp. However it is important to say that because of the architecture of docker anyone who can start container has more privileges on the host.

It is still important to secure the Host Operating system and maybe to run a deamonless container technology like `podname`. What we did not touch are things like network security and monitoring. More an that in the Kubernetes Security lab.

![Give me more](../give-me-more.jpg)

If you are still good on time you can play around with escaping Docker containers yourself:

## (Optional) Docker Breakout

We will explore Docker breakout techniques to demonstrate potential privilege escalation risks in Docker environments. The exercises will guide you through creating Docker containers, testing breakout scenarios, and understanding the mechanisms that can be exploited to gain unauthorized access outside the container.

**Note**: You will not be given direct solutions. The exercises increase in difficulty, encouraging you to think critically and apply security concepts.

### Device Injection

Launch a Container with Host Device Access, use the `df` command to get the Host's file-systems. Here is an example:

```bash
df
docker run -it --rm --device /dev/sda1 ubuntu /bin/bash
```

Now check if you can view or access the host’s devices (such as `/dev/sda1`).

### Escape via Docker Socket

Run a Container that binds the Docker socket i.e. with `-v /var/run/docker.sock:/var/run/docker.sock`. Use Docker commands within the container to explore breakout opportunities.

### Host PID Namespace Access

Execute the following command

```bash
sleep 900 &
docker run -it --rm --pid=host ubuntu /bin/bash
```

This runs a Container with `--pid=host`. Explore how this enables access to host processes. Attempt to kill the sleep process.

## (Optional) Podman

You can install Podman alongside docker to test it out:

```bash
sudo apt -y install podman
```

Try different user command with podman. Try to start a pod. Try with `buildah` to build an image.
