---
author: "Yanuar Arifin"
title: "Connect To MySQL in Golang"
date: "2022-04-14"
description: ""
tags: [
    "golang",
    "mysql",
    "query",
]
draft: true
---

It's more than just `db.Open(...)`
<!--more-->

Here, I'll give you the finished connection:

```go
    // Open only validate DSN data, not creating connection
	db, err := sql.Open("mysql", "root:@/test_db")
	if err != nil {
		panic(err.Error()) // TODO: better error handling
	}


    // defer is called when you exit the function,
    // most of the times you need to call this exact code
    // on your main function because you wanted to keep the
    // database open for your app to create connection to it
	defer db.Close()

	// Open doesn't open a connection. Only validate DSN data

    // Ping:  verify if connection is still alive
    // and establish connection if necessary
	err = db.Ping()
	if err != nil {
		panic(err.Error()) // TODO: better error handling
	}

// recomended to use less than 5 min
	db.SetConnMaxLifetime(time.Minute * 3)
// recomended to limit connection used by this app. No number recommendation
	db.SetMaxOpenConns(20)
// set same as above
	db.SetMaxIdleConns(20)
```

Let's dissect what happened here.

---
# sql.Open

`sql.Open` does not create connection to MySQL server, it only validate Data Soure Name(DSN). This is an example of DSN

```
username:password@protocol(address)/dbname?param=value
```

Which is the second parameter of the connection.

Then what the first parameter for?
See  Register MySQL Driver below

**Note**

if not connect try wrap the `host:port` part with protocol(ex. `net`, or `tcp`).

like this:
```
# From this
db, err := sql.Open("mysql", "root:password@100.22.1.10:3357/test_db")
# Into
db, err := sql.Open("mysql", "root:password@tcp(100.22.1.10:3357)/test_db")
# Or
db, err := sql.Open("mysql", "root:password@net(100.22.1.10:3357)/test_db")

```

---
# defer db.close()

`defer db.Close()` is telling Go to close the database when the function that calls this code ends regardless of where `defer` is called.

If it is a one file application and you need to connect to database for a few operation, you could apply this code. But if your codebase is big and getting a database connection is in a function that being called by other functions, **DO NOT** put this code on your get database connection function, As it will close when the function call, resulting in error `nil reference` even though `Ping()` success and when you copy paste the full code above it works too. Don't ask how I know this.

**NOTE:** closing database is very rare, you would put `db.close()` somewhere when your application shutting down, like at main function where it ended last.

---
# db.Set...

This part of the code make sure that you use your database connection pool wisely. As database has limits on how much connection they could have, and there are more than one app connect to database, for example your app, database clients, a scheduler, etc. And there could be more that 1 person working using the same database server and use many app to connect to the database server

From https://github.com/go-sql-driver/mysql README.md
- `db.SetConnMaxLifetime()` is required to ensure connections are closed by the driver safely before connection is closed by MySQL server, OS, or other middlewares. Since some middlewares close idle connections by 5 minutes, we recommend timeout shorter than 5 minutes. This setting helps load balancing and changing system variables too.

- `db.SetMaxOpenConns()` is highly recommended to limit the number of connection used by the application. There is no recommended limit number because it depends on application and MySQL server. In real life there will be many application making connection to MySQL server, like your app, your database client(phpMyAdmin, MySQL Workbench, etc). Those will use connection(s) to your database, you limit your application connection to make sure other app can connect to MySQL server

- `db.SetMaxIdleConns()` is recommended to be set same to `db.SetMaxOpenConns()`. When it is smaller than `SetMaxOpenConns()`, connections can be opened and closed much more frequently than you expect. Idle connections can be closed by the `db.SetConnMaxLifetime()`. If you want to close idle connections more rapidly, you can use `db.SetConnMaxIdleTime()` since Go 1.15.

