---
title: "Capabilities"
weight: 24
sectionnumber: 2.4
---

## Understanding container capabilities

Capabilities in Linux are fine-grained controls that are part of the POSIX permissions system. These capabilities allow you to limit or extend the privileges of a process.
Container capabilities are a set of predefined permissions that control what operations a container can perform on the host system. By default, containers run with a wide set of capabilities, but in many cases, they do not require all of them, so giving them only the permissions they need makes them safer to use.

Some of the most commonly used capabilities in Docker include:

* CAP_NET_BIND_SERVICE: Allows binding to ports below 1024 (e.g., running a web server on port 80).
* CAP_SYS_ADMIN: Provides broad system administration privileges (often considered too powerful for most use cases).
* CAP_CHOWN: Allows changing file ownership.
* CAP_DAC_OVERRIDE: Allows bypassing file read, write, and execute permission checks.

By default, a container has a wide range of these capabilities, but you can restrict or grant specific capabilities using configuration options when starting the container.

Check the default capabilites of a container:

```bash
docker run --rm -it alpine sh -c 'apk add -U libcap; capsh --print'
```

Important for us are the sections `Current` and `Bounding Set`, these are the Capabilites a process in our container has or can aquire. You can deny processes from gaining more privileges by adding `--security-opt=no-new-privileges` to the docker run command.

Furthermore we have `Current Inheritable and Blocked (IAB) Capabilities`
, this shows the capabilities that the process does NOT have. Each capability here is prefixed with !, indicating that the capability is disabled or not granted.

## Configuring a Container to Use Only What It Needs

In Docker, you can drop unnecessary capabilities or explicitly add only the required ones using the --cap-drop and --cap-add options in the command line.

Example: Configuring a Web Server with Minimal Capabilities
Consider running an Nginx web server inside a Docker container. By default, the Nginx process needs to bind to port 80, but it doesn't need many other elevated privileges.

Let us optimize our Frontend Application even further. We stop it and then start it with no privileges at all:

```bash
docker stop frontend
docker rm frontend
export ip=$(docker inspect mariadb-container-with-external-volume  -f '{{ range.NetworkSettings.Networks }}{{ .IPAddress }}{{ end }}')
docker run --name frontend -d -e username=peter -e password=venkman -e servername=$ip --cap-drop ALL --security-opt=no-new-privileges container-lab-frontend:v2.0
```

Let's see if it is still running

```bash
frontendIP=$(docker inspect frontend  -f '{{ range.NetworkSettings.Networks }}{{ .IPAddress }}{{ end }}')
curl http://$frontendIP:5000
```

and you should still see the available users in the backend database, we just dropped all CAPs the process in the container doesn't need implementing a least privelege strategy.
