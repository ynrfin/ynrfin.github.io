---
author: "Yanuar Arifin"
title: "Dockerize Laravel Application Using Laradock"
date: "2021-06-20"
description: "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
tags: [
    "php",
    "laravel",
    "docker",
    "container",
    "apache",
    "mysql",
]
draft: false
---

Create Docker containers for Laravel, MySQL, and PhpMyAdmin without fiddling with Dockerfile and docker-compose.yml.
<!--more-->

## Why Laradock

- it is faster compared to creating your own dockerfile & docker-compose.yml
- no need to know about how docker works. You just edit some files

## Prequisite

- you know docker concept
- you have docker and docker-compose installed already
- this tutorial is tested on linux


## Create Laravel Project or Use Already Made Project

From laravel documentation page:

```bash
composer create-project laravel/laravel example-app
cd example-app
```

## Add Laradock

To add laradock is simple. Clone it inside your laravel project

Because we already go inside the app, next step is clone laradock

```
git clone https://github.com/Laradock/laradock.git
```

Wait until its done cloning. Then your directory structure should be like this

```
|- example-app
  |- app
  |- artisan
  |- bootstrap
  |- composer.json
  ...
  |- laradock        <-- this is the laradock directory
  ...
```

# Set Up

### 1. Configure Laradock Via .env

This .env file is the configuration of laradock. What we will configure here:

- which php version should we use
- which MySQL version should we use
- MySQL username, password, & port
- where volume data will be stored at
- container prefix. Laradock would create a few containers, a prefix would differentiate between different project container

copy `env-example` file inside laradock to `.env`

```
cp env-example .env
```

Change this values inside the `.env`

```env
# Which directory to store volume data
DATA_PATH_HOST = ~/.laradock/data/example-app

# Prefix for the containers
COMPOSE_PROJECT_NAME = example-app

# PHP version
PHP_VERSION = 7.3

# MySQL setting & credential
MYSQL_VERSION=5.7
MYSQL_DATABASE=example-app
MYSQL_USER=default
MYSQL_PASSWORD=secret
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD=root
```

That is it for this file.

### 2. Use Custom Domain To Access Our App

We wil make it so that when we access `example.local` from our browser, it will redirect
to our laravel application using Apache2.

Edit `app\laradock\apache2\sites\default.apache.conf`. The file already exists

```apacheconf
...
ServerName example.local
DocumentRoot /var/www/public/
...
<Directory "/var/www/public/">
...

```

- `ServerName example.local`. Pass request to our app when theres request to `example.local`
- `DocumentRoot /var/www/public`. Specify where the content is located. `/var/www` is the default apache location for application directory. Laradock copy our `app` directory here when creating container. `/public` dir have `index.php` is the entry point for Laravel application.

**NOTE**: When changing config, always build the container again (`docker-compose build apache2`)

### 3. Build Docker Containers

The configuration is ready, now we could create docker containers for our application.

Run this command from `laradock` directory.

```bash
docker-compose up -d phpmyadmin mysql apache2
```

This command will build container for Apache2, MySQL, and PhpMyAdmin (or skip it if already exists). This is the output

```
Starting example-app_mysql_1            ... done
Starting example-app_docker-in-docker_1 ... done
Starting example-app_workspace_1        ... done
Starting example-app_phpmyadmin_1       ... done
Starting example-app_php-fpm_1          ... done
Starting example-app_apache2_1          ... done
```


**Always** specify which container you want to build/run, if not laradock will build all available images on laradock


### 4. Redirect Request to example.local To Our Application

The application is ready to serve our requests. But if you access `example.local`, it will try to search for `example.local` from internet instead of redirecting to our app. To tell our computer to look within itself when there is a request to `example.local`, we need to add entry to `/etc/hosts` file and add this at the bottom

```
...
127.0.0.1     example.local
...
```

Save the file and try access the domain again. You should be served with your application

## Using Artisan

To use app command line utilities, use it inside your container, NOT on your host computer.

To go to your application container use this command:
```bash
docker-compose exec workspace bash
```

You will be directed to the container at `/var/www` directory. Right where your app is. Like this

```
root@e12a9c08fc3a:/var/www#
```

Then you could use `artisan` command there.

To exit just type `exit` on the terminal and enter.


# Debugging

## Could not access the server

### Check if container is created & being run

If you could not access the domain at all, first check if docker-compose could create the containers. The container might stop after successfully created. Use this command

```bash
docker container ls
```

Then check if these containers exist
- `example-app_mysql_1`
- `example-app_docker-in-docker_1`
- `example-app_workspace_1`
- `example-app_phpmyadmin_1`
- `example-app_php-fpm_1`
- `example-app_apache2_1`

### Check server access logs

To check if the application server accepting your request, use this command:

```
docker logs example-app_apache2_1
```

This command is used to print Apache access logs. Look for HTTP verb like `GET`, or `POST` depending on your access like this one

```
example.local:80 172.19.0.1 - - [29/Jun/2021:17:08:03 +0000] "GET / HTTP/1.1" 200 3875 "http://example.local" "Mozilla/5.0 (X11; Linux x86_64; rv:89.0) Gecko/20100101 Firefox/89.0"
```

If you do not see any of it, it means you request haven't arrived at the docker application. Make sure `ServerName` in `laradock/apache2/sites/default.apache.conf` equals to the redirect in `/etc/hosts` file that you entered before.


# Lesson When Using Laradock

- `DATA_PATH_HOST`. Always change this to specific location for each laradock. My MySQL data got corrupted because I did not change this path for 2 different MySQL version
- `COMPOSE_PROJECT_NAME`. change this to differentiate containers between many laradock installation
- `MYSQL_VERSION`. Use 5.7 or lower. Latest MySQL for docker has bugs per this writing

# Caveats

- **Ports**; using this setup, there are ports that is being used by laradock. Make sure those ports is free or the container creation would fail. And this port problem leads to
- **One app at a time.** When you run this, there will be only one running laradock because of port usage problem above
