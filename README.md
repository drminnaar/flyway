![flyway-cover](https://user-images.githubusercontent.com/33935506/116072731-99059900-a6e3-11eb-857a-fff425de830b.png)

# Flyway Guide

This is a repository that consists of a collection of examples that illustrate how to use Flyway in terms of SQL migrations.

## Contents

* [Toolchain](#toolchain)
* [Environment Setup](#environment-setup)
* [Getting Started](#getting-started)
* [Example 1](#example-1)

---

## Toolchain

* [Visual Studio Code](https://code.visualstudio.com/)

  Visual Studio Code is a source code editor developed by Microsoft for Windows, Linux and macOS. It includes support for debugging, embedded Git control, syntax highlighting, intelligent code completion, snippets, and code refactoring.

* [Docker](https://www.docker.com)

  Docker is a computer program that performs operating-system-level virtualization also known as containerization. It is developed by Docker, Inc.

* [Docker-Compose](https://docs.docker.com/compose/overview/)

  Compose is a tool for defining and running multi-container Docker applications.

* [PostgreSQL](https://www.postgresql.org/)

  PostgreSQL is an object-relational database management system.

* [pgAdmin4](https://www.pgadmin.org/)

  Open Source administration and development platform for PostgreSQL

* [Flyway](https://flywaydb.org/)

  Flyway is an open source database migration tool.

---

## Environment Setup

You should have _Docker_ and _Docker-Compose_ installed. This can be verified by running the following commands from the command line.

* To verify Docker:

  ```bash
  docker version
  docker info
  docker run hello-world
  ```

* To verify Docker-Compose:

  ```bash
  docker-compose --version
  ```

If all is well, the above commands should have run flawlessly.

## Getting Started

There are 3 ways to get the repository for this guide:

1. Clone Repo Using HTTPS

   ```bash
   git clone https://github.com/drminnaar/flyway.git
   ```

1. Clone Repo Using SSH

   ```bash
   git clone git@github.com:drminnaar/flyway.git
   ```

1. Download Zip File

   ```bash
   wget https://github.com/drminnaar/flyway/archive/refs/heads/main.zip
   unzip ./main.zip
   ```

---

## Example 1

Everything that is required to run this example is managed via *docker-compose*.

The `docker-compose.yml` file defines a stack that provides the following:

* Postgres 12 Database
* Entry point script that creates the *heroes* database
* pgAdmin UI to view/manage database

There are 3 additional *docker-compose* files that are used to manage the *Flyway* migrations. They are listed as follows:

* `docker-compose-info` - Runs *Flyway* to obtain information regarding migrations
* `docker-compose-validate` - Runs *Flyway* to verify validity of migrations
* `docker-compose-migrate` - Runs *Flyway* to migrate migrations

### Start Stack

* Type the following command to initialise environment:

  ```bash
  docker-compose -f ./example1/docker-compose.yml up --detach

  # output
  Creating network "flywaynet" with driver "bridge"
  Creating volume "flyway-pg-data" with default driver
  Creating flyway-pgadmin ... done
  Creating flyway-pg      ... done
  ```

* Type the following command to verify that there are 2 containers running. One container will be our PostgreSQL server. The second container will be our pgAdmin web application.

  ```bash
  docker-compose -f ./example1/docker-compose.yml ps

  # output
       Name                   Command              State               Ports
  --------------------------------------------------------------------------------------
  flyway-pg        docker-entrypoint.sh postgres   Up      0.0.0.0:5432->5432/tcp
  flyway-pgadmin   /entrypoint.sh                  Up      443/tcp, 0.0.0.0:8080->80/tcp
  ```

### Define Migrations

For clarity sake, please take note that a migration is nothing more than a SQL file consisting of various SQL operations to be performed on the database.

The _heroes_ database now exists and we are ready to run our migrations. Please take note of the _migrations_ folder that is part of the repo for this example. The _migrations_ folder consists of 7 migrations that are briefly described as follows:

* V1_1__Create_hero_schema.sql - Creates a new _hero\_data_ schema

  ```sql
  CREATE SCHEMA hero_data AUTHORIZATION postgres;
  ```

* V1_2__Create_hero_table.sql - Create a new _hero_ table in the _hero\_data_ schema

  ```sql
  CREATE TABLE hero_data.hero
  (
      id BIGSERIAL NOT NULL,
      name VARCHAR(250) NOT NULL,
      description TEXT NOT NULL,
      debut_year INT NOT NULL,
      appearances INT NOT NULL,
      special_powers INT NOT NULL,
      cunning INT NOT NULL,
      strength INT NOT NULL,
      technology INT NOT NULL,
      created_at TIMESTAMPTZ NOT NULL,
      updated_at TIMESTAMPTZ NOT NULL
  );

  ALTER TABLE hero_data.hero ADD CONSTRAINT pk_hero_id PRIMARY KEY (id);
  ```

* V1_3__Add_Destroyer_hero.sql - Inserts our first hero into _hero_ table

  ```sql
  INSERT INTO hero_data.hero (
      name,
      description,
      debut_year,
      appearances,
      special_powers,
      cunning,
      strength,
      technology,
      created_at,
      updated_at) VALUES (
      'Destroyer',
      'Created by Odin, locked in temple, brought to life by Loki',
      1965,
      137,
      15,
      1,
      19,
      80,
      now(),
      now());
  ```

* V1_4__Create_user_schema.sql - Create a _user\_data_ schema

  ```sql
  CREATE SCHEMA user_data AUTHORIZATION postgres;
  ```

* V1_5__Create_user_table.sql - Create a new _user_ table in the _user\_data_ schema

  ```sql
  CREATE TABLE user_data.user
  (
      id BIGSERIAL NOT NULL,
      first_name VARCHAR(250) NOT NULL,
      last_name VARCHAR(250) NOT NULL,
      email VARCHAR(250) NOT NULL,
      alias VARCHAR(250) NOT NULL,
      created_at TIMESTAMPTZ NOT NULL,
      updated_at TIMESTAMPTZ NOT NULL
  );
  
  ALTER TABLE user_data.user ADD CONSTRAINT pk_user_id PRIMARY KEY (id);
  ```

* V1_6__Add_unique_hero_name_contraint.sql - Alter _hero_ table by adding a unique name constraint

  ```sql
  ALTER TABLE hero_data.hero ADD CONSTRAINT uk_hero_name UNIQUE (name);
  ```

* V1_7__Add_unique_user_email_constraint.sql - Alter _user_ table by adding a unique email constraint

  ```sql
  ALTER TABLE user_data.user ADD CONSTRAINT uk_user_email UNIQUE (email);
  ```

You will have noticed the strange naming convention. The way we name a migrations is as follows:

[According to the official Flyway documentation](https://flywaydb.org/documentation/concepts/migrations.html#naming), the file name consists of the following parts:

![flyway-naming-convention](https://user-images.githubusercontent.com/33935506/40931818-bc78fb5a-682c-11e8-90ce-cb9f8d0e8c95.png)

* **Prefix:** V for versioned migrations, U for undo migrations, R for repeatable migrations
* **Version:** Underscores (automatically replaced by dots at runtime) separate as many parts as you like (Not for repeatable migrations)
* **Separator:** __ (two underscores)
* **Description:** Underscores (automatically replaced by spaces at runtime) separate the words

### Manage Migrations

Finally we get to run our migrations. To run the migrations, we will use *Docker* and the *[official Flyway Docker Image](https://hub.docker.com/r/flyway/flyway)*

#### Get Migrations Info

Before running the migration, lets see what migrations we have. We can do that by running the following command:

```powershell
# run docker-compose-info.yml stack
docker-compose -f ./example1/docker-compose-info.yml up

# output

Flyway Community Edition 7.6.0 by Redgate
Database: jdbc:postgresql://flyway-pg/heroes (PostgreSQL 12.5)
Schema version: << Empty Schema >>
+------------+---------+----------------------------------+------+--------------+---------+
| Category   | Version | Description                      | Type | Installed On | State   |
+------------+---------+----------------------------------+------+--------------+---------+
| Versioned  | 1.1     | Create hero schema               | SQL  |              | Pending |
| Versioned  | 1.2     | Create hero table                | SQL  |              | Pending |
| Versioned  | 1.3     | Add Destroyer hero               | SQL  |              | Pending |
| Versioned  | 1.4     | Create user schema               | SQL  |              | Pending |
| Versioned  | 1.5     | Create user table                | SQL  |              | Pending |
| Versioned  | 1.6     | Add unique hero name contraint   | SQL  |              | Pending |
| Versioned  | 1.7     | Add unique user email constraint | SQL  |              | Pending |
| Repeatable |         | 001 Install extensions           | SQL  |              | Pending |
+------------+---------+----------------------------------+------+--------------+---------+
```

#### Validate Migrations

We can validate our migrations to determine anythin that should be fixed before running migrations.

```powershell
# run docker-compose-validate.yml
docker-compose -f ./example1/docker-compose-validate.yml up

# output
Flyway Community Edition 7.6.0 by Redgate
Database: jdbc:postgresql://flyway-pg/heroes (PostgreSQL 12.5)
ERROR: Validate failed: Migrations have failed validation
Detected resolved migration not applied to database: 1.1. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
Detected resolved migration not applied to database: 1.2. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
Detected resolved migration not applied to database: 1.3. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
Detected resolved migration not applied to database: 1.4. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
Detected resolved migration not applied to database: 1.5. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
Detected resolved migration not applied to database: 1.6. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
Detected resolved migration not applied to database: 1.7. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
Detected resolved repeatable migration not applied to database: 001 Install extensions. To fix this error, either run migrate, or set -ignorePendingMigrations=true.
```

#### Migrate Migrations

```powershell
# run docker-compose-migrate.yml stack
docker-compose -f ./example1/docker-compose-migrate.yml up

# output
Flyway Community Edition 7.6.0 by Redgate
Database: jdbc:postgresql://flyway-pg/heroes (PostgreSQL 12.5)
Successfully validated 8 migrations (execution time 00:00.043s)
Creating Schema History table "public"."flyway_schema_history" ...
Current version of schema "public": << Empty Schema >>
Migrating schema "public" to version "1.1 - Create hero schema"
Migrating schema "public" to version "1.2 - Create hero table"
Migrating schema "public" to version "1.3 - Add Destroyer hero"
Migrating schema "public" to version "1.4 - Create user schema"
Migrating schema "public" to version "1.5 - Create user table"
Migrating schema "public" to version "1.6 - Add unique hero name contraint"
Migrating schema "public" to version "1.7 - Add unique user email constraint"
Migrating schema "public" with repeatable migration "001 Install extensions"
Successfully applied 8 migrations to schema "public" (execution time 00:00.438s)
```

Let's validate our migrations again:

```powershell
docker-compose -f .\example1\docker-compose-validate.yml up

# output
Flyway Community Edition 7.6.0 by Redgate
Database: jdbc:postgresql://flyway-pg/heroes (PostgreSQL 12.5)
Successfully validated 8 migrations (execution time 00:00.046s)
```

Let's see what information is displayed for our migrations now:

```powershell
Flyway Community Edition 7.6.0 by Redgate
Database: jdbc:postgresql://flyway-pg/heroes (PostgreSQL 12.5)
Schema version: 1.7

+------------+---------+----------------------------------+------+---------------------+---------+
| Category   | Version | Description                      | Type | Installed On        | State   |
+------------+---------+----------------------------------+------+---------------------+---------+
| Versioned  | 1.1     | Create hero schema               | SQL  | 2021-04-26 10:15:15 | Success |
| Versioned  | 1.2     | Create hero table                | SQL  | 2021-04-26 10:15:15 | Success |
| Versioned  | 1.3     | Add Destroyer hero               | SQL  | 2021-04-26 10:15:15 | Success |
| Versioned  | 1.4     | Create user schema               | SQL  | 2021-04-26 10:15:15 | Success |
| Versioned  | 1.5     | Create user table                | SQL  | 2021-04-26 10:15:15 | Success |
| Versioned  | 1.6     | Add unique hero name contraint   | SQL  | 2021-04-26 10:15:15 | Success |
| Versioned  | 1.7     | Add unique user email constraint | SQL  | 2021-04-26 10:15:15 | Success |
| Repeatable |         | 001 Install extensions           | SQL  | 2021-04-26 10:15:15 | Success |
+------------+---------+----------------------------------+------+---------------------+---------+
```

### Connect To Database

Use you Postgres client of choice to connect to *heroes* database. If you're new to *Postgresql*, I have written a guide *[Postgresql Getting Started](https://github.com/drminnaar/guides/blob/master/postgresql-guide/1-getting-started.md)* that you can use to explore some tools and techniques for working with *Postgresql*.

I personally like to use the Postgres CLI tool *'psql'*. For more info on *psql*, I provide some details in my guide on how to install *psql* in Windows 10 and Ubuntu:

* [Ubuntu](https://github.com/drminnaar/guides/blob/master/postgresql-guide/1-getting-started.md#ubuntu-2004)
* [Windows 10](https://github.com/drminnaar/guides/blob/master/postgresql-guide/1-getting-started.md#using-the-installer)

#### Connect

```bash
# enter 'password' when prompted to enter password
psql -h localhost -U dbadmin --dbname heroes

# output
Password for user dbadmin:
psql (12.6 (Ubuntu 12.6-0ubuntu0.20.04.1), server 12.5)
Type "help" for help.

heroes=>
```

#### List Schemas

```bash
\dn

# output
   List of schemas
   Name    |  Owner
-----------+----------
 hero_data | postgres
 public    | postgres
 user_data | postgres
(3 rows)
```

#### List Tables

```bash
# list tables in public schema
\dt

# output
                 List of relations
 Schema |         Name          | Type  |  Owner
--------+-----------------------+-------+----------
 public | flyway_schema_history | table | postgres
```

```bash
# list tables in hero_data schema
\dt hero_data.

# output
          List of relations
  Schema   | Name | Type  |  Owner
-----------+------+-------+----------
 hero_data | hero | table | postgres
(1 row)
```
```bash
# list tables in user_data schema
\dt user_data.

# output
  Schema   | Name | Type  |  Owner
-----------+------+-------+----------
 user_data | user | table | postgres
(1 row)
```

#### Show Data

```bash
select * from hero_data.hero;

# output
 id |   name    |                        description                         | debut_year | appearances | special_powers | cunning | strength | technology |          created_at           |          updated_at
----+-----------+------------------------------------------------------------+------------+-------------+----------------+---------+----------+------------+-------------------------------+-------------------------------
  1 | Destroyer | Created by Odin, locked in temple, brought to life by Loki |       1965 |         137 |             15 |       1 |       19 |         80 | 2021-04-26 10:15:15.122975+00 | 2021-04-26 10:15:15.122975+00
(1 row)
```

---

## Versioning

I use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/drminnaar/guides/tags).

---

## Authors

* **Douglas Minnaar** - *Initial work* - [drminnaar](https://github.com/drminnaar)