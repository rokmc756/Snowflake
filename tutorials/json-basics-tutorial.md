
### Creating the database, table, warehouse, and external stage
# ( Execute the following statements to create a database, a table, a virtual warehouse, and an external stage needed for this tutorial. After you complete the tutorial, you can drop these objects. )

jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE DATABASE mydatabase;
+-------------------------------------------+
| status                                    |
|-------------------------------------------|
| Database MYDATABASE successfully created. |
+-------------------------------------------+
1 Row(s) produced. Time Elapsed: 5.402s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> USE SCHEMA mydatabase.public;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.039s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE TABLE raw_source (
                                       SRC VARIANT);
+----------------------------------------+
| status                                 |
|----------------------------------------|
| Table RAW_SOURCE successfully created. |
+----------------------------------------+
1 Row(s) produced. Time Elapsed: 0.114s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE WAREHOUSE mywarehouse WITH
                                       WAREHOUSE_SIZE='X-SMALL'
                                       AUTO_SUSPEND = 120
                                       AUTO_RESUME = TRUE
                                       INITIALLY_SUSPENDED=TRUE;
+---------------------------------------------+
| status                                      |
|---------------------------------------------|
| Warehouse MYWAREHOUSE successfully created. |
+---------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.087s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> USE WAREHOUSE mywarehouse;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.038s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE STAGE my_stage
                                       URL = 's3://snowflake-docs/tutorials/json';


+-------------------------------------------+
| status                                    |
|-------------------------------------------|
| Stage area MY_STAGE successfully created. |
+-------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.831s


Note the following:
The CREATE DATABASE statement creates a database. The database automatically includes a schema named ‘public’.
The USE SCHEMA statement specifies an active database and schema for the current user session. Specifying a database now enables you to perform your work in this database without having to provide the name each time it is requested.
The CREATE TABLE statement creates a target table for JSON data.
The CREATE WAREHOUSE statement creates an initially suspended warehouse. The statement also sets AUTO_RESUME = true, which starts the warehouse automatically when you execute SQL statements that require compute resources. The USE WAREHOUSE statement specifies the warehouse you created as the active warehouse for the current user session.
The CREATE STAGE statement creates an external stage that points to the S3 bucket containing the sample file for this tutorial.



### Execute COPY INTO <table> to load your staged data into the target RAW_SOURCE table.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO raw_source
                                       FROM @my_stage/server/2.6/2016/07/15/15
                                       FILE_FORMAT = (TYPE = JSON);
+--------------------------------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                                                                           | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|--------------------------------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| s3://snowflake-docs/tutorials/json/server/2.6/2016/07/15/15/json_tutorial.json | LOADED |           1 |           1 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+--------------------------------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 2.724s


### Execute a SELECT query to verify the data is copied successfully.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * FROM raw_source;
+-----------------------------------------------------------------------------------+
| SRC                                                                               |
|-----------------------------------------------------------------------------------|
| {                                                                                 |
|   "device_type": "server",                                                        |
|   "events": [                                                                     |
|     {                                                                             |
|       "f": 83,                                                                    |
|       "rv": "15219.64,783.63,48674.48,84679.52,27499.78,2178.83,0.42,74900.19",   |
|       "t": 1437560931139,                                                         |
|       "v": {                                                                      |
|         "ACHZ": 42869,                                                            |
|         "ACV": 709489,                                                            |
|         "DCA": 232,                                                               |
|         "DCV": 62287,                                                             |
|         "ENJR": 2599,                                                             |
|         "ERRS": 205,                                                              |
|         "MXEC": 487,                                                              |
|         "TMPI": 9                                                                 |
|       },                                                                          |
|       "vd": 54,                                                                   |
|       "z": 1437644222811                                                          |
|     },                                                                            |
|     {                                                                             |
|       "f": 1000083,                                                               |
|       "rv": "8070.52,54470.71,85331.27,9.10,70825.85,65191.82,46564.53,29422.22", |
|       "t": 1437036965027,                                                         |
|       "v": {                                                                      |
|         "ACHZ": 6953,                                                             |
|         "ACV": 346795,                                                            |
|         "DCA": 250,                                                               |
|         "DCV": 46066,                                                             |
|         "ENJR": 9033,                                                             |
|         "ERRS": 615,                                                              |
|         "MXEC": 0,                                                                |
|         "TMPI": 112                                                               |
|       },                                                                          |
|       "vd": 626,                                                                  |
|       "z": 1437660796958                                                          |
|     }                                                                             |
|   ],                                                                              |
|   "version": 2.6                                                                  |
| }                                                                                 |
+-----------------------------------------------------------------------------------+


