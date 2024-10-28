---
title: "Frontend"
weight: 3
sectionnumber: 1.3
---

## Frontend

Let's create a frontend to demonstrate port-forwarding and container connections. We will run a simple Python web server that displays all users in our MariaDB.

First, get the IP of the currently running MariaDB container. By default, all containers are started in the bridge network, where no DNS service is available, so we can't use container names. As a workaround, we use the IP of the container:

```bash
export ip=$(docker inspect mariadb-container-with-external-volume -f '{{ range.NetworkSettings.Networks }}{{ .IPAddress }}{{ end }}')
```

`docker inspect <container>` shows details about a running container in JSON format (run it yourself to explore). We filtered the JSON output to get only the container's IP address.  
Alternatively, we could filter the output with `grep`, as in `docker inspect mariadb-container | grep IPAddress`, but our solution is more concise 😊.

Next, we’ll start the frontend container. Fortunately, an image is available online. If you're interested, you can check the source code [here](https://github.com/songlaa/container-lab-fronted):

```bash
docker run -d --name frontend -e username=peter -e password=venkman -e servername=$ip grafgabriel/container-lab-frontend
```

How do we access it? Try connecting to the server using the container's Docker-assigned IP address:

```bash
docker inspect frontend -f '{{ range.NetworkSettings.Networks }}{{ .IPAddress }}{{ end }}'
```

This will show only the IP of the container as output:

```
172.17.0.4
```

Since we don’t have a browser in the web shell, use `curl http://172.17.0.4:5000` to view the page in your terminal. On a local installation, you could simply open <http://172.17.0.4:5000> in your browser.

```bash
curl http://172.17.0.4:5000
```

```
