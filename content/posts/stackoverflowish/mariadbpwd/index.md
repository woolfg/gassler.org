---
title: "Reset Root Password in MariaDB + MySQL in Docker and Docker Swarm"
date: "2020-04-09"
slug: reset-password-mariadb-mysql-docker
tags: [stackoverflowish,mysql,mariadb,databases,docker]
hero: "steve-johnson-e4Davs7-XnE-unsplash.jpg"
summary: "Note to myself: do not delete your docker swarm cluster when using docker secrets for your docker based databases ;) - In case you do, you have to reset the passwords in the docker databases."
---

### Note to myself: do not delete your docker swarm cluster when using docker secrets for your docker based databases ;)

In case you do, you have to reset the passwords in the docker databases.

Change your docker compose file and add the argument `--skip-grant-tables` which can be done by using `command` in your docker compose file. If you haven’t used command in your docker compose, just add the line below or just add the argument if you already use `command`.

```
version: "3.7"
  db:
    image: mariadb
    volumes:
      - mysqldata:/var/lib/mysql
    environment:
      MYSQL_DATABASE: mydb
      MYSQL_USER: myuser
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
      MYSQL_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_root_password
      - db_password
    command: --skip-grant-tables
```

After this change restart your docker container and login to it. 

```
$ sudo docker ps | grep project_db
29b86cab6d42 mariadb:10 ...

$ sudo docker exec -it 29b86cab6d42 bash
root@29b86cab6d42:/# mysql -u root
```

Don’t forget to flush privileges otherwise you will get the following errors while trying to change the password:

```
MariaDB [mysql]> update user SET PASSWORD=PASSWORD("newpassword") WHERE USER='root';
ERROR 1348 (HY000): Column 'Password' is not updatable
MariaDB [mysql]> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';
ERROR 1290 (HY000): The MariaDB server is running with the --skip-grant-tables option so it cannot execute this statement
```

So flush privileges and alter the root password. Usually you have two different root users, one for localhost and one for everything else, so don’t forget to alter the second one.

```
MariaDB [mysql]> flush privileges;
Query OK, 0 rows affected (0.002 sec)

MariaDB [mysql]>  ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';
Query OK, 0 rows affected (0.005 sec)
MariaDB [mysql]>  ALTER USER 'root'@'%'  IDENTIFIED BY 'newpassword';
Query OK, 0 rows affected (0.007 sec)
```

In case you have specified a `MYSQL_USER` in your docker compose file, also change the password of this user. In case you have already generated new docker secrets, just use the password specified in the files in `/run/secrets`.

The last step is to remove the `--skip-grant-tables` in your docker compose file and restart the container using docker compose.