### Retrieve device_type.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT src:device_type
                                       FROM raw_source;
+-----------------+
| SRC:DEVICE_TYPE |
|-----------------|
| "server"        |
+-----------------+
1 Row(s) produced. Time Elapsed: 0.321s


### Retrieve the device_type value without the quotes.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT src:device_type::string AS device_type
                                       FROM raw_source;
+-------------+
| DEVICE_TYPE |
|-------------|
| server      |
+-------------+
1 Row(s) produced. Time Elapsed: 0.136s


### To retrieve these nested keys, you can use the FLATTEN function. The function flattens the events into separate rows.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT
                                       value:f::number
                                       FROM
                                         raw_source
                                       , LATERAL FLATTEN( INPUT => SRC:events );
+-----------------+
| VALUE:F::NUMBER |
|-----------------|
|              83 |
|         1000083 |
+-----------------+
2 Row(s) produced. Time Elapsed: 0.359s


### Query the data for each event:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT src:device_type::string,
                                         src:version::String,
                                         VALUE
                                     FROM
                                         raw_source,
                                         LATERAL FLATTEN( INPUT => SRC:events );
+-------------------------+---------------------+-------------------------------------------------------------------------------+
| SRC:DEVICE_TYPE::STRING | SRC:VERSION::STRING | VALUE                                                                         |
|-------------------------+---------------------+-------------------------------------------------------------------------------|
| server                  | 2.6                 | {                                                                             |
|                         |                     |   "f": 83,                                                                    |
|                         |                     |   "rv": "15219.64,783.63,48674.48,84679.52,27499.78,2178.83,0.42,74900.19",   |
|                         |                     |   "t": 1437560931139,                                                         |
|                         |                     |   "v": {                                                                      |
|                         |                     |     "ACHZ": 42869,                                                            |
|                         |                     |     "ACV": 709489,                                                            |
|                         |                     |     "DCA": 232,                                                               |
|                         |                     |     "DCV": 62287,                                                             |
|                         |                     |     "ENJR": 2599,                                                             |
|                         |                     |     "ERRS": 205,                                                              |
|                         |                     |     "MXEC": 487,                                                              |
|                         |                     |     "TMPI": 9                                                                 |
|                         |                     |   },                                                                          |
|                         |                     |   "vd": 54,                                                                   |
|                         |                     |   "z": 1437644222811                                                          |
|                         |                     | }                                                                             |
| server                  | 2.6                 | {                                                                             |
|                         |                     |   "f": 1000083,                                                               |
|                         |                     |   "rv": "8070.52,54470.71,85331.27,9.10,70825.85,65191.82,46564.53,29422.22", |
|                         |                     |   "t": 1437036965027,                                                         |
|                         |                     |   "v": {                                                                      |
|                         |                     |     "ACHZ": 6953,                                                             |
|                         |                     |     "ACV": 346795,                                                            |
|                         |                     |     "DCA": 250,                                                               |
|                         |                     |     "DCV": 46066,                                                             |
|                         |                     |     "ENJR": 9033,                                                             |
|                         |                     |     "ERRS": 615,                                                              |
|                         |                     |     "MXEC": 0,                                                                |
|                         |                     |     "TMPI": 112                                                               |
|                         |                     |   },                                                                          |
|                         |                     |   "vd": 626,                                                                  |
|                         |                     |   "z": 1437660796958                                                          |
|                         |                     | }                                                                             |
+-------------------------+---------------------+-------------------------------------------------------------------------------+
2 Row(s) produced. Time Elapsed: 0.073s


### Use a CREATE TABLE AS SELECT statement to store the preceding query result in a table
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE TABLE flattened_source AS
                                       SELECT
                                         src:device_type::string AS device_type,
                                         src:version::string     AS version,
                                         VALUE                   AS src
                                       FROM
                                         raw_source,
                                         LATERAL FLATTEN( INPUT => SRC:events );
+----------------------------------------------+
| status                                       |
|----------------------------------------------|
| Table FLATTENED_SOURCE successfully created. |
+----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.470s


