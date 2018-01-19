---
layout: post
title: Dockerized MySQL hot backup tool with Percona Xtrabackup
---

Just created a dockerized tool that wraps Percona Xtrabackup binary and 
periodically does a hot backup of your MySQL/MariaDB database. I already use it on production 
and it works like a charm! Feel free to use and/or contribute. Licensed under MIT so anyone can use.

[Docker image](https://hub.docker.com/r/mikemix/percona-xtrabackup/) /
[GitHub repository](https://github.com/mikemix/percona-xtrabackup-cron)

Just mount your cloud bucket to the filesystem, one docker command to run the tool
and don't worry about your database anymore.
