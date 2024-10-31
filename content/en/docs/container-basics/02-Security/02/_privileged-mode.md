---
title: "Privileged Containers"
weight: 28
sectionnumber: 2.8
---

## A note on privileged containers

The --privileged option in Docker is a special flag that gives the container full access to the host system. It’s much more powerful than simply assigning specific capabilities because it bypasses most of Docker's built-in security restrictions and grants the container elevated permissions, similar to running as root on the host. Often times you will see it in tutorials for running Docker inside Docker or similar things.

When a Docker container is run with the --privileged flag, several key things happen:

* All Capabilities Granted
* Full Device Access (Containers usually have limited access to devices on the host system (like /dev, which includes devices like disks, USBs, etc.))
* AppArmor and Seccomp Disabled
* Unrestricted Network Access

It’s recommended to avoid using --privileged unless necessary and to use capabilities and specific device access options instead for better security and control.

### Start a privileged container

We refrain from starting a privileged container on our shared infra. So please read the instructions in the `/home/project/welcome` file to access a dedicated VM with ssh:

```bash
cat /home/project/welcome
```

Copy and execute the relevant part:

```
ssh -i id-edcsa userx@192.168.0.1
```

This VM has docker already installed. Before starting a privileged container first check the mounted file systems, then just execute this command to run and access a privileged container:

```bash
df
docker run -it --rm --privileged ubuntu /bin/bash
```

Now try to escape this container from the inside (try to read a file like `/etc/passed` on the host). This time we don't provide a solution.