### Query the resulting table
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * FROM flattened_source;
+-------------+---------+-------------------------------------------------------------------------------+
| DEVICE_TYPE | VERSION | SRC                                                                           |
|-------------+---------+-------------------------------------------------------------------------------|
| server      | 2.6     | {                                                                             |
|             |         |   "f": 83,                                                                    |
|             |         |   "rv": "15219.64,783.63,48674.48,84679.52,27499.78,2178.83,0.42,74900.19",   |
|             |         |   "t": 1437560931139,                                                         |
|             |         |   "v": {                                                                      |
|             |         |     "ACHZ": 42869,                                                            |
|             |         |     "ACV": 709489,                                                            |
|             |         |     "DCA": 232,                                                               |
|             |         |     "DCV": 62287,                                                             |
|             |         |     "ENJR": 2599,                                                             |
|             |         |     "ERRS": 205,                                                              |
|             |         |     "MXEC": 487,                                                              |
|             |         |     "TMPI": 9                                                                 |
|             |         |   },                                                                          |
|             |         |   "vd": 54,                                                                   |
|             |         |   "z": 1437644222811                                                          |
|             |         | }                                                                             |
| server      | 2.6     | {                                                                             |
|             |         |   "f": 1000083,                                                               |
|             |         |   "rv": "8070.52,54470.71,85331.27,9.10,70825.85,65191.82,46564.53,29422.22", |
|             |         |   "t": 1437036965027,                                                         |
|             |         |   "v": {                                                                      |
|             |         |     "ACHZ": 6953,                                                             |
|             |         |     "ACV": 346795,                                                            |
|             |         |     "DCA": 250,                                                               |
|             |         |     "DCV": 46066,                                                             |
|             |         |     "ENJR": 9033,                                                             |
|             |         |     "ERRS": 615,                                                              |
|             |         |     "MXEC": 0,                                                                |
|             |         |     "TMPI": 112                                                               |
|             |         |   },                                                                          |
|             |         |   "vd": 626,                                                                  |
|             |         |   "z": 1437660796958                                                          |
|             |         | }                                                                             |
+-------------+---------+-------------------------------------------------------------------------------+
2 Row(s) produced. Time Elapsed: 0.109s



### The following CREATE TABLE AS SELECT statement creates a new table named events with the event object keys stored in separate columns.
### Each value is cast to a data type that is appropriate for the value, using a double-colon (::) followed by the type.
### If you omit the casting, the column assumes the VARIANT data type, which can hold any value:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> create or replace table events as
                                       select
                                         src:device_type::string                             as device_type
                                       , src:version::string                                 as version
                                       , value:f::number                                     as f
                                       , value:rv::variant                                   as rv
                                       , value:t::number                                     as t
                                       , value:v.ACHZ::number                                as achz
                                       , value:v.ACV::number                                 as acv
                                       , value:v.DCA::number                                 as dca
                                       , value:v.DCV::number                                 as dcv
                                       , value:v.ENJR::number                                as enjr
                                       , value:v.ERRS::number                                as errs
                                       , value:v.MXEC::number                                as mxec
                                       , value:v.TMPI::number                                as tmpi
                                       , value:vd::number                                    as vd
                                       , value:z::number                                     as z
                                       from
                                         raw_source
                                       , lateral flatten ( input => SRC:events );


+------------------------------------+
| status                             |
|------------------------------------|
| Table EVENTS successfully created. |
+------------------------------------+
1 Row(s) produced. Time Elapsed: 0.784s


### The statement flattens the nested data in the EVENTS.SRC:V key, adding a separate column for each value.
### The statement outputs a row for each key/value pair. The following output shows the first two records in the new events table:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * FROM events;
+-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------+
| DEVICE_TYPE | VERSION |       F | RV                                                                   |             T |  ACHZ |    ACV | DCA |   DCV | ENJR | ERRS | MXEC | TMPI |  VD |             Z |
|-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------|
| server      | 2.6     |      83 | "15219.64,783.63,48674.48,84679.52,27499.78,2178.83,0.42,74900.19"   | 1437560931139 | 42869 | 709489 | 232 | 62287 | 2599 |  205 |  487 |    9 |  54 | 1437644222811 |
| server      | 2.6     | 1000083 | "8070.52,54470.71,85331.27,9.10,70825.85,65191.82,46564.53,29422.22" | 1437036965027 |  6953 | 346795 | 250 | 46066 | 9033 |  615 |    0 |  112 | 626 | 1437660796958 |
+-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------+
2 Row(s) produced. Time Elapsed: 0.160s


