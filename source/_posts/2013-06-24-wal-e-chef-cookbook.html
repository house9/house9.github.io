---
layout: post
title: "WAL-e chef cookbook"
date: 2013-06-24
comments: false
categories:
 - postgresql
---

Postgres has various backup and restore options

* [http://www.postgresql.org/docs/9.2/static/backup.html](http://www.postgresql.org/docs/9.2/static/backup.html)

Do you have a recovery plan in case your Postgres server crashes - your daily pg_dump is probably not going to cut it.

Postgres uses Write-Ahead Logging (WAL)

> Write-Ahead Logging (WAL) is a standard method for ensuring data integrity. A detailed description can be found in most (if not all) books about transaction processing. Briefly, WAL's central concept is that changes to data files (where tables and indexes reside) must be written only after those changes have been logged, that is, after log records describing the changes have been flushed to permanent storage. If we follow this procedure, we do not need to flush data pages to disk on every transaction commit, because we know that in the event of a crash we will be able to recover the database using the log: any changes that have not been applied to the data pages can be redone from the log records. (This is roll-forward recovery, also known as REDO.)

* from [http://www.postgresql.org/docs/9.2/static/wal-intro.html](http://www.postgresql.org/docs/9.2/static/wal-intro.html)


Setting up Continuous Archiving and Point-in-Time Recovery (PITR) for your Postgres WAL files is very complex, lucky for us the WAL-e project has simplified this process greatly. WAL-e has utilities for sending all WAL files to an AWS S3 bucket as the log files are being generated. More Information, see:

* [http://www.postgresql.org/docs/9.2/static/continuous-archiving.html](http://www.postgresql.org/docs/9.2/static/continuous-archiving.html)
* [https://github.com/wal-e/wal-e](https://github.com/wal-e/wal-e)

Recently I put together a chef cookbook which installs WAL-e on a Postgres instance, see:

* [https://github.com/house9/wal-e-cookbook](https://github.com/house9/wal-e-cookbook)
* [https://github.com/house9/use_wal_e](https://github.com/house9/use_wal_e)
