---
layout: post
title:  The Do Over - SQL Setup 
date:   2022-05-14 15:11
categories: postgreql linux containers podman
---
Turns out that you should read the manual...

## Container Setup - Redo
While my container setup worked without issue, I noticed a couple of mistakes post setup. Specifically, the way in which I mapped the volume. This was a trivial issue to resolve, but it doesreinforce a key point. Always consult the manual/documentation if you're uncertain about something. 

The first time I ran this command, I exported the path of the podman volume to an environment variable and mounted it. That was unnecessary. This time I just used the name of the volume. I probably should have picked something more descriptive. I also created a directory in my directory to mount within the container. Everything else is the same.
```
$ podman run -d --name pg14 --pod postgres --volume=postgres:/var/lib/postgresql/data:Z --volume=/home/chris/data:/data:Z -e "POSTGRES_PASSWORD=$POSTGRES_PASSWORD" -e POSTGRES_USER=postgres docker.io/library/postgres
```
In order to get this working so that the postgres user could write from the container and not just root, I had to modify the permissions on the host(my laptop). I ran the command shown belowto set the group owner to the gid of the postgres group in the cotainer.
```
$ podman unshare chgrp 999 data/
```

With the volume mounted, I am able to easily read and write files to and from the container. 
Here I log into the container and create a text file.
```
$ podman container exec -it pg14 /bin/bash
```
Here we can see the /data volume mounted in the container
```
root@postgres:/# df -h /data/
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       238G   23G  215G  10% /data
```
```
# cd data/
# echo "hello from the container" > hello.txt
```

From my laptop/local machine, I can view the contents of the file.
```
$ pwd
/home/chris/data
$ ls
hello.txt
$ cat hello.txt 
hello from the container
```
I can also create files from my local machine that will be accessible in the container.
```
$ touch hello{1..9}.txt
```

Back to the container
```
root@postgres:/data# ls hello*
hello1.txt  hello2.txt	hello3.txt  hello4.txt	hello5.txt  hello6.txt	hello7.txt  hello8.txt	hello9.txt  hello.txt
root@postgres:/data# rm !$
rm hello*
root@postgres:/data#
```

## Practical SQL
After redoing the setup, I was able to progress through chapters 3 and 4 of Practical SQL 2nd Edition. These weren't too complicated. Chapter 3 covered the basics of writing ```SELECT``` statements. Chapter 4 covered data types. There isn't much else to say at this point, but my answers to the 'Try It Yourself' section are below. Without context, these answers probably mean little to those reading, but I think writing them here helps me out.
 
### Chapter 3 Answers
```sql
--Chapter 3 exercise 1
SELECT school, first_name, last_name
FROM teachers
ORDER BY school ASC, last_name ASC;

--Chapter 3 exercise 2
SELECT first_name, last_name, salary
FROM teachers
WHERE first_name LIKE 'S%' AND salary > 40000;

--Chapter 3 exercise 3
SELECT first_name, 
       last_name, 
       hire_date, 
       salary
FROM teachers
WHERE hire_date >= '2010-01-01'
ORDER BY salary DESC;
```

### Chapter 4 Answers
```
1. The numeric type would be approriate for a column that needs to store mileage information. If mileage won't exceed 999 miles and we are tracking to the tenth, we need at most 4 digits of precision. numeric(4, 1)

2. Either varchar() or text data types would be appropriate for storing first and last names. It's a good idea to have them as seperate columns because it will allow you to sort based on these columns.

3. If you try to cast '4//2021' as a timestamp, you will get an error. The time stamp is malformed. It's missing information.
```
