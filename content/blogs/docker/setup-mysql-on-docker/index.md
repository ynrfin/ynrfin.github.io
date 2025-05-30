---
title: setup docker for MySQL
date: 2024-09-26
tags: [ linux, docker, mysql ]
draft: false
---

# Overview
This is how I setup MySQL in docker for development so  I can just copy and paste this.

# create volume 
create volume for saving data, by creating volume, it will make sure that tis volume is readable for our project. else it will use the parent directory name where the `docker-compose.yml` is located. If you use volume from inside MySQL container, the data will be lost when the container stops.

```
docker volume create mysql_data
```

# Create .env file for var names

`.env`

```env
DB_DATABASE="project_name"
DB_USERNAME="admin"
DB_PASSWORD="admin"
```

**PS**: `root` username is prohibited by `phpmyadmin`. If you do not use phpmyadmin, go ahead

the `docker-compose.yml`

```yaml
version: "3"
services:
  db:
    # this is for Mac ARM
    platform: linux/x86_64
    image: mysql:5.7
    ports:
      # host:container
      - "3310:3306"
    restart: always
    environment:
      MYSQL_DATABASE: ${DB_DATABASE1}
      MYSQL_USER: ${DB_USERNAME}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
    command: --sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    platform: linux/x86_64
    image: phpmyadmin/phpmyadmin
    restart: always
    ports:
      # host:container
      - "8080:80"
    environment:
      # db refer to mysql db service name under `services:`
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - db

volumes:
  db_data: mysql_data
```

# Run the file

run by using
```
docker compose run -d
```

you can add `--build` flag to make sure `docker compose` rebuild your containers instead of running already available containers. MySQL data will not be cleaned when using this flag because we have use external volume, not volume that is inside `db` service.

# Accessing MySQL

username : `admin`

password : `admin`

Access via:
- phpmyadmin at `http://localhost:8080`
- any database client at host `localhost` or `127.0.0.1` and port `3310`
- your docker application at host `db` the service name

Using the username and password above


# Trivias

## Change volume when you change the image

If you want to change the image, make sure to change the volume, as it will corrupt the current volume.

## Sourcing big sql file

Use built in `source` command from MySQL.

connect to MySQL container using cli command in the directory where the `.sql` file is located.

```
mysql -h 127.0.0.1 -P 3310 -u admin -p

```

set the database:
```
USE <database_name>;
```


source the SQL file:
```
source <sql_filename>.sql;
```
