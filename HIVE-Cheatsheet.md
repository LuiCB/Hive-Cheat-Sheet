

## Metadata

```
#Selecting a database	
USE database_name;

#Listing databases, show all the databases
SHOW DATABASES [fmt];

#Listing tables in a database	
SHOW TABLES [fmt] [IN DB_NAME];

#Describing the format of a table	
DESCRIBE [EXTENDED] table;	
DESCRIBE (FORMATTED|EXTENDED) table;

#Describing a column
DESCRIBE db.table.column_name;

#Describing the info of a database
DESCRIBE DATABASE database_name;

#Creating a database	
CREATE DATABASE db_name;	
CREATE external DATABASE db_name location '/user/demo/stats';

# Partition
CREATE external DATABASE 
(transdate Date,
transid Date)
db_name location '/user/demo/stats' Partitioned by (store String);
  ## column 'store' doesn't need to exist in create statement
show partitions TABLENAME;

# ORC
Create table states_orc STORED AS ORC TBLPROPERTIES("ORC.COMPRESS"="SNAPPY") as SELECT * FROM STATES;

#Dropping a database	
DROP DATABASE db_name;	DROP DATABASE db_name (CASCADE);
```

## Database properties

```
#Alter Database, There is no way to delete or “unset” a DBPROPERTY.
ALTER DATABASE financials SET DBPROPERTIES ('edited-by' = 'Joe Dba');
```

## Creating Tables
```
# Create a table
CREATE TABLE IF NOT EXISTS mydb.employees (
    name
    salary subordinates deductions
    address
    STRING COMMENT 'Employee name',
    FLOAT COMMENT 'Employee salary',
    ARRAY<STRING> COMMENT 'Names of subordinates',
    MAP<STRING, FLOAT>
COMMENT 'Keys are deductions names, values are percentages', STRUCT<street:STRING, city:STRING, state:STRING, zip:INT> COMMENT 'Home address')
COMMENT 'Description of the table'
TBLPROPERTIES ('creator'='me', 'created_at'='2012-01-02 10:00:00', ...) 
LOCATION '/user/hive/warehouse/mydb.db/employees';

# Create a table with copying the schema of an existing table
CREATE TABLE IF NOT EXISTS mydb.employees2 
LIKE mydb.employees;

# Columns
# add columns
ALTER TABLE log_messages ADD COLUMNS (
    app_name STRING COMMENT 'Application name', 
    session_id LONG COMMENT 'The current session id');

# modify columns
ALTER TABLE log_messages
CHANGE [COLUMN] old-name new-name data-type
[COMMENT] 'type comment here' 
[FIRST / AFTER other_column_name]; # move the column to the first position or after ${other_column_name}

# replace/delete columns
ALTER TABLE log_messages REPLACE COLUMNS (
    name data-type [COMMENT "..."],
    ...);
```

## Table properties
```
"ALTER TABLE" modifies table metadata only.

# Renaming a Table
ALTER TABLE a RENAME TO b;
ALTER TABLE log_messages SET TBLPROPERTIES ('key' = 'value');

# Adding, Modifying and Dropping a table partition
ALTER TABLE table ADD [IF NOT EXISTS] 
PARTITION (...) LOCATION 'new/path/to/the/file'
...;

ALTER TABLE log_messages PARTITION(year = 2011, month = 12, day = 2) 
SET LOCATION 's3n://ourbucket/logs/2011/01/02'; # doesn't move or delete the original partition

ALTER TABLE log_messages DROP IF EXISTS PARTITION(year = 2011, month = 12, day = 2);

# Advanced
# todo: alter storage properties, SerDe properties..

```

## Hive SQL
```
# SELECT ... FROM cluases
SELECT function(columnM), columnA as A, array[0], map['key'], struct.attribute 
FROM table as T 
WHERE conditions
LIMIT num

WITH a as ( nested SELECT Statements),
     ...
SELECT a.A, ... FROM a, ...;

# CASE ... WHEN ... THEN 
SELECT a, b, 
    CASE 
      WHEN c ... THEN ... 
      ...
      ELSE ...
    END AS new_c FROM table;

# LIKE 
 SQL

# RLIKE or REGEXP 
follow java regex comprison
GROUP BY ... 
    HIVING ...;

# INNER JOIN
A JOIN B ON [conditions] [JOIN C ON ...] 

# LEFT OUTTER JOIN
A LEFT OUTER JOIN B ON [conditions] [LEFT OUTER JOIN C ON ...]
[WHERE ...;]
>  note that the JOIN clause is evaluated first, then the WHERE clause.
>  pay attention to the predicate expressions in the WHERE clause, especially for the
>  right-hand side table.

# RIGHT OUTER JOIN
A RIGHT OUTER JOIN B ON [conditions] [RIGHT OUTER JOIN C ON ...]

# FULL OUTER JOIN
A FULL OUTER JOIN B ON [conditions] [FULL OUTER JOIN C ON ...]
>  full-outer join returns all records from all tables that match the WHERE clause. 
>  NULL is used for fields in missing records in either table.

# LEFT SEMI-JOIN
SELET A.a, A.b, A.c 
FROM A LEFT SEMI JOIN B 
ON [conditions]

# pay attention to Cartesian Product JOINs because HIVE process JOIN before WHERE

# Map-side JOINs
> before Hive v0.7 
SELECT /*+ MAPJOIN(d) */ ...

> Hive v0.7+
set hive.auto.convert.join=true;
set hive.mapjoin.smalltable.filesize=25000000 (in bytes)

# ORDER BY & SORT BY
ORDER|SORT BY ... ASC|DESC;
> ORDER BY, one reducer
> SORT BY, get data shards sorted by each reducer

# DISTRIBUTE BY with SORT BY, CLUSTER BY
DISTRIBUTE BY ...
SORT BY ...[DESC];

=> CLUSTER BY ...;

# UNION
UNION [DISTINCT|ALL]
UNION DISTINCT = UNION

by default, UNION will sort the rows by columns, ACS
```


## Command line

```
#Run Query	
hive -e 'select a.col from tab1 a'

#Run Query Silent Mode	
hive -S -e 'select a.col from tab1 a'

#Set Hive Config Variables	
hive -e 'select a.col from tab1 a' -hiveconf hive.root.logger=DEBUG,console

#Use Initialization Script	
hive -i initialize.sql

#Run Non-Interactive Script	
hive -f script.sql


# Run script inside shell
source file_name

# Run ls (dfs) commands 
dfs –ls /user

# Run ls (bash command) from shell 
!ls

# Set configuration variables 
set mapred.reduce.tasks=32

# TAB auto completion 
set hive.<TAB>

# Show all variables starting with hive 
set

# Revert all variables 
reset

# Add jar to distributed cache 
add jar jar_path

# Show all jars in distributed cache 
list jars

# Delete jar from distributed cache 
delete jar jar_name

```

## Config your $HOME/.hiverc 

```
# todo: set mpa-side joins, 

```

## Config the hadoop (YARN)
