---
title: "Environment variables"
weight: 2
sectionnumber: 1.2
---

## Environment variables

So why was there an error in the previous lab?
The MariaDB server is not able to run without a proper configuration. Docker can pass variables into the instantiation process via environment variables.
Environment variables are passed via the parameter `-e`, e.g.:

```bash
docker run -it -e MARIADB_ROOT_PASSWORD=my-secret-pw mariadb
```

Once you run the command you will see an output like this:

```

Initializing database

PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER !
To do so, start the server, then issue the following commands:

'/usr/bin/mysqladmin' -u root password 'new-password'
'/usr/bin/mysqladmin' -u root -h  password 'new-password'

Alternatively you can run:
'/usr/bin/mysql_secure_installation'

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the MariaDB Knowledgebase at http://mariadb.com/kb or the
MySQL manual for more instructions.

Please report any problems at http://mariadb.org/jira

The latest information about MariaDB is available at http://mariadb.org/.
You can find additional information about the MySQL part at:
http://dev.mysql.com
Consider joining MariaDB's strong and vibrant community:
https://mariadb.org/get-involved/

Database initialized
MySQL init process in progress...
2020-05-27  6:21:03 0 [Note] mysqld (mysqld 10.3.7-MariaDB-1:10.3.7+maria~jessie) starting as process 101 ...
2020-05-27  6:21:03 0 [Note] InnoDB: Using Linux native AIO
2020-05-27  6:21:03 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2020-05-27  6:21:03 0 [Note] InnoDB: Uses event mutexes
2020-05-27  6:21:03 0 [Note] InnoDB: Compressed tables use zlib 1.2.8
2020-05-27  6:21:03 0 [Note] InnoDB: Number of pools: 1
2020-05-27  6:21:03 0 [Note] InnoDB: Using SSE2 crc32 instructions
2020-05-27  6:21:03 0 [Note] InnoDB: Initializing buffer pool, total size = 256M, instances = 1, chunk size = 128M

...

2020-05-27  6:21:08 0 [Note] mysqld: ready for connections.
Version: '10.3.7-MariaDB-1:10.3.7+maria~jessie'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
```

If you re-read the command above you will notify that we used the arguments `-it` (interactive/terminal). And you might have also found out that mariadb does not react to the usual `CTRL-c`.
So how do we exit this terminal? Docker has an escape sequence to detach from a container and leave it running. For this you have to press `CTRL-p` and then `CTRL-q` in bash. In the webshell the shortcuts `CTRL-p` and `CTRL-q` are not working. Simply close the terminal and open a new one as a workaround.

This will leave the container running while you are back in your shell. To verify that the container is running use the following command:

```bash
docker ps
```

The output should look much like this:

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
7cb31f821233        mariadb             "docker-entrypoint..."   5 minutes ago       Up 5 minutes        3306/tcp            upbeat_blackwell
```

## Access the container

To connect to the container again you can use the following command:

```bash
docker exec -it <container> bash
```

Where `<container>` can refer to the `CONTAINER ID` (the first two characters are normally sufficient) or one of the `NAMES` from the output of `docker ps`.
In the above output this would be `7cb31f821233` or `upbeat_blackwell`.

{{% alert title="Note" color="primary" %}}
The docker exec command needs either the ID or NAME of the container. Additionally, at the end, an executable.
{{% /alert %}}

In this example, it's `bash` as we want to do something interactively in the container.

Once the command is executed you should see this:

`root@7cb31f821233:/#`

{{% alert title="Note" color="primary" %}}
Every time you connect yourself to a container you will always be the user that was defined in the Dockerfile.
{{% /alert %}}

Now that we are connected, let's find out if the MariaDB is working...

```bash
mariadb -uroot -pmy-secret-pw
```

If everything works as expected, you should now see the MariaDB command line:

