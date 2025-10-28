+++
date = '2025-06-09T23:10:03+05:30'
draft = true
title = 'Postgres'
+++

### linux list all users 

```bash
cut -d: -f1 /etc/passwd 
```

 

### switch to **postgres** user

switching to postgres user as this user has some extended privileges
may be in order to prevent random users changing properties 
```bash
sudo -i -u postgres
```

### launch psql
Now in order to interact with PostgresSQL Database server we need to connect to CLI i.e psql
```bash
postgres@user-B760M:~$ psql
```



### List all DB
```bash
\l
```
### Connect to DB
```bash
# switch to mydb
\c mydb
# directly
psql -d db_name
# from an external url 
psql postgresql://libmgmtdb_user:xOpWmuHy1eJItkYLAqaJCyPiq5etX0fz@ppg-d3ka8t7fte5s7387ssj0-a.oregon-postgres.render.com/libmgmtdb
```

## List Tables 
```bash
\dt
```

## About Tables 
```bash
\d table_name
```

### Get applications having connection with DB 
```bash
# why ? we can't drop db if there is a connection
SELECT application_name FROM pg_stat_activity WHERE datname ='db_name';
```



