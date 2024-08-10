# docker-compose-moodle
[![Build Status](https://travis-ci.org/jobcespedes/docker-compose-moodle.svg?branch=master)](https://travis-ci.org/jobcespedes/docker-compose-moodle)
![Moodle](https://img.shields.io/badge/Moodle-3.5-blue.svg?colorB=f98012)
![Apache2](https://img.shields.io/badge/Apache2-2.4-blue.svg?colorB=557697)
![PHP](https://img.shields.io/badge/PHP-7-blue.svg?colorB=8892BF)
![Postgres](https://img.shields.io/badge/Postgres-9.6-blue.svg?colorB=0085B0)
[![Buy me a coffee](https://img.shields.io/badge/$-BuyMeACoffee-blue.svg)](https://www.buymeacoffee.com/jobcespedes)
[![Software License](https://img.shields.io/badge/License-APACHE-black.svg?style=flat-square&colorB=585ac2)](LICENSE)

>Leer en [**Español**](#español)

This project quickly builds a local workspace for Moodle  (Apache2, PHP-FPM with XDEBUG y Postgres) using containers for each of its main components. The local workspace is built and managed by Docker Compose

## Quickstart:
1. Install Docker. Check out how to install [Docker](https://docs.docker.com/install/)
2. Install Docker Compose. Check out how to install [Docker Compose](https://docs.docker.com/compose/install/)
3. Download this repo: `git clone https://github.com/Rennesz/moodle-docker-compose.git && cd moodle-docker-compose`
4. Clone Moodle repo: `git clone --branch MOODLE_401_STABLE --depth 1 https://github.com/moodle/moodle html`
5. Change variables to your need
6. Run with: `docker-compose up -d`

## Contents
1. [Environment Variables](#Environment-variables)
2. [Docker Compose Resources](#Docker-Compose-resources)
3. [Workspace Operations](#Project-management-with-Docker-Compose)
4. [Debugging with XDEBUG](#XDEBUG)
5. [Moodle Cron Debugging](#Cron-debugging)
6. [Database management with Pgadmin4](#Pgadmin4)
7. [Backup and restore database](#Backup-and-restore-database)
8. [Install Docker](https://docs.docker.com/install/)
9. [Install Docker Compose](https://docs.docker.com/compose/install/)

## Environment variables
The following table describes environment variables set in [**.env**](.env). The defaults work for a initial setup. They can be modified if needed.

| Variable              | Default value      | Use                                                                      |
| :-------------------- | :----------------- | :----------------------------------------------------------------------- |
| **REPO_FOLDER**       | html               | Default relative path for Moodle repo                                    |
| **DOCUMENT_ROOT**     | /var/www/html      | Mount point inside containers for volume **REPO_FOLDER**                 |
| **MY_TZ**             | America/Costa_Rica | Containers timezone                                                      |
| **PG_LOCALE**         | es_CR              | Containers locale                                                        |
| **PG_PORT**           | 5432               | Postregres port to expose                                                |
| **POSTGRES_DB**       | moodle             | Postgres DB for Moodle                                                   |
| **POSTGRES_USER**     | user               | DB user for Moodle                                                       |
| **POSTGRES_PASSWORD** | password           | DB password for Moodle                                                   |
| **PHP_SOCKET**        | 9000               | PHP-FPM socket to connect apache2  and php-fpm services                  |
| **ALIAS_DOMAIN**      | localhost          | Domain Alias                                                             |
| **WWW_PORT**          | 80                 | Web port to be bound                                                     |
| **MOODLE_DATA**       | /var/moodledata    | Mount point inside containers for Moodle data folder                     |
| **WWWROOT**           | localhost          | Host part to set in Moodle file 'config.php' for config option 'wwwroot' |

## Docker Compose resources
The following table sums up Docker Compose resources.

| Component         | Type      | Responsability          | Content                      | Config                                                                                                                                                                                                 |
| :---------------- | :-------- | :---------------------- | :--------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **apache2**       | Container | Web server              | Debian9, Apache2             | [Apache2](http://dockerfile.readthedocs.io/en/latest/content/DockerImages/dockerfiles/php-apache.html#web-environment-variables) server and [modules for Moodle](https://docs.moodle.org/35/en/Apache) |
| **cron**          | Container | Cron task for Moodle    | Debian9, Cron                | [Moodle cron task](https://docs.moodle.org/35/en/Cron) and its frequency                                                                                                                               |
| **php-fpm**       | Container | Process manager for PHP | Debian9, PHP-FPM, XDEBUG     | PHP, its modules and Moodle dependencies                                                                                                                                                               |
| **postgres**      | Container | DBMS                    | Debian9, Postgres            | [User and DB](https://hub.docker.com/_/postgres/)                                                                                                                                                      |
| **db_dumps**      | Volume    | Restore db when built   | Dump files for DB to restore | To restore an initial database if you start the container with a data directory that is empty. File name should be "dump-init.dump"                                                                    |
| **moodledata**    | Volume    | Moodle data store       | Data generated by moodle     | [Moodle data dir ](https://docs.moodle.org/35/en/Installing_Moodle#Create_the_.28moodledata.29_data_directory)                                                                                         |
| ***REPO_FOLDER*** | Volume    | Moodle code             | Moodle code                  | It is set to 'html/' by deafult (check out [**.env**](.env))                                                                                                                                           |

## Project management with Docker Compose
> **Inside project folder**
1. Run project
``` bash
docker-compose up -d
# Different project name
# docker-compose -p my-proj up -d
```
2. Stop project
``` bash
docker-compose stop
# docker-compose stop php-fpm
```
3. Start project
``` bash
docker-compose start
# docker-compose start php-fpm
```
4. Remove project
``` bash
docker-compose down
# Remove volumes too
# docker-compose --volumes
# With different project name:
# docker-compose -p my-proj down
```
5. Logs
``` bash
docker-compose logs
# docker-compose logs -f --tail="20" php-fpm
```





## Pgadmin4
Config pgadmin4
1. Go to http://localhost:5050
2. In ```File -> Preferences -> Binary paths``` set ```/usr/bin```
3. Add new server:
    * Tab ```General```
        * Name: Any name you want
    * Tab ```Connection```
        * Host name/address: ```postgres```
        * Host Username: ```user```
        * Host Password: ```password```
    * Save

## Backup and restore database
When needed, backup and restore of the database could be done in the following way.
```bash
# Set env vars
POSTGRES_USER=user
POSTGRES_DB=moodle
DB_DUMP_NAME=dump-init.$(date +"%Y%m%d%H%M%S").dump

# Backup
# -Fc  Output a custom-format archive suitable for input into pg_restore
docker-compose exec postgres pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} -Fc -f /opt/db_dumps/${DB_DUMP_NAME}

# Restore
# -c  Clean (drop) database objects before recreating them.
# -C  Create the database before restoring into it
docker-compose exec postgres pg_restore -U ${POSTGRES_USER} -d postgres -c -C -O --role ${POSTGRES_USER} /opt/db_dumps/${DB_DUMP_NAME}
```

A database can be automatically restored when postgres service starts. By placing a dump file inside 'db_dumps' folder and naming it "dump-init.dump", postgres container will try to restore that file as an initial database if data directory is empty.
> **IMPORTANT**: Depending of  size, database initial availability could be delayed

