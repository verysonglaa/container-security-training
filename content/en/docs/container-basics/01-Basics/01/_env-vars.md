---
title: "Environment variables"
weight: 2
sectionnumber: 1.2
---


Why was there an error in the previous lab?  
The MariaDB server cannot run without a proper configuration. Docker can pass configuration variables into the setup process via environment variables. Environment variables are passed with the `-e` parameter, as in:

```bash
docker run -it -e MARIADB_ROOT_PASSWORD=my-secret-pw mariadb
```

After running the command, you will see output similar to this:

```
Initializing database

PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER!
To do so, start the server, then issue the following commands:

'/usr/bin/mysqladmin' -u root password 'new-password'
'/usr/bin/mysqladmin' -u root -h <host> password 'new-password'

Alternatively, you can run:
'/usr/bin/mysql_secure_installation'
```

Notice that we used the arguments `-it` (interactive terminal). You may have also found that MariaDB does not respond to the usual `CTRL-C`. To exit, Docker provides an escape sequence to detach from a container while leaving it running: press `CTRL-p`, then `CTRL-q`. In some web shells, these shortcuts may not work, so close the terminal and reopen it to continue.

To verify the container is running, use:

```bash
docker ps
```

Output should look like this:

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
7cb31f821233        mariadb             "docker-entrypoint..."   5 minutes ago       Up 5 minutes        3306/tcp            upbeat_blackwell
```

## Accessing the container

To reconnect to the container, use:

```bash
docker exec -it <container> bash
```

Here, `<container>` can be the `CONTAINER ID` (typically the first two characters suffice) or the `NAMES` from `docker ps`. In the example above, this could be `7cb31f821233` or `upbeat_blackwell`.

{{% alert title="Note" color="primary" %}}
The `docker exec` command requires either the ID or name of the container, plus the command to execute, such as `bash` for interactive use.
{{% /alert %}}

Executing the command should display:

`root@7cb31f821233:/#`

{{% alert title="Note" color="primary" %}}
Each time you connect to a container, you will be the user defined in the Dockerfile.
{{% /alert %}}

Now that we're connected, let's check if MariaDB is working:

```bash
mariadb -uroot -pmy-secret-pw
```

If successful, the MariaDB command line should appear:

```
Welcome to the MariaDB monitor. Commands end with ; or \g.
...
MariaDB [(none)]>
```

Type `exit;` to leave the MariaDB client, then type `exit` again to leave the container.

## Detached containers

Running a Docker container in "detached" mode (background) can be done with `-d`:

```bash
docker run -it -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb
```

Now, instead of seeing container logs, only the container ID is displayed. Verify by listing running containers:

```bash
docker ps
```

Stop any container as needed:

```bash
docker stop <container>
```

Or, to remove a container, use:

```bash
docker rm <container>
```

For a complete list, including stopped containers:

```bash
docker ps --all
```

## Mounting a volume in a container

{{% details title="🤔 What happens to my data when I remove the container?" %}}
It’s deleted. Containers don’t store data permanently without a persistence layer, so let's address that.
{{% /details %}}

The MariaDB container is a good example for using a persistent volume. We’ll create a Docker-managed volume for persistent MariaDB data:

```bash
docker volume create volume-mariadb
docker run --name mariadb-container-with-external-volume -v volume-mariadb:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb
```

Inspect your volume with:

```bash
docker volume inspect volume-mariadb
```

To add a new user to MariaDB, connect and run:

```bash
docker exec -it mariadb-container-with-external-volume mariadb -uroot -pmy-secret-pw
```

Inside MariaDB:

```sql
use mysql;
CREATE USER 'peter'@'%' IDENTIFIED BY 'venkman';
GRANT SELECT ON mysql.user TO 'peter'@'%';
```

Now quit MariaDB and the container:

```bash
exit
```

By using volumes, we have persisted the data in our database!
