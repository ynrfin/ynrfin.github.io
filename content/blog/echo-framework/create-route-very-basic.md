---
title: Echo Framework - Minimum working web server
date: 2023-04-28
tags: [golang, web-server, API, webapp, web-application]
---

## Create Most Basic Working Web Server

### Create Golang Project

```bash
# go mod init <project_name>
go mod init ynrfin.com/basic-project
```

### Install Echo Framework

```bash
go get github.com/labstack/echo/v4
```

### Create Web Server

#### File initialisation

Create `main.go` on project root and fill with this

```go
// main.go
package main

func main() {
    e := echo.New()

    e.GET("/", homeHandler)

    e.Logger.Fatal(e.Start(":8080"))
}

func helloHandler(c echo.Context) error {
    return c.JSON(200, "hello world")
}
```

### Try

Use curl to check the content

```bash
curl -X GET localhost:8080/
```
