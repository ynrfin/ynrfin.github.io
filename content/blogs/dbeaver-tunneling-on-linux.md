---
title: Dbeaver Jump Server Tunneling Problem On Linux
date: 2020-02-26
tags: [ linux, dbeaver, mysql, java, ssh, tunneling ]
---

# Overview
---

I want to connect to remote MySQL server using DBeaver on linux. Note I need to connect jump server before I could open a connection to the database server. On Windows I could do that using Putty tunneling by SSH to first server, then from that server, I SSH second time to the DB server and tunnel the connection to a port. Using this recipe does not work on linux in my case. it will show error:

    dbeaver Can not read response from server. Expected to read 4 bytes, read 0 bytes before connection was unexpectedly lost.

This is how the connection looks like:

```
    me
    |
|----------
|  bridge |-
|----------
    |
|---------------       |--------------
|  app server  |-------|  DB Server |-
|---------------       |--------------

```

*NOTE*: I use MySql Driver 5.* because the Mysql version is 5.6

*Solution*: for the last tunneling, use DBeaver's SSH tab to do that.

# #How To configure
---

### Configure SSH to bridge on putty
---

just plain old SSH to the bridge

-- Session Tab

- IP : bridge's IP
- PORT: 22(the port that they allow to connect)

-- Tunnel Tab

- exposed bridge port : 51717
- IP: 192.168.32.31:22 (the app IP)

It means that after we login in this bridge app, putty will expose our 51717 port to the app server, so we could connect to it

```

-------------------------------------------------------------------------------------------------------

               |--------|                        |--------|                           |--------|-
PC-(1)---------|        |------------------------|        |                           |        |-
               |        |                        |        |                           |        |-
PC -(2)-[51717]-------------[192.168.32.31:22]------      |                           |        |-
               | Bridge |                        | App IP |                           |  DB IP |-
               |--------|                        |        |                           |        |-
                                                 |        -(3)--[200.168.32.31:3066]--|        |-
                                                 |--------|                           |--------|-

---------------------------------------------------------------------------------------------------------
```

### Configure SSH to DB IP on Dbeaver
---

-- In General Tab

- IP : 200.168.32.31
- Port: 3066

-- In SSH Tab

- IP : 127.0.0.1
- Port: 51717

## Connection Steps
---

1. Connect To the bridge via Putty

    When we connect to the Bridge server using putty, Putty will link port 51717 to the app server ip & port. Meaning that if we connect to port 51717, that will make our PC act like it is the App Server.

2. Connect To App Server Via SSH on DBeaver
    We connect to localhost:51717 to tunnel to App server.

3. Connect to the DB Server using SSHed connection using Dbeaver.
    Step 2 will make us like we are in the App server, then use the fact that App server has link to DB Server to connect to DB server.

The way it works in DBeaver is:

- Connect to SSH tab if filled, then
- Connect on General tab

not the other way around.


# Further Modification
---

The question that remains:

Why I could not connect from Dbeaver using port to DB IP without using SSH first?