### Add the primary key constraint to the EVENTS table
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> ALTER TABLE events ADD CONSTRAINT pk_DeviceType PRIMARY KEY (device_type, rv);
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.070s


### Insert a new JSON event record into the RAW_SOURCE table
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> insert into raw_source
                                       select
                                       PARSE_JSON ('{
                                         "device_type": "cell_phone",
                                         "events": [
                                           {
                                             "f": 79,
                                             "rv": "786954.67,492.68,3577.48,40.11,343.00,345.8,0.22,8765.22",
                                             "t": 5769784730576,
                                             "v": {
                                               "ACHZ": 75846,
                                               "ACV": 098355,
                                               "DCA": 789,
                                               "DCV": 62287,
                                               "ENJR": 2234,
                                               "ERRS": 578,
                                               "MXEC": 999,
                                               "TMPI": 9
                                             },
                                             "vd": 54,
                                             "z": 1437644222811
                                           }
                                         ],
                                         "version": 3.2
                                       }');
+-------------------------+
| number of rows inserted |
|-------------------------|
|                       1 |
+-------------------------+
1 Row(s) produced. Time Elapsed: 0.324s


### Insert the new record that you added to the RAW_SOURCE table into the EVENTS table based on a comparison of the primary key values
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> insert into events
                                     select
                                           src:device_type::string
                                         , src:version::string
                                         , value:f::number
                                         , value:rv::variant
                                         , value:t::number
                                         , value:v.ACHZ::number
                                         , value:v.ACV::number
                                         , value:v.DCA::number
                                         , value:v.DCV::number
                                         , value:v.ENJR::number
                                         , value:v.ERRS::number
                                         , value:v.MXEC::number
                                         , value:v.TMPI::number
                                         , value:vd::number
                                         , value:z::number
                                         from
                                           raw_source
                                         , lateral flatten( input => src:events )
                                         where not exists
                                         (select 'x'
                                           from events
                                           where events.device_type = src:device_type
                                           and events.rv = value:rv);
+-------------------------+
| number of rows inserted |
|-------------------------|
|                       1 |
+-------------------------+
1 Row(s) produced. Time Elapsed: 0.467s


### Querying the EVENTS table shows the added row
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> select * from EVENTS;
+-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------+
| DEVICE_TYPE | VERSION |       F | RV                                                                   |             T |  ACHZ |    ACV | DCA |   DCV | ENJR | ERRS | MXEC | TMPI |  VD |             Z |
|-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------|
| server      | 2.6     |      83 | "15219.64,783.63,48674.48,84679.52,27499.78,2178.83,0.42,74900.19"   | 1437560931139 | 42869 | 709489 | 232 | 62287 | 2599 |  205 |  487 |    9 |  54 | 1437644222811 |
| server      | 2.6     | 1000083 | "8070.52,54470.71,85331.27,9.10,70825.85,65191.82,46564.53,29422.22" | 1437036965027 |  6953 | 346795 | 250 | 46066 | 9033 |  615 |    0 |  112 | 626 | 1437660796958 |
| cell_phone  | 3.2     |      79 | "786954.67,492.68,3577.48,40.11,343.00,345.8,0.22,8765.22"           | 5769784730576 | 75846 |  98355 | 789 | 62287 | 2234 |  578 |  999 |    9 |  54 | 1437644222811 |
+-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------+
3 Row(s) produced. Time Elapsed: 0.187s


### Insert a new JSON event record into the RAW_SOURCE table
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> insert into raw_source
                                       select
                                       parse_json ('{
                                         "device_type": "web_browser",
                                         "events": [
                                           {
                                             "f": 79,
                                             "rv": "122375.99,744.89,386.99,12.45,78.08,43.7,9.22,8765.43",
                                             "t": 5769784730576,
                                             "v": {
                                               "ACHZ": 768436,
                                               "ACV": 9475,
                                               "DCA": 94835,
                                               "DCV": 88845,
                                               "ENJR": 8754,
                                               "ERRS": 567,
                                               "MXEC": 823,
                                               "TMPI": 0
                                             },
                                             "vd": 55,
                                             "z": 8745598047355
                                           }
                                         ],
                                         "version": 8.7
                                       }');
