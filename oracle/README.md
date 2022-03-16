
# Introduction

This folder contains some content on how to install and use oracle database. 

This project is based on the following foursera course:

[Design and Build a Data Warehouse for Business Intelligence Implementation](https://www.coursera.org/learn/data-warehouse-bi-building/home/welcome)

# Installation

More information on how to configure Oracle Database see the following link:

[Oracle Database Server Docker Image Documentation](https://hub.docker.com/u/scherlac/content/sub-3c266627-e13f-46bd-9746-1dc341f0b7b8)
[Oracle 12c Post Installation Mandatory Steps](https://lalitkumarb.wordpress.com/2014/10/01/mandatory-steps-for-12c-installation/)

``` script
PS C:\Datawarehouse\oracle> .\run.ps1
```

# Connecting to the Database Server Container

The default password to connect to the database with sys user is Oradoc_db1.

## Connecting from within the container

The database server can be connected to by executing SQL*Plus,

$ docker exec -it <Oracle-DB> bash -c "source /home/oracle/.bashrc; sqlplus /nolog"

## Connecting from outside the container

The database server exposes port `1521` for Oracle client connections over SQLNet protocol and port `5500` for Oracle XML DB. SQLPlus or any JDBC client can be used to connect to the database server from outside the container.

To connect from outside the container start the container with `-P` or `-p` option as,

``` script
docker run -d -it --name <Oracle-DB> -P store/oracle/database-enterprise:12.2.0.1
```

option -P indicates the ports are allocated by Docker. The mapped port can be discovered by executing

``` script
docker port <Oracle-DB> 1521/tcp -> 0.0.0.0:<mapped host port>
```

Using this `<mapped host port>` and `<ip-address of host>` create `tnsnames.ora` in the directory pointed to by environment variable `TNS_ADMIN`.

``` ini
ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address of host>)(PORT=<mapped host port>))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))
ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address> of host)(PORT=<mapped host port>))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))
To connect from outside the container using SQL*Plus,
```

``` script
sqlplus sys/Oradoc_db1@ORCLCDB as sysdba
```

# Connecting to the database and creating the first user

We can open an sqlplus session using the command from [here](https://docs.oracle.com/database/121/ADMQS/GUID-DE8A79BD-FAE4-4364-98FF-D2BD992A06E7.htm#ADMQS0361) [notes](https://stackoverflow.com/a/33331148/5770014)

``` script
PS C:\Datawarehouse\oracle> docker exec -it MyOracleDB bash
[oracle@047610bfdce3 /]$ sqlplus / AS SYSDBA

SQL> show con_name;

CON_NAME
------------------------------
CDB$ROOT
SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 ORCLPDB1                       READ WRITE NO

SQL> alter session set container = ORCLPDB1;

Session altered.

SQL> create user Scott identified by tiger; 

User created.

SQL> GRANT DBA to Scott;

Grant succeeded.

[oracle@047610bfdce3 /]$ sqlplus Scott@ORCLPDB1

SQL*Plus: Release 12.2.0.1.0 Production on Thu Mar 10 23:20:49 2022

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Enter password:

Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> show con_name;

CON_NAME
------------------------------
ORCLPDB1

```
