---
title: "Docker image for non-blocking MySQL and MariaDB backups"
date: "2020-03-02"
slug: docker-image-mysql-mariadb-backups
tags: [docker,databases,backup]
hero: "fire.jpg"
summary: "Do you need a non-blocking docker based backup solution for your MySQL or MariaDB container? Here it is, a simple to configure docker image which creates (incremental) backups of your database and rotates them automatically."
---

Do you need a non-blocking docker based backup solution for your MySQL or MariaDB container? Here it is, a simple to configure docker image which creates (incremental) backups of your database and rotates them automatically. https://github.com/woolfg/mysql-backup-sidecar

The listing below shows a basic setup which creates a backup of your MariaDB every night at 3:05 AM using a docker-compose file. The backups are incremental and by default, every Sunday, a full backup is created. The backups are rotated automatically according to a predefined schedule which can be configured (see later in this blog post).

```
version: "3.7"
services:
  db:
    image: mariadb:10.4
    volumes:
      - mysqldata:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secure

  dbbackup:
    image: woolfg/mysql-backup-sidecar:mariadb-10.4
    volumes:
      - mysqldata:/var/lib/mysql
      - backup:/backup
    environment:
      CRON_SCHEDULE: "5 3 * * *"
      BACKUP_DIR: /backup
      INCREMENTAL: "true"
      MYSQL_USER: root
      MYSQL_PASSWORD: secure
      MYSQL_HOST: db
    links:
      - db  
```


# Why did I create this backup sidecar image?

I have several very small *LAMP* based websites which I recently migrated to a docker based setup. To create a high separation, every single website has its own web server and database container. To have a safe setup, I needed a simple but flexible backup solution for MySQL and MariaDB which works without changing the database container.

Therefore, I created this non-blocking backup sidecar container, which can backup the database container without touching the actual DB container or interfering with the running database itself.

For me, it is also very important that I can use the backup solution for high traffic websites which should not be blocked by a backup process (happens e.g. when using `mysqldump`). Therefore, I decided to use [XtraBackup](https://www.percona.com/doc/percona-xtrabackup) for MySQL and the fork of it, [MariaDBbackup](https://mariadb.com/kb/en/mariabackup-overview/) for MariaDB.

# It should be configurable!

My requirement was to be able to configure the backup container using environment variables without changing the image itself. This simplifies updates, maintenance in general, and disk usage of the image repository I use. I can simply use the same image for a lot of database containers. The following list gives an overview of all variables which can be used to configure the behavior of the backup sidecar.

## Basic setup

| Variable | Default | Description  |
|---|---|---|
| `CRON_SCHEDULE` | `5 5 * * *` | Schedule of backups as [cron expression](https://en.wikipedia.org/wiki/Cron#CRON_expression) |
| `BACKUP_DIR` | `/backup` | Directory in the container which is used for backups |
| `DIR_DATE_PATTERN` | `%Y%m%d` | [Date pattern](http://man7.org/linux/man-pages/man1/date.1.html) which is used for naming a backup |
| `COMPRESS_THREADS` | `0` | if >0 backups are compressed using the defined number of threads |
| `MYSQL_HOST` | `db` | Hostname/IP of the database server |
| `MYSQL_PORT` | `3306`| Port of the database server to connect to |
| `MYSQL_USER` | `root`| Username of the database backupuser |
| `MYSQL_PASSWORD` | - | Password for the database backupuser |
| `MYSQL_PASSWORD_FILE` | - | Path to a password file that contains the password in plaintext. It can be used with docker secrets. The password variable is overwritten by the file content.|

## Incremental backup setup

By default, incremental backups are activated and a full backup is created every Sunday. The condition for full backups can be defined by two variables. The left side contains the pattern for the Linux `date` command that is executed for the condition. If the date matches the result (`Sun` in the example), a full backup is created.

| Variable | Default | Description  |
|---|---|---|
| `INCREMENTAL` | `"true"` | defines if incremental backups should be created |
| `FULL_BACKUP_DATE_FORMAT` | `%a` | [Date pattern](http://man7.org/linux/man-pages/man1/date.1.html) which is compared to the content of the following variable |
| `FULL_BACKUP_DATE_RESULT` | `Sun` | If this string matched with the date string, a full backup is created. The default values result in a full backup on Sunday |

## Rotation of backups

To save disk space, backups should be rotated automatically. The default pattern is:
- every day an incremental backup
- every Sunday a full backup
- keep weekly backups for a month
- keep monthly backups for a year
- keep yearly backups after one year

The behavior is controlled using the following variables which define three rotation cycles. Every cycle is defined by a time range of days and a condition based on the Linux `date` command. For example, defines the first cycle that backups older than `6` days should only be kept if the backup was done on a Sunday.

| Variable | Default | Description  |
|---|---|---|
| `ROTATION1_DAYS` | `6` | condition for backups older than specified number of days |
| `ROTATION1_DATE_FORMAT` | `%a` | linux date pattern of the condition |
| `ROTATION1_DATE_RESULT` | `Sun` | condition if backup is kept |
| `ROTATION2_DAYS` | `30` |  condition for backups older than specified number of days |
| `ROTATION2_DATE_FORMAT` | `%d` | linux date pattern of the condition |
| `ROTATION2_DATE_RESULT` | `<8` |  condition if backup is kept |
| `ROTATION3_DAYS` | `365` | condition for backups older than specified number of days |
| `ROTATION3_DATE_FORMAT` | `%m` | linux date pattern of the condition |
| `ROTATION3_DATE_RESULT` | `01` | condition if backup is kept |

# Feedback

The image is in early *Alpha* and I am already started to roll it out. So far, everything worked as expected. I hope you give it a try or give some feedback. I am also happy to receive pull requests or bug reports.

- [Github-Repo](https://github.com/woolfg/mysql-backup-sidecar)
- [Docker Image](https://hub.docker.com/repository/docker/woolfg/mysql-backup-sidecar)

# Credits
Thanks to [Tim Hannemann](https://twitter.com/thevictim02) and [Matthias Endler](https://twitter.com/matthiasendler) for reviewing drafts of this article.