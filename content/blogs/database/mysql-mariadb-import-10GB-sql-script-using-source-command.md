---
title: Import 10GB SQL script successfully using source command 
date: 2025-05-03
tags: [ linux, dbeaver, mysql, ssh, sql, docker ]
---
If you've ever tried importing a large .sql file using GUI tools like DBeaver, SQLyog, or HeidiSQL, chances are you've seen them freeze, timeout, or crash when the file is too big.

I’ve personally experienced this multiple times — especially when working with SQL files larger than 1GB.

But here's the tool that just works, even for a 10GB+ SQL script:

the `source` command in the MySQL (or MariaDB) CLI.

# How to Use the source Command

 1. (Optional): Open terminal and cd to the directory where your SQL file is located:

cd /path/to/your/sqlfile/

2. Login to your MySQL/MariaDB server using `mariadb` or `mysql`

```
mariadb -h 127.0.0.1 -P 3310 -u root -padmin --skip-ssl
```

3. Select your target database:

```
USE your_database;
```

4. Run the import with source:
```
source truckload.sql;
```

Note: this will work when you are in the same directory as the `truckload.sql`. If not, you need to do some `../../somewhere-to-your-sql`

That’s it. The import will start and you’ll see progress directly in the terminal.