+-------------------------+
| number of rows inserted |
|-------------------------|
|                       1 |
+-------------------------+
1 Row(s) produced. Time Elapsed: 0.317s


### Insert the new record in the RAW_SOURCE table into the EVENTS table based on a comparison of all repeating key values
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> insert into events
                                     select
                                           src:device_type::string
                                         , src:version::string
                                         , value:f::number
                                         , value:rv::variant
                                         , value:t::number
                                         , value:v.ACHZ::number
                                         , value:v.ACV::number
                                         , value:v.DCA::number
                                         , value:v.DCV::number
                                         , value:v.ENJR::number
                                         , value:v.ERRS::number
                                         , value:v.MXEC::number
                                         , value:v.TMPI::number
                                         , value:vd::number
                                         , value:z::number
                                         from
                                           raw_source
                                         , lateral flatten( input => src:events )
                                         where not exists
                                         (select 'x'
                                           from events
                                           where events.device_type = src:device_type
                                           and events.version = src:version
                                           and events.f = value:f
                                           and events.rv = value:rv
                                           and events.t = value:t
                                           and events.achz = value:v.ACHZ
                                           and events.acv = value:v.ACV
                                           and events.dca = value:v.DCA
                                           and events.dcv = value:v.DCV
                                           and events.enjr = value:v.ENJR
                                           and events.errs = value:v.ERRS
                                           and events.mxec = value:v.MXEC
                                           and events.tmpi = value:v.TMPI
                                           and events.vd = value:vd
                                           and events.z = value:z);

+-------------------------+
| number of rows inserted |
|-------------------------|
|                       1 |
+-------------------------+
1 Row(s) produced. Time Elapsed: 0.493s


### Querying the EVENTS table shows the added row
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> select * from EVENTS;
+-------------+---------+---------+----------------------------------------------------------------------+---------------+--------+--------+-------+-------+------+------+------+------+-----+---------------+
| DEVICE_TYPE | VERSION |       F | RV                                                                   |             T |   ACHZ |    ACV |   DCA |   DCV | ENJR | ERRS | MXEC | TMPI |  VD |             Z |
|-------------+---------+---------+----------------------------------------------------------------------+---------------+--------+--------+-------+-------+------+------+------+------+-----+---------------|
| server      | 2.6     |      83 | "15219.64,783.63,48674.48,84679.52,27499.78,2178.83,0.42,74900.19"   | 1437560931139 |  42869 | 709489 |   232 | 62287 | 2599 |  205 |  487 |    9 |  54 | 1437644222811 |
| server      | 2.6     | 1000083 | "8070.52,54470.71,85331.27,9.10,70825.85,65191.82,46564.53,29422.22" | 1437036965027 |   6953 | 346795 |   250 | 46066 | 9033 |  615 |    0 |  112 | 626 | 1437660796958 |
| cell_phone  | 3.2     |      79 | "786954.67,492.68,3577.48,40.11,343.00,345.8,0.22,8765.22"           | 5769784730576 |  75846 |  98355 |   789 | 62287 | 2234 |  578 |  999 |    9 |  54 | 1437644222811 |
| web_browser | 8.7     |      79 | "122375.99,744.89,386.99,12.45,78.08,43.7,9.22,8765.43"              | 5769784730576 | 768436 |   9475 | 94835 | 88845 | 8754 |  567 |  823 |    0 |  55 | 8745598047355 |
+-------------+---------+---------+----------------------------------------------------------------------+---------------+--------+--------+-------+-------+------+------+------+------+-----+---------------+
4 Row(s) produced. Time Elapsed: 0.154s


### Execute the following DROP <object> commands to return your system to its state before you began the tutorial:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP DATABASE IF EXISTS mydatabase;
+----------------------------------+
| status                           |
|----------------------------------|
| MYDATABASE successfully dropped. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.085s


### Execute the following DROP <object> commands to return your system to its state before you began the tutorial:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP WAREHOUSE IF EXISTS mywarehouse;
+-----------------------------------+
| status                            |
|-----------------------------------|
| MYWAREHOUSE successfully dropped. |
+-----------------------------------+
1 Row(s) produced. Time Elapsed: 0.102s


