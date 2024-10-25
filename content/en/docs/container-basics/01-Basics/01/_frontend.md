---
title: "Frontend"
weight: 3
sectionnumber: 1.3
---

## Frontend

Let us create a frontend to showcase port-forwarding and the connection between containers. We will run a simple python webserver which display all the users in our mariadb.

First get the IP of the currently running mariadb container. By default all container are started in the bridge network, where no DNS service is available and we can't use container name. As a workaround we use the IP of the container:

```bash
export ip=$(docker inspect mariadb-container-with-external-volume  -f '{{ range.NetworkSettings.Networks }}{{ .IPAddress }}{{ end }}')
```

`docker inspect <container>` shows you details about a running container in JSON format (run it yourself and take a look at it). We filtered the json to only get the IP address of the container.  
We could also have filtered the output with grep: `docker inspect mariadb-container | grep IPAddress` but our solution is more elegant 😊.

Next we start the frontend container, lucky for us there is an image available online. If interested you can check the source-code [here](https://github.com/songlaa/container-lab-fronted):

```bash
docker run -d --name frontend -e username=peter -e password=venkman -e servername=$ip grafgabriel/container-lab-frontend
```

Now how do we access it? Try to connect to the server using the container-assigned docker IP address:

```bash
docker inspect frontend -f '{{ range.NetworkSettings.Networks }}{{ .IPAddress }}{{ end }}'
```

Which will show only the IP of the container as output:

```
172.17.0.4
```

As we don't have a browser in the webshell use `curl http://172.17.0.4:5000` to open the page in your terminal. On alocal installation you could simply open <http://172.17.0.4:5000> using your browser.

```bash
curl http://172.17.0.4:5000`
```
