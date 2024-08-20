---
title: "1. Getting started"
weight: 1
---

## The command line tool

First it is time to become familiar with the command line utility. Using Docker consists of passing at least one command. `docker --help` shows the available options:

```bash
docker --help
```

```bash
Usage: docker COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/home/user/.docker")
  -D, --debug              Enable debug mode
      --help               Print usage
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/home/user/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/home/user/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/home/user/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

To view the switches available to a specific command, type:

```bash
docker <command> --help
```

To view system-wide information about Docker, use:

```bash
docker info
```

## Hello world (with Docker images)

Docker containers are run from Docker images. By default, they pull these images from Docker Hub, a Docker registry managed by Docker Inc, the company behind the Docker project. Anybody can build and host their Docker images on Docker Hub, so for many applications and Linux distributions you'll find Docker images that are hosted on Docker Hub.

To check whether you can access and download images from Docker Hub, type:

```bash
docker run hello-world
```

The output, which should include the following, indicates that Docker appears to be working correctly:

```bash
Hello from Docker.
This message shows that your installation appears to be working correctly.
...
```

## Your first container 😃

With this command, we just ran our first container on our computers. It ran a simple process that printed a message to standard out, the container itself is not very useful though.

## A closer look at the docker command

We've learned that the term "Docker" is used somewhat imprecisely. It's employed to refer to various components such as the CLI (Command Line Interface), the Docker Engine, the OCI image format, and the Container runtime. Let's take a closer look at what's happening when we use the command

```bash
docker run --rm -d --name sleep-container alpine sleep 900
```

We will come to the meaning of `-rm` and the other arguments later on. For now we need only to know that we started a container which sleeps for 900 seconds on our host.
First let us get the process id of the sleep process we just started:

```bash
PID=`docker inspect --format '{{.State.Pid}}' sleep-container`
```

Let us see the process running:

```bash
ps -u root -U root --forest -f | grep -B1 $PID
```

we see something like this

```bash
root       50221       1  0 17:00 ?        00:00:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 724930591e3fbf44f9cacb60285c0420464c41f5a6366e2b4443c2b53e6cd251 -address /run/containerd/containerd.sock
root       53658   50221  0 17:00 ?        00:00:00  \_ sleep 900
```

indeed we see that we don't use docker as a container runtime but containerd at a higher level and runc at a lower level. The parent of each of these containerd-shim-runc-v2 processes is PID 1 on the system.

The shim becomes the parent process of the containerized application. It is responsible for tasks such as reaping zombie processes, handling container process I/O (standard input, output, error), and ensuring proper container cleanup upon exit. As a result containerd can upgrade and restart without affecting running containers.

Secondly we see that in the end a container is just a processes running on the host. If nothing else is configured it runs as root! Let us see the different isolation techniques beeing used:

```bash
sudo lsns -p $PID
```

which shows use the different (and newly created) namespaces beeing used for this container:

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

By comparision a simple sleep command in the current shell would run in the same namespaces as the parent shell giving no isolation.
