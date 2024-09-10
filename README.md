# Defacto2 SQL database

This repository contains a [Docker container](https://www.docker.com/) to create and import the Defacto2 database. The database is a collection of tens of thousands of records that document the history of the PC scene. The data is used to power the [Defacto2.net](https://defacto2.net) website.

> [!IMPORTANT]  
> The [preserved](https://github.com/Defacto2/database-ps/blob/main/sql/create-defacto2.sql) MySQL database used in this migration was finalized and retired in September 2024.
> After importing there should be a total of 49,083 records in the "files" table.

### Table of contents

- [Defacto2 PostgreSQL database](#defacto2-postgresql-database)
    - [Table of contents](#table-of-contents)
    - [Setup](#setup)
    - [First time setup](#first-time-setup)
      - [Optional, cleanup the migration containers](#optional-cleanup-the-migration-containers)
    - [PostgreSQL admin interface](#postgresql-admin-interface)
    - [Start and stop](#start-and-stop)
    - [Remove, reset or resync the database data](#remove-reset-or-resync-the-database-data)
    - [Dataset](#dataset)
      - [Files](#files)
    - [License](#license)

### Setup

The database is a [Docker Compose](https://docs.docker.com/compose/) project that first must be cloned to your local machine. The project includes a `docker-compose.yml` file that defines the database container and a the migration container. The migration container is used to import the data from the live, MySQL Defacto2 database.

Docker or [Docker Desktop](https://www.docker.com/products/docker-desktop/) also needs to be installed on your machine.
```sh
# go to your projects folder
cd ~

# clone this repository
git clone git@github.com:Defacto2/database-ps.git

# OR, use the gh cli tool
gh repo clone Defacto2/database-ps
```

### First time setup

On a new install for the Defacto2 database, a data migration will need to be run for the first time. This will create the PostgreSQL database Docker container and import the data from the live, MySQL Defacto2 database. The migration will take a few minutes to complete.

Docker or Docker desktop must first be running, then open a terminal and run the following commands.

```sh
cd database-ps

# migrate the Defacto2 data from MySQL to PostgreSQL
docker compose --profile migrater up
```

If successful, the output will look similar to this

```
...

migration   |              table name     errors       rows      bytes      total time
migration   | -----------------------  ---------  ---------  ---------  --------------
migration   |         fetch meta data          0          4                     0.066s
migration   |          Create Schemas          0          0                     0.001s
migration   |        Create SQL Types          0          0                     0.003s
migration   |           Create tables          0          2                     0.027s
migration   |          Set Table OIDs          0          1                     0.004s
migration   | -----------------------  ---------  ---------  ---------  --------------
migration   |            public.files          0      49083    41.3 MB          1.134s
migration   | -----------------------  ---------  ---------  ---------  --------------
migration   | COPY Threads Completion          0          4                     1.136s
migration   |  Index Build Completion          0          3                     0.150s
migration   |          Create Indexes          0          3                     0.251s
migration   |         Reset Sequences          0          1                     0.028s
migration   |            Primary Keys          0          1                     0.002s
migration   |     Create Foreign Keys          0          0                     0.000s
migration   |         Create Triggers          0          0                     0.000s
migration   |        Install Comments          0         49                     0.006s
migration   |              after load          0          2                     0.011s
migration   | -----------------------  ---------  ---------  ---------  --------------
migration   |       Total import time          ✓      49083    41.3 MB          1.584s
```

Note, the post-migration process will delete both the unused, `public.groupnames` and `public.netresources` tables.

#### Optional, cleanup the migration containers

Once the migration is complete the databases will be running in the background. To stop the database containers, tap `Ctrl+c` in the terminal window. Then run the following commands to clean the migration containers.

```sh
cd database-ps

# delete the one-time use, migration containers and associated volumes
docker compose rm migrate mysql dbdump --stop
docker volume rm database-ps_tmpdump database-ps_tmpsql
```

### PostgreSQL admin interface

The database container includes a web-based admin interface that can be used to view and edit the database. The Adminer interface is found at http://localhost:8080/?pgsql=db&username=root&db=defacto2_ps&ns=public,

- _System_, `PostgreSQL`
- _Server_, `db`
- _Username_, `root`
- __Password__, `example`

To select and show the data use, http://localhost:8080/?pgsql=db&username=root&db=defacto2_ps&ns=public&select=files

### Start and stop

Once the migration is complete, the database container can be started and stopped as needed.

```sh
cd database-ps

# start the postgres database server (tap Ctrl+c to stop)
docker compose up
```

Or to run the database in the background.

```sh
cd database-ps

# start the postgres database server in the background
docker compose -d up

# stop the postgres database server
docker compose down
```

### Remove, reset or resync the database data

The simplist way to reset the database is to delete the container and start again. **This will delete all the data and the database container**.

```sh
cd database-ps

docker compose --profile migrater up --force-recreate
```

---

### Dataset

The data gets created into a single table `files`. The table is a collection of tens of thousands of records that document the history of the PC scene. The data is used to power the [Defacto2.net](https://defacto2.net) website.

#### Files

| Column                    | Description                                                         | Example value                              |
| ------------------------- | ------------------------------------------------------------------- | ------------------------------------------ |
| id                        | Primary key                                                         | `8968`                                     |
| uuid                      | Unique identifier used as the stored file and images name           | `b826e39b-66c6-4929-8e5a-f59b07ffaa00`     |
| list_relations            | List of associated Defacto2 records                                 | Alternative;a84626                         |
| web_id_github             | Github repository Id                                                |                                            |
| web_id_youtube            | YouTube video Id                                                    |                                            |
| web_id_pouet              | Pouët record Id                                                     |                                            |
| web_id_demozoo            | Demozoo record Id                                                   | `158448`                                   |
| group_brand_for           | Group or brand authorship                                           | The Dream Team                             |
| group_brand_by            | Group or brand authorship                                           |                                            |
| record_title              | Production title or magazine issue                                  | Midwinter II                               |
| date_issued_year          | Published year                                                      | 1992                                       |
| date_issued_month         | Published month                                                     | 3                                          |
| date_issued_day           | Published day                                                       | 26                                         |
| credit_text               | Writing credits                                                     |                                            |
| credit_program            | Programming credits                                                 |                                            |
| credit_illustration       | Artist credits                                                      |                                            |
| credit_audio              | Composer credits                                                    |                                            |
| filename                  | Filename of the download                                            | `MID2TDT1.ZIP`                             |
| filesize                  | Size of the download                                                | 30923                                      |
| list_links                | List of associated URLs                                             |                                            |
| file_security_alert_url   | A URL to the results of a virus scan                                |                                            |
| file_zip_content          | List of files and directories contained in the download archive     | DREAM.NFO INTRO.EXE MUSIC TEXT             |
| file_magic_type           | File type metadata                                                  | Zip archive data, at least v1.0 to extract |
| preview_image             | The name of a file within the archive that was used as a screenshot |                                            |
| file_integrity_strong     | SHA386 hash value of the download                                   | 22370b5e81...                              |
| file_integrity_weak       | MD5 hash value of the download                                      | c00fccc640...                              |
| file_last_modified        | Last modified date value of the download                            | 2017-03-19 05:49:14                        |
| platform                  | Computer platform tag                                               | dOS                                        |
| section                   | Category tag                                                        | releaseadvert                              |
| comment                   | Description of the download                                         |                                            |
| deletedat                 | When this record was disabled                                       |                                            |
| deletedby                 | The id of the account which disabled this record                    |                                            |
| createdat                 | When this record was created                                        | 2014-10-13 12:08:52                        |
| dosee_run_program         | Program filename to run in DOSee, the MS-DOS emulator               |                                            |
| retrotxt_readme           | The filename of a text file to display in the browser               | `DREAM.NFO`                                |
| retrotxt_no_readme        | Toggle to disable retrotxt_readme                                   |                                            |
| dosee_hardware_cpu        | DOSee CPU emulation selection                                       | 486                                        |
| dosee_hardware_graphic    | DOSee graphic card selection                                        | vga                                        |
| dosee_hardware_audio      | DOSee audio card selection                                          | covox                                      |
| dosee_no_aspect_ratio_fix | DOSee aspect-ratio toggle                                           |                                            |
| dosee_incompatible        | Flag this record as incompatible with DOSee                         |                                            |
| dosee_no_ems              | DOSee Expanded memory toggle                                        | 1                                          |
| dosee_no_xms              | DOSee Extended memory toggle                                        | 1                                          |
| dosee_no_umb              | DOSee Upper memory toggle                                           | 1                                          |
| dosee_load_utilities      | Load DOSee utilities                                                |                                            |
| updatedby                 | The id of the account which updated this record                     | `ADB7C2BF-7221-467B-B813-3636FE4AE16B`     |
| updatedat                 | When this record was last updated                                   | 2017-03-19 05:57:12                        |

---

### License

The database data is licensed under a [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license.
