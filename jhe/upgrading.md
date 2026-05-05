---
title: Upgrading
---

## Upgrading

1. Locate the version of interest, eg `v0.0.8`, from the [Github Releases](https://github.com/jupyterhealth/jupyterhealth-exchange/releases) page
2. Stop the server
3. From the root directory:
   1. Replace the source code by running: `git checkout tags/v0.0.8`
   2. Migrate the database by running: `python manage.py migrate`
4. Start the server

## Rolling back

1. Locate the version of interest, eg `v0.0.7`, from the [Github Releases](https://github.com/jupyterhealth/jupyterhealth-exchange/releases) page
2. Click on the tag icon at the top to view the repository tree
3. Click througfh to the `core/migrations` directory
4. Make a note of the numeric prefix from the latest migration file name, eg `0011` for `0011_drop_jhe_user_permissions_groups.py`
5. Stop the server
6. In case of data loss, make a copy of your database by running the psql command:
   - `CREATE DATABASE jhe_backup WITH TEMPLATE jhe_original OWNER your_user;`
6. From the root directory:
   1. Roll back the database by running: `python manage.py migrate core 0011`
   2. Replace the source code by running: `git checkout tags/v0.0.7`
7. Start the server