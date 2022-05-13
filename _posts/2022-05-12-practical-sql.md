---
layout: post
title:  Practical SQL
date:   2022-05-12 20:10
categories: sql containers python
---
My original motivation behind this blog was to document my progress while working through 'Practical SQL, 2nd Edition', by Anthony DeBarros. I have a big interest in data, the gathering, andthe manipulation of. Maybe that falls in line with data analysis. With that said, I am not a statistician, just a sysadmin trying to grow my skills. 

## Envrionment Setup
This book uses PostgreSQL. While that is easy enough to install locally, I preferred not to. Instead, I decided to use containers to standup the environment that I will use while working through this book. My laptop is running Fedora, so I am using podman. However, the same commands should work with Docker. 

### Container Setup
The first step is probably the easiest and that is just to download/pull the images that we want to use. In this case it is pgadmin4 and PostgreSQL 14.

```
$ podman pull docker.io/dpage/pgadmin4:latest
$ podman pull postgres:latest
```

Next I created a pod. Containers in the same pod can communicate with each other without extra setup. This also allows me to pause, stop, start, or restart all containers in the pod at once.

```
$ podman pod create --name postgres --publish 8080:80
```
Next I created a specific volume for this deployment. I'll be mapping /var/lib/postgres to it. In my mind, this will make it easier to migrate to a new version of the database if needed. Thestep after creating it just exports the mountpoint path to an environment variable. I used this when deploying the container.

```
$ podman volume create postgres
```
{% raw %}
```bash
$ export POSTGRES_MOUNT=$(podman volume inspect postgres --format="{{ .Mountpoint }}")
```
{% endraw %}
 
run pgadmin in container using the required environment variables...

```
$ podman run -d --name pgadmin4 --pod postgres -e PGADMIN_DEFAULT_EMAIL=admin@localhost.com -e PGADMIN_DEFAULT_PASSWORD=$PGADMIN_DEFAULT_PASSWORD
```

Here I test and varify that pgadmin is up and running.
```
$ curl -s localhost:8080/login|grep pgadmin
    <link type="text/css" rel="stylesheet" href="/static/js/generated/pgadmin.style.css?ver=60800"/>
        <link type="text/css" rel="stylesheet" href="/static/js/generated/pgadmin.css?ver=60800" data-theme="standard"/>
              <span class="d-flex justify-content-center pgadmin_header_logo"
```

Run postgres container
```
$ podman run -d --name pg14 --pod postgres --volume=$POSTGRES_MOUNT:/var/lib/pgsql/data:Z -e "POSTGRES_PASSWORD=$POSTGRES_PASSWORD" -e POSTGRES_USER=postgres docker.io/library/postgres
```
Connect to postgres container and verify that it is running.
```
$ podman container exec -it pg14 /bin/bash
root@postgres:/# su - postgres
postgres@postgres:~$ psql
psql (14.2 (Debian 14.2-1.pgdg110+1))
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)
```
```
postgres=# SELECT version();
                                                           version
-----------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 14.2 (Debian 14.2-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
(1 row)
```

## Chapter 2 Exercises
Here are my solutions to the 'Try It Yourself' exercises at the end of chapter 2. I realize my solutions may not be best practice or leading practice, but I do hope to improve upon that as I work through the book. Specifically, I probably shouldn't run, ```SELECT *```. Also, there should probably be a unique key that links both tables together.
```sql
-- Chapter 2 Exercise 1:
CREATE TABLE animals(
	id bigserial,
	name varchar(50),
	age smallint
);

CREATE TABLE animal_info(
	id bigserial,
	name varchar(50),
	class varchar(50),
	ord varchar(50),
	family varchar(50),
	genus varchar(50),
	species varchar (50),
	birth_date date
)
```

```sql
-- Chapter 2 Exercise 2:
INSERT INTO animals(name, age)
VALUES ('Tony the Tiger', 70),
           ('Tucan Sam', 59),
           ('Chester Cheetah', 37);
           
INSERT INTO animal_info(name, class, ord, family, genus, species, birth_date)
VALUES ('Tony the Tiger','Mammalia', 'Carnivora', 'Felidae', 'Panthera', 'P. tigris', '1952-01-01'),
       ('Tucan Sam', 'Aves', 'Piciformes', 'Ramphastidae', 'Ramphastos', 'Ramphastos toco', '1963-01-01'),
       ('Chester Cheetah', 'Mammalia', 'Carnivora', 'Felidae', 'Acinonyx', 'A. jubatus', '1985-01-01');
```

Here I varify what was run previously.
```
postgres=# \d
                 List of relations
 Schema |        Name        |   Type   |  Owner   
--------+--------------------+----------+----------
 public | animal_info        | table    | postgres
 public | animal_info_id_seq | sequence | postgres
 public | animals            | table    | postgres
 public | animals_id_seq     | sequence | postgres
(4 rows)

postgres=# SELECT * FROM animal_info;
 id |      name       |  class   |    ord     |    family    |   genus    |     species     | birth_date 
----+-----------------+----------+------------+--------------+------------+-----------------+------------
  1 | Tony the Tiger  | Mammalia | Carnivora  | Felidae      | Panthera   | P. tigris       | 1952-01-01
  2 | Tucan Sam       | Aves     | Piciformes | Ramphastidae | Ramphastos | Ramphastos toco | 1963-01-01
  3 | Chester Cheetah | Mammalia | Carnivora  | Felidae      | Acinonyx   | A. jubatus      | 1985-01-01
(3 rows)

postgres=# SELECT * FROM animals;
 id |      name       | age 
----+-----------------+-----
  1 | Tony the Tiger  |  70
  2 | Tucan Sam       |  59
  3 | Chester Cheetah |  37
(3 rows)
```

That's enough for now. I'll do my best to document my progress here as I go through the book.
