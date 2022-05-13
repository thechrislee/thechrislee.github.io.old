---
layout: post
title: "My first official post on the blog"
date: 2022-05-12
categories: jekyll blogging
---

This is a my first official post. As I do this more, hopefully the structure and content flows a little bit more.

## Creating this post

Creating this post was mostly manual. I may create a python script to generate blog templates. The command shown below was used to generate an empty file for this post. More to come...

```bash
$ touch $(date +%Y-%m-%d)-first-post.markdown
```

late edit...but I created a program to do this for me. The repo is here: <a href="https://github.com/thechrislee/new_post">new_post</a>

```
$ ./new_post.py -h
usage: new_post.py [-h] [-d DIR] [-c categories [categories ...]] name title

Generate blog post file

positional arguments:
  name                  the name of the file to write
  title                 blog title

optional arguments:
  -h, --help            show this help message and exit
  -d DIR, --dir DIR     output directory (default: .)
  -c categories [categories ...], --categories categories [categories ...]
```
