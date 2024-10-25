---
title: "Volume security"
weight: 24
sectionnumber: 2.4
---


So far we configured user and process permissions of the container. Another important step is to check the filesystem permissions and mount options of a container.

A common method is to run the containers with a read-only filesystem.  Let us try to write into a read-only mounted filesystem:

```bash
docker run --rm --read-only alpine sh -c 'echo "whatever" > /tmp/blub'
```

The command fails with an error:

```
sh: can't create /tmp/blub: Read-only file system
```

If you still need to write temporary files, we can do that using the `--tmpfs`, this will create a temporary in-memory filesystem which is gone as soon at the container is stopped.

Try it using:

```bash
docker run --rm --read-only --tmpfs /tmp alpine sh -c 'echo "whatever" > /tmp/blub'
```

In addition, if the volume is mounted only for reading, mount them as a read-only. It can be done by appending :ro to the -v. Here is an example:

```
docker run -v volume-name:/path/in/container:ro alpine
```

We continue on improving security for our `frontend` application by adding this options to our docker run command.

```bash
docker stop frontend
docker rm frontend
docker run --name frontend -d -e username=peter -e password=venkman -e servername=$ip --cap-drop ALL --security-opt=no-new-privileges --read-only --tmpfs /tmp container-lab-frontend:v2.0
```

You can check now with curl that our frontend is still running fine:

```bash
frontendIP=$(docker inspect frontend  -f '{{ range.NetworkSettings.Networks }}{{ .IPAddress }}{{ end }}')
curl http://$frontendIP:5000
```