```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.3.8-MariaDB-1:10.3.8+maria~jessie mariadb.org binary distribution

Copyright (c) 2000, 2020, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Type

```bash
exit;
```

to leave the mysql client. Type

```bash
exit
```

one more time to leave the container.

## Detached containers

One might think: _This whole starting process is a bit cumbersome with `CTRL-p` and then `CTRL-q`_.
Therefore, you can run a Docker container directly with `-d` (detached) mode, e.g.:

```bash
docker run -it -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb
```

Instead of the output of the container itself, you will now only get the ID of the started container.
If you have a look into the container list, you should see two running containers:

```bash
docker ps
```

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
699e82ed8f1f        mariadb             "docker-entrypoint..."   3 minutes ago       Up 3 minutes        3306/tcp            jolly_bardeen
7cb31f821233        mariadb             "docker-entrypoint..."   32 minutes ago      Up 32 minutes       3306/tcp            upbeat_blackwell
```

We don't need both of them running. To stop a container use the command:

```bash
docker stop <container>
```

After that, check the new state with

```bash
docker ps
```

This should show only one container running:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
699e82ed8f1f        mariadb             "docker-entrypoint..."   9 minutes ago       Up 9 minutes        3306/tcp            jolly_bardeen
```

We just exited the container "gracefully", but as an alternative you can also kill a container with the `docker kill <container>` command. This stops the container immediately by using the KILL signal.

You may recognize that the container _upbeat_blackwell_ is not present in the container list anymore. That is because `docker ps` only shows running containers, but as always you have a parameter that helps:

```bash
docker ps --all
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                         PORTS               NAMES
699e82ed8f1f        mariadb             "docker-entrypoint..."   12 minutes ago      Up 12 minutes                  3306/tcp            jolly_bardeen
7cb31f821233        mariadb             "docker-entrypoint..."   41 minutes ago      Exited (0) 2 minutes ago                           upbeat_blackwell
67d79f95c712        hello-world         "/hello"                 About an hour ago   Exited (0) About an hour ago                       upbeat_boyd
```

Now that the _upbeat_blackwell_ container is stopped delete it:

```bash
docker rm <container>
```

Now the container has disappeared from the list:

```bash
docker ps --all
```

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                         PORTS               NAMES
699e82ed8f1f        mariadb             "docker-entrypoint..."   13 minutes ago      Up 13 minutes                  3306/tcp            jolly_bardeen
67d79f95c712        hello-world         "/hello"                 About an hour ago   Exited (0) About an hour ago                       upbeat_boyd
```

It is a good idea to delete unused containers to save disk space and remove the data wich resides in the read/write layer of the container. So please delete the hello-word container too:

```bash
docker rm <container>
```

## Mounting a volume in a container

{{% details title="🤔 I have a container with a database server running. What happens to my data when I remove the container?" %}}
It's gone. The docker instance has no persistence layer to store data permanently, let us address that problem in this chapter.
{{% /details %}}

The MariaDB container is a good example as to why it's good to have an external volume.
There are several possibilities on how to work with volumes in Docker, in this case, we're going to create a docker volume to store the persistent data of our MariaDB. The volume is managed by Docker itself.

Create the docker-managed volume and attach it to a path in the container:

```bash
docker volume create volume-mariadb
docker run --name mariadb-container-with-external-volume -v volume-mariadb:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb
```

In case you are wondering where your data ends up you can inspect the volume with:

```bash
docker volume inspect volume-mariadb
```

Okay, now create a new user in the MariaDB container:

```bash
docker exec -it mariadb-container-with-external-volume mariadb -uroot -pmy-secret-pw
```

Inside the mariadb-client execute some SQL commands to grant user peter read permissions to the user table:

```bash
use mysql
CREATE USER 'peter'@'%' IDENTIFIED BY 'venkman';
GRANT SELECT ON mysql.user TO 'peter'@'%';
```

Once all steps are completed quit the mysql session which also will exit the container:

```bash
exit
```

We have added a new user to our DB and persistet the data through the use of volumes!
