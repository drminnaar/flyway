![flyway](https://user-images.githubusercontent.com/33935506/40938269-afdc99c8-6841-11e8-9e11-860bf2e50596.png)

# Flyway README

This is a repository that consists of a collection of examples that illustrate how to use Flyway in terms of SQL migrations.

---

## Technology Used

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

### Get Repository

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
   wget https://github.com/drminnaar/flyway/archive/master.zip
   unzip ./master.zip
   ```

### Initialise Environment

* Navigate to the _'flyway'_ directory using the command line.

* Once inside the _flyway_ directory, you will notice a file called _'docker-compose.yml'_. This file contains all the instructions required to initialise our environment with all the required containerised software. In our case, the _docker-compose.yml_ files holds the instructions to run _postgresql_ and _pgadmin_ containers.

* Type the following command to initialise environment to run Flyway code migrations:

  ```bash
  docker-compose up
  ```

* Type the following command to verify that there are 2 containers running. One container will be our PostgreSQL server. The second container will be our pgAdmin web application.

  ```bash
  docker-compose ps
  ```

  The above command should display the running containers as specified in the _docker-compose_ file.

---

## Create Database

The first thing to be aware of when creating a migration, is that migrations do not create databases. Migrations only apply within the context of a database and do not create the database itself. Therefore, for my demonstration I will create an empty database from scratch and then create migrations for that database.

In this example, I create a database called _"heroes"_. It is a database that stores data related to, you guessed it, heroes.

* At this point, you should have a running PostgreSQL container instance. To verify this, run the following command:

  ```bash
  docker-compose ps
  ```

* List available databases by running the following command:

  ```bash
  docker exec -it $(docker container ls -qf name=pg-dev) psql -U postgres -c '\l'
  ```

  Currently, there is no _heroes_ database. 
  
* Type the following command to create a _heroes_ database:

  ```bash
  docker exec -it $(docker container ls -qf name=pg-dev) psql -U postgres -c 'CREATE DATABASE heroes OWNER postgres'
  ```

  List available databases by running the following command:

  ```bash
  docker exec -it $(docker container ls -qf name=pg-dev) psql -U postgres -c '\l'
  ```

---

## Create Migrations

For clarity sake, please take note that a migration is nothing more than a SQL file consisting of various SQL operations to be performed on the database.

### Understanding The Migrations

The _heroes_ database now exists. We are now ready to run our migrations. Please take note of the _migrations_ folder that is part of the repo for this example. The _migrations_ folder consists of 7 migrations that are briefly described as follows:

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

[According to the official Flyway documentation](https://flywaydb.org/documentation/migrations#naming), the file name consists of the following parts:

![flyway-naming-convention](https://user-images.githubusercontent.com/33935506/40931818-bc78fb5a-682c-11e8-90ce-cb9f8d0e8c95.png)

* **Prefix:** V for versioned migrations, U for undo migrations, R for repeatable migrations
* **Version:** Underscores (automatically replaced by dots at runtime) separate as many parts as you like (Not for repeatable migrations)
* **Separator:** __ (two underscores)
* **Description:** Underscores (automatically replaced by spaces at runtime) separate the words

### Run Migrations

Finally we get to run our migrations. To run the migrations, we will execute the [_Flyway_ Docker container](https://hub.docker.com/r/boxfuse/flyway/). 

Before running the migration, we need to obtain the IP address of the postgres container as follows:

```bash
docker container inspect -f "{{ .NetworkSettings.Networks.flyway_skynet.IPAddress}}" flyway_pg-dev_1
```

We plug the obtained IP address from above into the command below. In my case, my IP address is _172.18.0.2_

```bash
docker run --rm --network docker_skynet -v $PWD/migrations:/flyway/sql boxfuse/flyway -url=jdbc:postgresql://172.18.0.2:5432/heroes -user=postgres -password=password migrate
```

You should see an output similar to the following output:

![flyway-migration-result](https://user-images.githubusercontent.com/33935506/40933249-2e5510b6-6831-11e8-8df5-526f6c191434.png)

As can be seen from output above, all 7 migrations ran successfully.

Run the following command to see a list of tables in the heroes database:

```bash
docker exec -it $(docker container ls -qf name=pg-dev) psql -U postgres -d heroes -c "SELECT table_schema, table_name FROM INFORMATION_SCHEMA.TABLES WHERE table_schema NOT IN ('pg_catalog', 'information_schema')"
```

You should see a list of tables as follows:

table_schema | table_name
--- | ---
 public       | flyway_schema_history
 hero_data    | hero
 user_data    | user

The database table _flyway\_schema\_history_ contains all the records for the database migrations that took place.

Lastly, log into pgAdmin to view the _flyway\_schema\_history_ table.

* Login

  Navigate to http://localhost:8080 in your browser
  * **email/username:** iamhero@heroes.com
  * **password:** password

  If you're wondering where the pgadmin credentials come from, you can find them specified in the _docker-compose.yml_ file. They're passed in as environment variables.

  ![pgadmin-login](https://user-images.githubusercontent.com/33935506/40934525-5ccbda3e-6835-11e8-8f6d-33efc5eea30c.png)

* Once logged in, you can connect to the PostgreSQL server by adding a connection as follows:

  ![pgadmin-create-server-1](https://user-images.githubusercontent.com/33935506/40934521-5bf26fec-6835-11e8-83dd-ea686c47be22.png)

  ![pgadmin-create-server-2](https://user-images.githubusercontent.com/33935506/40934522-5c29c0b4-6835-11e8-8a9d-b1324f377011.png)

* Open the _flyway\_schema\_history_ table that is located in the public schema of the heroes database.

  ![pgadmin-flyway-table](https://user-images.githubusercontent.com/33935506/40934524-5c976d62-6835-11e8-9b51-892aa1493c8b.png)

---

## Versioning

I use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/drminnaar/guides/tags).

---

## Authors

* **Douglas Minnaar** - *Initial work* - [drminnaar](https://github.com/drminnaar)