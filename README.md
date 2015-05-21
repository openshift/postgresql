PostgreSQL for OpenShift - Docker images
========================================

This repository contains Dockerfiles for PostgreSQL images for OpenShift.
Users can choose between RHEL and CentOS based images.


Versions
---------------
PostgreSQL versions currently provided are:
* postgresql-9.2

RHEL versions currently supported are:
* RHEL7

CentOS versions currently supported are:
* CentOS7


Installation
----------------------
Choose between CentOS7 or RHEL7 based image:

*  **RHEL7 based image**

    To build a RHEL7-based image, you need to run Docker build on a properly
    subscribed RHEL machine.

    ```
    $ git clone https://github.com/openshift/postgresql.git
    $ cd postgresql
    $ make build TARGET=rhel7 VERSION=9.2
    ```

*  **CentOS7 based image**

    This image is available on DockerHub. To download it use:

    ```
    $ docker pull openshift/postgresql-92-centos7
    ```

    To build PostgreSQL image from scratch use:

    ```
    $ git clone https://github.com/openshift/postgresql.git
    $ cd postgresql
    $ make build VERSION=9.2
    ```

**Notice: By omitting the `VERSION` parameter, the build/test action will be performed
on all provided versions of PostgreSQL. Since we are now providing only version `9.2`,
you can omit this parameter.**


Environment variables and volumes
----------------------------------

The image recognizes following environment variables that you can set during
initialization, by passing `-e VAR=VALUE` to the Docker run command.

|    Variable name             |    Description                                 |
| :--------------------------- | ---------------------------------------------- |
|  `POSTGRESQL_USER`           | User name for PostgreSQL account to be created |
|  `POSTGRESQL_PASSWORD`       | Password for the user account                  |
|  `POSTGRESQL_DATABASE`       | Database name                                  |
|  `POSTGRESQL_ADMIN_PASSWORD` | Password for the `postgres` admin account (optional)     |

Following environment variables influence PostgreSQL configuration file. They are all optional.

|    Variable name              |    Description                                                          |    Default
| :---------------------------- | ----------------------------------------------------------------------- | -------------------------------
|  `POSTGRESQL_MAX_CONNECTIONS` | The maximum number of client connections allowed. This also sets the maximum number of prepared transactions. |  100
|  `POSTGRESQL_SHARED_BUFFERS`  | Sets how much memory is dedicated to PostgreSQL to use for caching data |  32M
|  `POSTGRESQL_MAX_PREPARED_TXS | Sets the maximum number of simultaneous "prepared" transactions - XA support. |  0 (XA disabled)

You can also set following mount points by passing `-v /host:/container` flag to docker.

|  Volume mount point      | Description                           |
| :----------------------- | ------------------------------------- |
|  `/var/lib/pgsql/data`   | PostgreSQL database cluster directory |

**Notice: When mouting directory from host into the container, ensure that the mounted
directory has the appropriate permissions and that the owner and group of the directory
matches the user UID or name which is running inside the container.**

Usage
----------------------

We will assume that you are using the `openshift/postgresql-92-centos7` image.
If you want to set only mandatory environment variables and not store the database
in a host directory, you need to execute the following command:

```
$ docker run -d --name postgresql_database -e POSTGRESQL_USER=user -e POSTGRESQL_PASSWORD=pass -e POSTGRESQL_DATABASE=db -p 5432:5432 openshift/postgresql-92-centos7
```

This will create a container named `postgresql_database` running PostgreSQL with
database `db` and user with credentials `user:pass`. Port 5432 will be exposed
and mapped to host. If you want your database to be persistent across container
executions, also add a `-v /host/db/path:/var/lib/pgsql/data` argument. This is
going to be the PostgreSQL database cluster directory.

If the database cluster directory is not initialized, the entrypoint script will
first run [`initdb`](http://www.postgresql.org/docs/9.2/static/app-initdb.html)
and setup necessary database users and passwords. After the database is initialized,
or if it was already present, [`postgres`](http://www.postgresql.org/docs/9.2/static/app-postgres.html)
is executed and will run as PID 1. You can stop the detached container by running
`docker stop postgresql_database`.


PostgreSQL admin account
------------------------
The admin account `postgres` has no password set by default, only allowing local
connections.  You can set it by setting `POSTGRESQL_ADMIN_PASSWORD` environment
variable when initializing your container. This will allow you to login to the
`postgres` account remotely. Local connections will still not require password.


Test
---------------------------------

This repository also provides test framework, which checks basic functionality
of the PostgreSQL image.

User can choose between testing PostgreSQL based on RHEL or CentOS image.

*  **RHEL based image**

    To test a rhel7-based PostgreSQL image, you need to run the test on a properly
    subscribed RHEL machine.

    ```
    $ cd postgresql
    $ make test TARGET=rhel7 VERSION=9.2
    ```

*  **CentOS based image**

    ```
    $ cd postgresql
    $ make test VERSION=9.2
    ```

**Notice: By omitting the `VERSION` parameter, the build/test action will be performed
on all provided versions of PostgreSQL. Since we are now providing only version `9.2`,
you can omit this parameter.**
