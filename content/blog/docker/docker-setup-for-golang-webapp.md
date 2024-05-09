# Docker Setup for Golang Web App
This post will explain how I create Dockerfile and docker-compose.yml for golang development. The full code can be seen [here](https://github.com/ynrfin/go-market-warehouse-api/tree/b88913f0a2f01fd1a73d43bff7ea76cb127cdaf7)

This code for Dockerfile
```Dockerfile
FROM golang:1.22 

WORKDIR /app

COPY go.mod go.sum ./

RUN go mod download

COPY . ./

RUN CGO_ENABLED=0 GOOS=linux go build -o /go-market-warehouse-api ./cmd/main.go

EXPOSE 8000

CMD [ "/go-market-warehouse-api" ]

```

`FROM golang:1.22` is basing the  image to current newest golang version available

`WORKDIR /app`  tell docker to create and make this `/app` directory as the current directory the docker is in

`COPY go.mod go.sum ./` will copy `go.mod` and `go.sum` from project directory to `/app` directory inside docker 

`RUN go mod download` will download the dependency listed in go.mod that we just copied

`COPY . ./` will copy content in current directory (the project root dir ). Basically copying the project to docker
`RUN CGO_ENABLED=0 GOOS=linux go build -o /go-market-warehouse-api ./cmd/main.go` is building go binary to from `cmd/main.go` and give output to root dir `/` and use  `go-market-warehouse-api` as the binary name, same as `go-market-warehouse-api.exe` in windows

`EXPOSE 8000` tell the image it builds to open port 8000 to outside the image

`CMD [ "/go-market-warehouse-api" ]` will run the binary that we create

This setup `Dockerfile` will create an image that is around 1 Gb. Big but it contains necessary tools to build golang app. Suitable for development.

# building docker-compose.yml

## Application Service

The purpose for this `docker-compose.yml` is to run the golang application and a database in my application stack. My database choice is PostgreSQL.

Before we create the file, first I will create new docker `network` for the application and database to communicate. And a `volume` to persist data for the database, so when we restart the database, the data that we previously insert still exist.

Create  volume:
```
docker volume create pg-16
```

This will create a volume called `pg-16`. You can check it using `docker volume list`

Create network:
```
docker network create -d bridge my-local-net
```

This will create a new bridge network called `my-local-net`. Check with `docker network list`

The `bridge` part is a network that can be accessed inside the docker

Create the application service `docker-compose.yml` file:
```yml
services:
  go-market-warehouse-api:
    # This will be uncommented when db service is introduced
    # depends_on:
    #  local-pg-16:
    #    condition: service_healthy

    # always restart when app crashes
    restart: always
    build:
      context: .
    image: go-market-warehouse-api:v1.0
    container_name: go-market-warehouse-api
    hostname: go-market-warehouse-api
    networks:
      - my-local-net
    ports:
      - 80:8080
    environment:
      - PGUSER=${PGUSER:-totoro}
      - PGPASSWORD=${PGPASSWORD:?database password not set}
      - PGHOST=${PGHOST:-db}
      - PGPORT=${PGPORT:-5432}
      - PGDATABASE=${PGDATABASE:-mydb}
    deploy:
      restart_policy:
        condition: on-failure
```

`services` this mark the list of available services when we run docker compose. In this case, the services that is available only 1, called `go-market-warehouse-api`(the one under `services:`)

`build` and `context: .`: will build from current directory where docker-compose.yml is located(indicated by `.`)

`image` which image it will use to build. when not available locally, it will search from docker registry

`container_name` the name of  the container to be built

`hostname` how can this service be called by other service in the network. Docker container can refer to other services location(ip) using this `hostname`

`networks`: the network this service attached to

`ports`: exposed ports by this service

`environment` : setting environment variable

`deploy` and its child specs :  it will deploy the service and restart it when failure happens

## Database Service

This is the description of the database service:

```yml
  local-pg-16:
    image: postgres:16.2
    container_name: local-pg-16
    hostname: local-pg-16
    networks:
      - my-local-net
    ports:
      - 5432:5432
      - 8080:8080
    volumes:
      - tes-pg:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=${PGPASSWORD}
    # Make sure postgres is ready to accept connection as the indicator that 
    # Postgres is ready
    # ref https://www.postgresql.org/docs/current/app-pg-isready.html
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${PGUSER}"]
      # a more complete command
      # test: ["CMD-SHELL", "pg_isready -U ${PGUSER} -d ${PGDATABASE} -h 127.0.0.1"]
      interval: 2s
      timeout: 10s
      retries: 5
volumes:
  tes-pg:
    external: true

networks:
  # this will always create new network
  # my-local-net:
  #   driver: bridge
  my-local-net:
    name: mynet
    external: true

```

`image: postgres:16.2` : I want to use the current latest postgres, which is 16.2. Make sure you specify the major version(16 in this case), as the default is the latest, and when the version change, it almost always has incompatibility 

`container_name`: the name of the container to be built

`hostname`: address for this service in the docker network

`networks`: specify which network this service attached to, so when you want to access this service, you need to attach to this network and use `hostname` to connect to the database. If the network is not specified, docker compose will create new network and attach all services described in the yml file to it, so all services can communicate with each other. I use predefined network because it will be easier when other container outside the one described in this `docker-compose.yml` to connect to the services here

`ports` : mapping exposed port from container to host(the computer)

`volume`: persist the data created by postgre to disk. So when we restart the container, it can resume using data from volume instead of creating data. Other container can use this data too. Beware though when you use different version of postgres, it could result in corrupt data(I did it with different mysql version before lol)

`environment`: this will setup the postgres password when we want to enter the postgres. You could setup username ,password, and database name, the default is `postgres` or if the username is specified and database name is not, it would default to username. I read the documentation [here on POSTGRES_DB section](https://hub.docker.com/_/postgres)

`healthcheck` : is how I defined healthcheck that I will use in the application service(the `condition` on `depends_on` key in `go-market-warehouse-api` service). The `depends_on` is to tell the `go-market-warehouse-api` that it should start when the the database service is deemed to be healthy, which I define as the database ready to accept TCP connection. Why it is necessary? if I don't specify the `depends_on` the application will try to connect to db when db is not fully instantiated or ready  or both, so the application would throw error. If I only specify `depends_on: local-pg-16`, docker would start creating the application service after the database service is not yet ready to accept TCP connection. The are use cases when you need to seed data to db before the db is ready to accept connection too. Hence I use the healthcheck function

`volumes:` I specify `volume` here to point it at the predefined volume that I execute before I create the `docker-compose.yml`. If `external: true` is not specified, docker-compose will create new volume that has the name of `<project-name>_tes-pg`, same as network

`network`: specify which network to be used. `my-local-net` here is the name, `name: mynet` is the network name on the docker, and `external: true` is same as the volume, it tells docker to use predefined network  that exists on the docker(external of this `docker-compose.yml`)


Here's the full `docker-compose.yml`:
```yml
services:
  go-market-warehouse-api:
    depends_on:
      local-pg-16:
        condition: service_healthy
    # always restart when app crashes
    restart: always
    build:
      context: .
    image: go-market-warehouse-api:v1.0
    container_name: go-market-warehouse-api
    hostname: go-market-warehouse-api
    networks:
      - my-local-net
    ports:
      - 80:8080
    environment:
      - PGUSER=${PGUSER:-totoro}
      - PGPASSWORD=${PGPASSWORD:?database password not set}
      - PGHOST=${PGHOST:-db}
      - PGPORT=${PGPORT:-5432}
      - PGDATABASE=${PGDATABASE:-mydb}
    deploy:
      restart_policy:
        condition: on-failure
  local-pg-16:
    image: postgres:16.2
    container_name: local-pg-16
    hostname: local-pg-16
    networks:
      - my-local-net
    ports:
      - 5432:5432
      - 8080:8080
    volumes:
      - tes-pg:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=${PGPASSWORD}
    # Make sure postgres is ready to accept connection as the indicator that 
    # Postgres is ready
    # ref https://www.postgresql.org/docs/current/app-pg-isready.html
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${PGUSER}"]
      # a more complete command
      # test: ["CMD-SHELL", "pg_isready -U ${PGUSER} -d ${PGDATABASE} -h 127.0.0.1"]
      interval: 2s
      timeout: 10s
      retries: 5
volumes:
  tes-pg:
    external: true

networks:
  # this will always create new network
  # my-local-net:
  #   driver: bridge
  my-local-net:
    name: mynet
    external: true
```

You can clone and try it from the repo then hit `localhost` 
