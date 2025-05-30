---
title: Use Golang Migrate on Docker Compose
date: 2024-05-25
tags: [docker, golang, migration, database, mysql, postgreSQL]
---

Previously I setup docker compose for golang application and PostgreSQL. It can run the application and can connect to PostgreSQL. While doing basic CRUD, I found that I need a tool to migrate my database structure and seed the database for easy onboarding and development. Thus I want to use `golang-migrate` to do that. 

# What Is Golang Migrate
Contrary to the name, it is not a migration tool specifically  created for golang development, although we can use it as a golang package in your application. This tool is a CLI tool that can be installed on Windows, Mac, and Linux. As a CLI tool, it means that no matter what language you use to code, your migration can be managed using this `golang-migrate`.

[Here](https://github.com/golang-migrate/migrate/blob/master/cmd/migrate/README.md) is the documentation how to install it.

# Golang migrate commands

What I use is this for now. 

Create migration script:
```bash
migrate create -ext sql -dir migrations -seq create_users_table
```

- `migrate` golang-migrate command

- `create` create new migration script, both up and down

- `-ext sql` use `sql` extension

- `-dir migrations` the target directory where the migration files is generated

- `-seq` make the file name sequential, increment from 1. if not it would be current datetime

- `create_users_table` migration files name

which will create 2 files in `migrations` dir
```diff
    ├── Dockerfile
    ├── Dockerfile.multistage
    ├── README.md
    ├── cmd
    │   └── main.go
    ├── controllers
    ├── docker-compose.yml
    ├── go-market-warehouse-api
    ├── go.mod
    ├── go.sum
    ├── main
    ├── middlewares
    ├── migrations
+++ │   ├── 000001_create_users_table.down.sql <- this
+++ │   └── 000001_create_users_table.up.sql   <- and this
    ├── models
    └── repositories
```

`000001_create_users_table.up.sql` contains the DDL for the changes that I want to make, in this case creating `users` table

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE TABLE users (
	id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	name VARCHAR NOT NULL,
	email VARCHAR(255) NOT NULL UNIQUE
);
```

and `000001_create_users_table.down.sql` contains the DDL to undo the above code.

```sql
DROP TABLE IF EXISTS "users";
```

And Apply the migration once the DDL is supplied in the files:
```
migrate -path /migrations/ -database "postgres://username:password@host:port/db_name?sslmode=disable" up
```

Here's the database tables before we execute the migration:
![Table list before migration](db-before-migration.png)

And here it is after running migration
![Table list after migration](db-after-migrtion.png)

- `schema_migrations` is used by golang migrate to track migrations
- `users` table is the table that being generated from my script above

# How To Use Golang Migrate in Docker Compose

Golang migrate has its own image in docker hub. The way to use this, is to create a new service for this golang migrate, then run the migration that targetted to PostgreSQL service. By creating it's own service and execute it, the migration is another service that is not added to the main application service, make the application smaller.

Here's the new `golang-migrate` service:
```yml
service:
  # ... go-market-warehouse-api & local-pg-16 services
  migrate:
    image: migrate/migrate
    depends_on:
      local-pg-16:
        condition: service_healthy
    networks:
      - my-local-net 
    volumes:
      - ./migrations/:/migrations
    command: ["-path", "/migrations/", "-database", "postgres://${PGUSER}:${PGPASSWORD}@local-pg-16:5432/postgres?sslmode=disable", "up"]
```

Full code [in this commit](https://github.com/ynrfin/go-market-warehouse-api/tree/83ed175ea03b75cad75f36f63c7295ef8f33352f)

`image: migrate/migrate` base this service to the image of `golang-migrate`

`depends_on: local-pg-16: condition: service_healthy`  I rearrange the application startup order to `PostgreSQL`  then `golang-migrate` then `go-market-warehouse-api` because after we startup the database, I want the database to be updated using latest DDL then the application can connect to database.

`networks` use already declared network, which is `my-local-net` to connect to PostgreSQL service

`volumes` here I specify which directory contains the migration scripts, in my case, it is the `migrations` directory, the LEFT one `./migrations/`.

`commands` is the arguments the `migrate` service use to run the migration

The `commands` is the bash command to apply the migration(s) like the one above without using docker.

```bash
migrate -path /migrations/ -database "postgres://${PGUSER}:${PGPASSWORD}@local-pg-16:5432/postgres?sslmode=disable" up
```

This is the docker log with migration executed

![Docker log for migrate execution](migrate-execution.png)

`migrate-1` is the service name
`1/u create_users_table` 1 migration upped, with name of create_users_table

If no new migration found, there won't be message on the log.

That is it for this article, hope you enjoy it.
