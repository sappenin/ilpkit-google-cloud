# Installing PostgreSQL (Google Cloud SQL Edition)

This guide details how to intall and configure [PostgreSQL](https://www.postgresql.org/) to support your ILP Kit installation.  It is a continuation of a broader installation guide, [found here](./README.md).  

## Table of Contents

- [Install PostgreSQL](#install-postgresql)
- [Configure PostgreSQL](#configure-postgresql)
- [Data Required to Complete the ILPKit Installation](#data-required-to-complete-the-ilpkit-installation)

## Initialize CloudSQL VM
In order to utilize Google Cloud SQL, you'll need to create an instance using the following instructions:

1. Navigate to the CloudSQL instances page [here](https://console.cloud.google.com/sql/instances).
1. Click the `Create Instance` button to start the new-instance configuration wizard.
1. Select `PostgreSQL version 9.6`, and click `Next`.
1. Type an `Instance Id` into the text field.  This will be used to identify your database instance in various GCP console screens.
1. Choose a `Location` and a `Machine Type`.  For exploratory or initial-use, the lowest number of cores and least amount of memory should suffice.  
1. To save money, choose `HDD` as your storage type.  It will be a bit slower in terms of performance, but you probably won't notice, especially at first.
1. Accept the rest of the default settings, and click the `Create` button.

> Be sure to take note of the password you generated for the `postgres` user.  You'll need it to connect to your database later in this guide.

Once your CloudSQL instance is up and running, take note of the `IP Address` that was assigned.  We'll use that later in this guide as well.

## Configure Network Access
Follow [the instructions here](https://cloud.google.com/sql/docs/postgres/connect-compute-engine) to connect to your CloudSQL instance from your ILP-Kit VM.

## Create ilpkit Database User
In order to utilize Postgres, we need to configure it for ILP-Kit.  Follow the instructions here to create a new user called `ilpkit` in your database instance:  [Create and Manage Users](https://cloud.google.com/sql/docs/postgres/create-manage-users).

## Create ilpkit Database
Next, create a database called `ilpkit` in your CloudSQL instance by [following the instructions here](https://cloud.google.com/sql/docs/postgres/create-manage-databases).

## Data Required to Complete the Installation
For later, be sure to take note of the following pieces of data that you have configured.  You'll need them  when you complete the ILP Kit configuration steps in the main [installation guide](../README.md).

* **Posgres DB URI**  
  This is a string that you'll enter into your ILPKit's `env.list` configuration file in order to tell ILPKit where to connect to your database. 
  * This value consists of the following information: `postgres://[USER]:[PASSWORD]@[HOST]:[POSTGRES_ROLE]`.  
  * Using the information specified in this guide, a concrete example would be:  `postgres://ilpkit:ilpkit@100.10.1.1/ilpkit`
  
> Security Note: Don't use the password `ilpkit` for your Postgres `ilpkit` role.  Instead, choose a strong password that is not easily guessable.  By default, only your VM has access to your CloudSQL instance, but better safe than sorry.