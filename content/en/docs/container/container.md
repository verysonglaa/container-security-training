---
title: Docker/Container Security
weight: 21
---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Namespaces](#namespaces)
    - [PID Namespace](#pid-namespace)
    - [Network Namespace](#network-namespace)
    - [Mount Namespace](#mount-namespace)
4. [Seccomp](#seccomp)
5. [AppArmor](#apparmor)
6. [SELinux](#selinux)
7. [Cgroups](#cgroups)
8. [Additional Security Practices](#additional-security-practices)
9. [Conclusion](#conclusion)

## Introduction

In this lab, you will learn about various security mechanisms available in Docker and containers, including namespaces, seccomp, AppArmor, SELinux, and cgroups. You will also learn how containers communicate with the Linux kernel and how to secure this communication.

## Prerequisites

- A system with Docker installed (preferably a Linux system)
- Basic knowledge of Docker and Linux commands

## Namespaces

Namespaces are one of the fundamental building blocks of container isolation. They provide isolated instances of global system resources, ensuring processes inside a container do not interfere with processes outside the container or in other containers.

### PID Namespace

**Theoretical Introduction:**
PID namespaces isolate the process ID number space, meaning that processes in different PID namespaces can have the same PID. This is essential for containers, as it allows each container to have its own init process (PID 1).

**Famous Exploits:**
There have been vulnerabilities related to improper handling of PID namespaces that could potentially lead to privilege escalation, such as CVE-2013-1763.

**Exercise:**

```bash
# Run a container in detached mode
docker run -d --name test-container nginx

# Enter the container's namespace
docker exec -it test-container bash

# Inside the container, check the process tree
ps aux

# From another terminal, check the process tree on the host
ps aux | grep test-container
```

### Network Namespace

**Theoretical Introduction:**
Network namespaces provide isolated network stacks, including interfaces, IP addresses, routing tables, and firewall rules. This allows each container to have its own network interfaces and IP addresses.

**Famous Exploits:**
Exploits can occur if containers are allowed to access the host's network namespace improperly. For example, CVE-2020-14386 was a Linux kernel vulnerability that could allow container escape through the network namespace.

**Exercise:**

```bash
# Check network interfaces on the host
ip a

# Enter the container's network namespace
docker exec -it test-container bash

# Inside the container, check network interfaces
ip a
```

### Mount Namespace

**Theoretical Introduction:**
Mount namespaces isolate the set of filesystem mount points seen by processes. Changes to mounts in one namespace are not visible to other namespaces, which helps in providing file system isolation.

**Famous Exploits:**
Improper mount namespace configurations can lead to vulnerabilities like CVE-2014-5206, where an attacker could escape the container by exploiting mounts.

**Exercise:**

```bash
# Inside the container, create a new file
docker exec -it test-container touch /tmp/container-file

# Check if the file is present on the host
ls /var/lib/docker/containers/<container-id>/mnt/tmp/

# It won't be there because the mount namespace is isolated
```

## Seccomp

**Theoretical Introduction:**
Seccomp (secure computing mode) is a security feature in the Linux kernel that restricts the system calls a process can make. This helps in limiting the attack surface by disallowing unnecessary or dangerous system calls.

**Famous Exploits:**
Seccomp has mitigated several kernel vulnerabilities, such as CVE-2014-0038, where improper system call handling could lead to privilege escalation.

**Exercise:**

```bash
# Run a container with a custom seccomp profile
cat <<EOF > my-seccomp-profile.json
{
    "defaultAction": "SCMP_ACT_ERRNO",
    "syscalls": [
        {
            "names": ["clone", "fork", "vfork"],
            "action": "SCMP_ACT_ALLOW"
        }
    ]
}
EOF

docker run --security-opt seccomp=my-seccomp-profile.json nginx
```

## AppArmor

**Theoretical Introduction:**
AppArmor (Application Armor) is a Linux security module that provides mandatory access control. It confines programs to a limited set of resources based on a specified profile.

**Famous Exploits:**
AppArmor has helped mitigate vulnerabilities like CVE-2016-9962, where a misconfigured profile could allow an attacker to escape the Docker container.

**Exercise:**

```bash
# Check the current AppArmor profiles
sudo aa-status

# Run a container with a custom AppArmor profile
docker run --security-opt "apparmor=your_profile" nginx
```

## SELinux

**Theoretical Introduction:**
SELinux (Security-Enhanced Linux) is another Linux security module providing mandatory access control. It uses security policies to define how processes and users can access resources.

**Famous Exploits:**
SELinux has been crucial in preventing exploits such as CVE-2017-5123, which could allow a process to gain unauthorized access to system resources.

**Exercise:**

```bash
# Check the SELinux status
sestatus

# Run a container with a specific SELinux label
docker run --security-opt label:type:your_type nginx
```

## Cgroups

**Theoretical Introduction:**
Cgroups (control groups) limit, account for, and isolate the resource usage of process groups. This includes CPU, memory, disk I/O, and network bandwidth.

**Famous Exploits:**
Misconfigurations in cgroups can lead to denial of service attacks, such as CVE-2014-3519, where an attacker could cause excessive resource consumption.

**Exercise:**

```bash
# Run a container with limited CPU and memory
docker run -d --name limited-container --cpus=".5" --memory="256m" nginx

# Check the resource usage
docker stats limited-container
```

## Additional Security Practices

**Theoretical Introduction:**
In addition to the above mechanisms, there are several best practices to follow to enhance container security. These include using minimal base images, keeping images up to date, scanning images for vulnerabilities, and running containers as non-root users.

**Exercise:**

```bash
# Example of running a container as a non-root user
docker run -u 1000:1000 nginx
```

## Conclusion

In this lab, you explored various aspects of Docker/container security, including namespaces, seccomp, AppArmor, SELinux, and cgroups. By following these practices and utilizing these tools, you can enhance the security of your containerized applications.
