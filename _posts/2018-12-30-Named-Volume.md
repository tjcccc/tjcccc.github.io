---
layout: post
title: Use Named Volume For Persistent Data (MySQL)
key: 20181239
tags: Docker Volume Data MySQL
---
# Use Named Volume For Persistent Data (MySQL)

In this log, I record the steps of making named volume for persistent data and make some necessary notes about it.

I use MySQL as database in the case.

`{ }` wraps the custom variable.

## Create a Named Volume

```shell
docker volume create --name {volume_name}
```

<!--more-->

## Create a Defined Network for Combining Containers

```shell
docker network create {network_name}
```

## Create a MySQL Container

Create and connect to the specified network you've just created:

```shell
docker run -d --rm --name {mysql_container_name} -v {volume_name}:/var/lib/mysql -p {new_mysql_port}:3306 -e MYSQL_ROOT_PASSWORD={your_mysql_root_password} --network {network_name} mysql:tag
```

- MySQL data directory: `/var/lib/mysql`
- `--daemon, -d`: Run container in background and print container ID.
- `--rm`: Automatically remove the container when it exits.
- `--volume, -v`: To use volume.
- `--publish, -p`: Map port for MySQL. The default is 3306. You should specify another one for multiple MySQL containers.
- `--env, -e`: Set the environment (root password) for MySQL: `-e MYSQL_ROOT_PASSWORD={password}`

## Create a PhpMyAdmin Container for MySQL Container

I use a PhpMyAdmin container as the front-end database management of the MySQL container. So we need combine these two containers.

There're two methods:

### Link (A Legacy Feature)

```shell
docker run --rm --name {phpmyadmin_container_name} --link {mysql_contain_name}:{alias_name} -p {new_phpmyadmin_port}:80 phpmyadmin/phpmyadmin
```

- `--link`: Add link to MySQL container.

This feature is deprecated by Docker. Don't use it.

### User Defined Network

Usage:

```shell
docker network connect {network} {container}
```

Create and connect to the specified network:

```shell
docker run -d --rm --name {phpmyadmin_container_name} --network {network_name} -p {new_phpmyadmin_port}:80 -e PMA_HOST={mysql_container_name} phpmyadmin/phpmyadmin
```

- `--network, --net`: Connect a container to a network.

### Use Your PhpMyAdmin

Open browser, visit `http://localhost:{new_phpmyadmin_port}`.

Login with username `root` and password `{your_mysql_root_password}`.

You might get the login error:

> The server requested authentication method unknown to the client.

It is because MySQL uses the `auth_socket` (>= v5.7) or `caching_sha2_password` (>= 8.0) plugin as the authentication method for `root` user. For logining the PhpMyAdmin, you should change the password with `mysql_native_password` plugin or create a new user.

## Use MySQL to Deal With Admin Account

Enter MySQL container's bash:

```shell
docker exec -it {mysql_container_name} bash
```

Enter MySQL cli:

```shell
mysql -u root -p{password}
```

You can change `root`'s password:

~~~sql
alter user 'root'@'localhost' identified with mysql_native_password BY {'password'};
flush privileges;
~~~

Or create a new admin account:

~~~sql
create user {'username'@'host'} identified with mysql_native_password by {'password'};
grant all privileges on {database} to {'username'@'server'};
flush privileges;
~~~

- `{'username'@'host'}`: 'host' could be 'localhost' or '%'. '%' means the account can be accessed from any ip.
- `{database}`: `*.*` means all.

## Backup Named Volume

First, create a new container (based on a tiny image like busybox) with the data volume mounted. The data `var/lib/mysql` as the volume mapped data will be mounted to the new container. Then we will tar it in the new container for backup. For getting the tar file in our local host, we mount a local directory as another volume for the new container's data `backup`.

```shell
docker run --rm --volumes-from {mysql_container_name} -v $(pwd):/backup busybox tar cvf /backup/{backup_file_name}.tar /var/lib/mysql
```

- `--volumes-from`: Mount volumes from the specified container(s).
- `$(pwd)`: Print the name of the current Working Directory.

## Restore container from backup

Mount data volume and backup directory to a temporary container. Then, untar the backup file to replace files in the data volume:

```shell
docker run --rm --volumes-from {mysql_container_name} -v $(pwd):/backup centos bash -c "cd /var/lib && rm -rf mysql/* && tar xvf /backup/{backup_file_name}.tar --strip 2"
```

- For PowerShell, use `${pwd}` instead of `$(pwd)`.

Restart the MySQL container to make the restored data works:

```shell
docker restart {mysql_container_name}
```

## To be continued

I will keep finding an easier and smarter way.
