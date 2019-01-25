# Migrating My Localhost MongoDB Data to Docker Container

## Create MongoDB Container

```shell
docker run -d --rm --name {mongo_container_name} --network {my_network} -v {db_volume}:/data/db -p 27018:27017 -e MONGO_INITDB_ROOT_USERNAME={username} -e MONGO_INITDB_ROOT_PASSWORD={password} mongo
```

<!--more-->

## Backup MongoDB database

First, you need lock the database for safety.

```shell
// mongo cli
use admin
db.auth("{usename}", "password")
db.fsyncLock()
``` 

Probably you will be told:

> not authorized on admin to execute command

We should add the privileges (include `backup` and `restore`) to admin account by adding the [Built-in Roles](https://docs.mongodb.com/manual/core/security-built-in-roles/):

```shell
// mongo cli
db.grantRolesToUser("{username}", [ { "role": "hostManager", "db": "admin" }, { "role": "backup", "db": "admin" }, { "role": "restore", "db": "admin" } ])
```

The hostManager is the role for locking the database.

Now we can use `mongodump` to backup the database:

```shell
sudo mongodump -u {username} -p {password} --authenticationDatabase admin -h localhost -d {database} --port 27017
```

It will create a dump directory with database files in the your current location. After dumping, you can unlock the database:

```shell
// mongo cli
use admin
db.auth("{usename}", "password")
db.fsyncUnlock()
```

## Restore from Dump Files

The mongo in port 27017 is my localhost old database. Now I want to restore it to my new docker container of mongo which port is 27018:

```shell
sudo mongorestore -u {username} -p {password} --authenticationDatabase admin -h localhost -d {database} ./dump/{database_directory} --port 27018
```

The migration is done.
