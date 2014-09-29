---
layout: page
title: Database
permalink: /database/
---

## MySQL Users and Routing

The *register* module provides an SQL database backend to user registration. This is an example configuration for MySQL. (The module can also be used for CDR and subscription features, which have been left out for simplicity.)

`mysqldb.conf:`

```INI
[default]
;host=
;port=0
database=yate
user=yate
password=mypassword
;socket=
;compress=disable
encoding=utf8
poolsize=1
```

`register.conf:`

```INI
[general]
expires=30
user.auth=yes
user.register=yes
user.unregister=yes
engine.timer=yes
call.route=yes
;...

[default]
priority=50
account=default

[user.auth]
query=SELECT password FROM users WHERE username='${username}' AND password IS NOT NULL AND password<>'' AND type='user' LIMIT 1
result=password

[user.register]
query=UPDATE users SET location='${data}',expires=NOW() + INTERVAL ${expires} SECOND,oconnection_id='${oconnection_id}' WHERE username='${username}' AND type='user' LIMIT 1

[user.unregister]
query=UPDATE users SET location=NULL,expires=NULL,oconnection_id=NULL WHERE expires IS NOT NULL AND username='${username}' AND type='user' LIMIT 1

[engine.timer]
query=UPDATE users SET location=NULL,expires=NULL,oconnection_id=NULL WHERE expires IS NOT NULL AND expires<=CURRENT_TIMESTAMP AND type='user'

[call.route]
query=SELECT location,(CASE WHEN location IS NULL THEN 'offline' ELSE NULL END) AS error,oconnection_id FROM users WHERE username='${called}' LIMIT 1
result=location
priority=120

;...
```

Priorities have to be set according to your configuration. I like to have regexroute:100 and register:120 for call.route. This way it is easy to simply {\em return} in regexroute and let register handle the routing.

The field `oconnection_id` is needed for Yate to map incoming calls to open TCP and TLS connections. SIP over UDP should work just fine without this field.

Tables were created like this:

```SQL
-- create Mysql user
CREATE USER 'yate'@'localhost' IDENTIFIED BY 'mypassword';
CREATE DATABASE yate;
GRANT DELETE, INSERT, SELECT, USAGE, UPDATE ON yate.* TO 'yate'@'localhost';
FLUSH PRIVILEGES;

use yate;

CREATE TABLE users (
username VARCHAR(128) UNIQUE,
`password` VARCHAR(128),
inuse INTEGER,
`location` VARCHAR(1024),
expires TIMESTAMP NULL DEFAULT NULL,
`type` VARCHAR(20) NULL DEFAULT NULL,
oconnection_id varchar(1024) null default null);
```
