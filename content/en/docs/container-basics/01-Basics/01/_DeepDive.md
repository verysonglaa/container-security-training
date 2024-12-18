---
title: "Under the hood"
weight: 6
sectionnumber: 1.6
---

## A closer look at the docker command and the runtime

We've learned that the term "Docker" is used somewhat imprecisely. It refers to various components such as the CLI (Command Line Interface), the Docker Engine, the OCI image format, and the Container runtime. Let's take a closer look at what's happening when we use the command:

```bash
docker run --rm -d --name sleep-container alpine sleep 900
```

We will see the meaning of `-rm` and the other arguments later. For now, we only need to know that we started a container that sleeps for 900 seconds on our host.
First let us get the process id of the sleep process we just started:

```bash
docker inspect --format '{{.State.Pid}}' sleep-container
```

Let us see the process running. In the webshell we have the docker backend running in another container, let us first change into that:

```bash
kubectl exec  $(kubectl get pod -l "app.kubernetes.io/name"=webshell -o name) -it -c dind -- sh
```

Don't worry the command will make sense after the Kubernetes Security training.

Let us see the process running on the host now, we don't have the necessary software installed in our lab so we are installing it now as well:

```bash
PID=$(pgrep sleep)
apk add procps
ps -u root -U root --forest -f | grep -B1 $PID
```

We see something like this

```bash
root       50221       1  0 17:00 ?        00:00:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 724930591e3fbf44f9cacb60285c0420464c41f5a6366e2b4443c2b53e6cd251 -address /run/containerd/containerd.sock
root       53658   50221  0 17:00 ?        00:00:00  \_ sleep 900
```

Indeed we see that we don't use docker as a container runtime but containerd at a higher level and runc at a lower level. The parent of each of these containerd-shim-runc-v2 processes is PID 1 on the system.

The shim becomes the parent process of the containerized application. It is responsible for tasks such as reaping zombie processes, handling container process I/O (standard input, output, error), and ensuring proper container cleanup upon exit. As a result, containerd can upgrade and restart without affecting running containers.

Secondly, we see that in the end a container is just a process running on the host. If nothing else is configured it runs as root! Let us see the different isolation techniques being used (again we need to install some software for that):

```bash
apk add util-linux
lsns -p $PID
```

Which shows use the different (and newly created) namespaces being used for this container:

```bash
        NS TYPE   NPROCS   PID USER COMMAND
4026531834 time      463     1 root /sbin/init splash # time isolation
4026531837 user      423     1 root /sbin/init splash # uid gid isolation (root inside is not root outside)
4026534002 mnt         1 53658 root sleep 900 
4026534003 uts         1 53658 root sleep 900 # hostname isolation
4026534004 ipc         1 53658 root sleep 900
4026534005 pid         1 53658 root sleep 900
4026534006 net         1 53658 root sleep 900
4026534064 cgroup      1 53658 root sleep 900
```

By comparision, a simple sleep command in the current shell would run in the same namespaces as the parent shell giving no isolation.

Don't forget to exit our Docker backend container if you work in the webshell

```bash
exit
```
