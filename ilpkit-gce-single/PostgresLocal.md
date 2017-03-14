#Installing PostgreSQL (Local Edition)
This guide details how to intall and configure [PostgreSQL](https://www.postgresql.org/) to support your ILP Kit installation.  It is a continuation of a broader installation guide, [found here](../README.md).  


## Table of Contents

- [Install PostgreSQL](#install-postgresql)
- [Configure PostgreSQL](#configure-postgresql)
- [Data Required to Complete the Installation](#data-required-to-complete-the-installation)

## Install PostgreSQL

First, install Postgres via the following (be sure you're still using the `root` user):

``` sh 
root$ apt-get install libssl-dev python build-essential libpq-dev postgresql postgresql-contrib
```

> The `postgres-9.6` package creates a `postgres` user when it is `apt-get installed`.

Next, start the Postgres service as root (postgres has functionality to actuall run the service as the `postgres` user):

``` sh
  me$ sudo service postgresql start
```

## Configure PostgreSQL
In order to utilize Postgres, we need to configure it for ILP-Kit.  To do that, first sudo as the `postgres` user:

``` sh
  me$ sudo su - postgres
```

In order to use Postgres, you'll need a user/password on postgres, as well as a database named `ilpkit` (or a name of your choosing).  Issue the following commands to 1) create a PostgreSQL role named `ilpkit` with `ilpkit` as the password and 2) create a database `ilpkit` owned by the `ilpkit` role:

``` sh
  postgres$ psql --command "CREATE USER ilpkit WITH SUPERUSER PASSWORD 'ilpkit';" &&\
        createdb -O ilpkit ilpkit
```

> Note: For extra security, you might want to consider using some other secret value as the password for the Postgres `ilpkit` user.  However, keep in mind that, by default, the database is not accessible outside of the VM, and the VM is not accessible without your Google Account credentials, so this is probably not a necessary thing to worry about.  However, if for some reason an attacker were to gain access to your machine or database process, using a stronger password here _might_ offer you some additional protection. 

Next, make sure your postgres version is at least 9.5 by running:

``` sh
  postgres$ psql --version 
```

> Using the commands in this guide will install PostgreSQL 9.6 or above.

Finally, exit the `postgres` user role so you're now running as your Google Account:

``` sh
  postgres$ exit
```

## Data Required to Complete the Installation
For later, be sure to take note of the following pieces of data that you have configured.  You'll need them later when you complete the ILP Kit configuration steps in the main [installation guide](../README.md).


* **Posgres DB URI**: `postgres://ilpkit:ilpkit@localhost:ilpkit`

The CLI provides example values, but I'll also put the configuration I'm using.

- Posgres DB URI  
  This is a string that you'll enter into your ILPKit's `env.list` configuration file in order to tell ILP Kit where to connect to your database.  It consists of the following information: `postgres://[USER]:[PASSWORD]@[HOST]:[POSTGRES_ROLE]`.  Using the information specified in this guide, a concrete example would be:  `postgres://ilpkit:ilpkit@localhost:ilpkit`