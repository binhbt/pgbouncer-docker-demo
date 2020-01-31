# docker-postgres-pgbouncer

This is a simple project to install, configure and monitor [PgBouncer](https://pgbouncer.github.io) connection pooler for [PostgreSQL](https://www.postgresql.org)

> This is a simple project using docker.

# Table of contents
1. [Install](#install)
2. [Configuration](#configuration)
3. [Monitoring](#monitoring)
4. [References](#references)


### Install services<a name="install"></a>

Download the [docker-compose](https://docs.docker.com/compose/) file:

```sh
wget https://raw.githubusercontent.com/guedim/docker-postgres-pgbouncer/master/docker-compose.yml
```

Later, start the services ([Postgres](https://www.postgresql.org/) - [PgBouncer](https://pgbouncer.github.io) - [PgAdmin](https://www.pgadmin.org)) using [docker-compose](https://docs.docker.com/compose/):
```sh
docker-compose up
```
<p align="center">
<img src="https://github.com/guedim/docker-postgres-pgbouncer/blob/master/resources/images/docker-compose-up.png" height="200" width="300" >
</p>


### Configuration<a name="configuration"></a>

Access to [PgAdmin](https://www.pgadmin.org)  console, using http://192.168.99.100:5050

> Use the credentials:
> Username:pgbouncer_database
> Password:12345678

<p align="center">
<img src="https://github.com/guedim/docker-postgres-pgbouncer/blob/master/resources/images/pgadmin_login.png" height="200" width="260" >
</p>

Create a server, but dont forget to use the [PgBouncer](https://pgbouncer.github.io) credentials, host and port:

> Use the parameters:
> host:192.168.99.100
> port:6432
> user:postgres
> password:postgres
> database:postgres

<p align="center">
<img src="https://github.com/guedim/docker-postgres-pgbouncer/blob/master/resources/images/create_server.png" height="250" width="300" >
</p>

Now, you are connected to Postgres but using [PgBouncer](https://pgbouncer.github.io).

### Monitoring<a name="monitoring"></a>

For Monitoring, we will use [PgBouncer stats](https://pgbouncer.github.io/usage.html) in one new database schema.

Execute the sql file https://raw.githubusercontent.com/guedim/docker-postgres-pgbouncer/master/monitoring-pgbouncer.sql:

```sql
# Create server and register pgbouncer
CREATE EXTENSION dblink;
CREATE SERVER pgbouncer FOREIGN DATA WRAPPER dblink_fdw OPTIONS (host '192.168.99.100',port '6432', dbname 'pgbouncer');
CREATE USER MAPPING FOR PUBLIC SERVER pgbouncer OPTIONS (user 'pgbouncer');

# Register pgbouncer data in one isolated schema
CREATE SCHEMA pgbouncer;
CREATE VIEW pgbouncer.clients 	AS SELECT * FROM dblink('pgbouncer', 'show clients') 	AS _(type text, "user" text, database text, state text, addr text, port integer, local_addr text, local_port integer, connect_time timestamp with time zone, request_time timestamp with time zone, ptr text, link text, remote_pid smallint, tls text);
CREATE VIEW pgbouncer.config  	AS SELECT * FROM dblink('pgbouncer', 'show config')  	AS _(key text, value text, changeable boolean);
CREATE VIEW pgbouncer.databases AS SELECT * FROM dblink('pgbouncer', 'show databases') 	AS _(name text, host text, port integer, database text, force_user text, pool_size integer, reserve_pool integer, pool_mode text, max_connections integer, current_connections integer, paused integer, disabled integer);
CREATE VIEW pgbouncer.lists 	AS SELECT * FROM dblink('pgbouncer', 'show lists') 		AS _(list text, items int);
CREATE VIEW pgbouncer.pools 	AS SELECT * FROM dblink('pgbouncer', 'show pools') 		AS _(database text, "user" text, cl_active integer, cl_waiting integer, sv_active integer, sv_idle integer, sv_used integer, sv_tested integer, sv_login integer, maxwait integer, pool_mode text);
CREATE VIEW pgbouncer.servers 	AS SELECT * FROM dblink('pgbouncer', 'show servers') 	AS _(type text, "user" text, database text, state text, addr text, port integer, local_addr text, local_port integer, connect_time timestamp with time zone, request_time timestamp with time zone, ptr text, link text, remote_pid smallint, tls text);
CREATE VIEW pgbouncer.sockets 	AS SELECT * FROM dblink('pgbouncer', 'show sockets') 	AS _(type text, "user" text, database text, state text, addr text, port int, local_addr text, local_port int, connect_time timestamp with time zone, request_time timestamp with time zone, ptr text, link text, recv_pos int, pkt_pos int, pkt_remain int, send_pos int, send_remain int, pkt_avail int, send_avail int);
```

Finally, you can query the [PgBouncer stats](https://pgbouncer.github.io/usage.html) 

<p align="center">
<img src="https://github.com/guedim/docker-postgres-pgbouncer/blob/master/resources/images/monitoring.png" height="250" width="800" >
</p>


### References<a name="references"></a>

* https://wiki.postgresql.org/wiki/PgBouncer
* https://pgbouncer.github.io/usage.html
* https://pgbouncer.github.io/faq.html
* https://opensourcedbms.com/dbms/setup-pgbouncer-connection-pooling-for-postgresql-on-centosredhatfedora/https://www.enterprisedb.com/resources/tutorials/how-set-pgbouncer-connection-pooling-postgres-plus-standard-server
* https://opensourcedbms.com/dbms/setup-pgbouncer-connection-pooling-for-postgresql-on-centosredhatfedora/
* https://www.enterprisedb.com/resources/tutorials/how-set-pgbouncer-connection-pooling-postgres-plus-standard-server
* http://get.enterprisedb.com/docs/Tutorial_All_PPSS_pgBouncer.pdf
