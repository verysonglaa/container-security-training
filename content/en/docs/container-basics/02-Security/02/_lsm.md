---
title: "Linux Security Modules"
weight: 26
sectionnumber: 2.6
---


## Introduction to Linux Security Modules

Linux Security Modules (LSMs) provide mechanisms for implementing various security policies in Linux. They help in enforcing access controls and securing applications by restricting their capabilities and interactions with the system. The most populare LSMs are:

**Seccomp (Secure Computing Mode)**:

Seccomp provides a way to filter system calls that a process can make. By defining a list of allowed or disallowed system calls, it can minimize the attack surface of applications by reducing the risk of exploitation through system call vulnerabilities. This allows you to create even more granular control over the system calls than you would have using capabilities.

**AppArmor (Application Armor)**:

AppArmor uses profiles to confine applications and restrict their access to system resources. Each profile defines the permissions for a specific application, including which files it can access, which network operations it can perform, and more. AppArmor is generally easier to manage and configure compared to SELinux. There is a default enabled AppArmor profile for Docker named usually named `docker-default`. You can find the template for this profile [here](https://github.com/moby/moby/blob/master/profiles/apparmor/template.go).

**SELinux (Security-Enhanced Linux)**:

SELinux provides a robust mechanism for supporting access control policies. It enforces security policies that dictate how processes interact with each other and with system resources, based on labels assigned to files, processes, and other objects. SELinux offers more granularity and flexibility than AppArmor but can be more complex to configure.

### Applying Seccomp to our frontend container

Seccomp can be used to restrict the system calls available to a container, thereby limiting its potential attack surface. Here’s how you can apply a Seccomp profile to an Nginx container:

First, create a Seccomp profile in JSON format. For example, create a file named `frontend-seccomp.json` under `/home/project` with the following content to restrict some potentially risky system calls:

```json
{
    "defaultAction": "SCMP_ACT_ALLOW",
    "syscalls": [
    {
        "names": [
        "accept",
        "bind",
        "connect",
        "getcwd",
        "getdents",
        "getpid",
        "recvfrom",
        "sendto",
        "socket"
        ],
        "action": "SCMP_ACT_ALLOW"
    }
    ]
}
```

This profile allows only a subset of system calls necessary for frontend to operate, blocking others. Adding another layer of defense in addition to our dropped capabilites.

To apply this profile to our frontend container, you can use the Docker command line with the `--security-opt` option:

```bash
cd /home/project
docker stop frontend
docker rm frontend
docker run --name frontend -d -e username=peter -e password=venkman -e servername=$ip \
        --cap-drop ALL --security-opt=no-new-privileges \
        --read-only --tmpfs /tmp \
        --security-opt seccomp=frontend-seccomp.json container-lab-frontend:v2.0
```

Replace `/path/to/nginx-seccomp.json` with the actual path to your Seccomp profile.

You can check if the Seccomp profile is applied by inspecting the container:

```bash
docker inspect frontend | grep seccomp
```

And again as a final check we test if our service is still available:

```bash
frontendIP=$(docker inspect frontend  -f '{{ range.NetworkSettings.Networks }}{{ .IPAddress }}{{ end }}')
curl http://$frontendIP:5000
```

AppArmor, Seccomp or SELinux can also play an important roles in mitigating unpatched vulnerabilities like [Leaky Vessels](https://www.redhat.com/en/blog/latest-container-exploit-runc-can-be-blocked-selinux)
