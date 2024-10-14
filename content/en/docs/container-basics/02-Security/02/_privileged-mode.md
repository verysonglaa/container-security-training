---
title: "2.7 Privileged Containers"
weight: 27
sectionnumber: 2.7
---

## A note on privileged containers

The --privileged option in Docker is a special flag that gives the container full access to the host system. It’s much more powerful than simply assigning specific capabilities because it bypasses most of Docker's built-in security restrictions and grants the container elevated permissions, similar to running as root on the host. Often times you will see it in tutorials for running Docker inside Docker or similar things.

When a Docker container is run with the --privileged flag, several key things happen:

* All Capabilities Granted
* Full Device Access (Containers usually have limited access to devices on the host system (like /dev, which includes devices like disks, USBs, etc.))
* AppArmor and Seccomp Disabled
* Unrestricted Network Access

It’s recommended to avoid using --privileged unless absolutely necessary and to use capabilities and specific device access options instead for better security and control.
