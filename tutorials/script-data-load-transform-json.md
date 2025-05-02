
### Creating the database, table, and virtual warehouse¶
### The following commands create objects specifically for use with this tutorial. When you have completed the tutorial, you can drop the objects.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> create or replace database mydatabase;
+-------------------------------------------+
| status                                    |
|-------------------------------------------|
| Database MYDATABASE successfully created. |
+-------------------------------------------+
1 Row(s) produced. Time Elapsed: 5.379s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> use schema mydatabase.public;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.044s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE TEMPORARY TABLE home_sales (
                                       city STRING,
                                       zip STRING,
                                       state STRING,
                                       type STRING DEFAULT 'Residential',
                                       sale_date timestamp_ntz,
                                       price STRING
                                       );
+----------------------------------------+
| status                                 |
|----------------------------------------|
| Table HOME_SALES successfully created. |
+----------------------------------------+
1 Row(s) produced. Time Elapsed: 0.137s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> create or replace warehouse mywarehouse with
                                       warehouse_size='X-SMALL'
                                       auto_suspend = 120
                                       auto_resume = true
                                       initially_suspended=true;
+---------------------------------------------+
| status                                      |
|---------------------------------------------|
| Warehouse MYWAREHOUSE successfully created. |
+---------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.114s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> use warehouse mywarehouse;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.042s

Note these commands creates temporary table. Temporary tables persist only for the duration of the user session and is not visible to other users.


### Execute the CREATE FILE FORMAT command to create the sf_tut_json_format file format.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE FILE FORMAT sf_tut_json_format
                                       TYPE = JSON;
+------------------------------------------------------+
| status                                               |
|------------------------------------------------------|
| File format SF_TUT_JSON_FORMAT successfully created. |
+------------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.065s


### Execute CREATE STAGE to create the internal sf_tut_stage stage.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE TEMPORARY STAGE sf_tut_stage
                                      FILE_FORMAT = sf_tut_json_format;
+-----------------------------------------------+
| status                                        |
|-----------------------------------------------|
| Stage area SF_TUT_STAGE successfully created. |
+-----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.077s


### Execute the PUT command to upload the JSON file from your local file system to the named stage.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> PUT file:///tmp/load/sales.json @sf_tut_stage AUTO_COMPRESS=TRUE;
+------------+---------------+-------------+-------------+--------------------+--------------------+----------+---------+
| source     | target        | source_size | target_size | source_compression | target_compression | status   | message |
|------------+---------------+-------------+-------------+--------------------+--------------------+----------+---------|
| sales.json | sales.json.gz |         307 |         192 | NONE               | GZIP               | UPLOADED |         |
+------------+---------------+-------------+-------------+--------------------+--------------------+----------+---------+
1 Row(s) produced. Time Elapsed: 5.506s


### Load the sales.json.gz staged data file into the home_sales table.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO home_sales(city, state, zip, sale_date, price)
                                        FROM (SELECT SUBSTR($1:location.state_city,4),
                                                     SUBSTR($1:location.state_city,1,2),
                                                     $1:location.zip,
                                                     to_timestamp_ntz($1:sale_date),
                                                     $1:price
                                              FROM @sf_tut_stage/sales.json.gz t)
                                        ON_ERROR = 'continue';
+----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                       | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| sf_tut_stage/sales.json.gz | LOADED |           3 |           3 |           3 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 1.039s


### Execute the following query to verify data is copied.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * from home_sales;
+------------+-------+-------+-------------+-------------------------+--------+
| CITY       | ZIP   | STATE | TYPE        | SALE_DATE               | PRICE  |
|------------+-------+-------+-------------+-------------------------+--------|
| Lexington  | 40503 | MA    | Residential | 2017-03-05 00:00:00.000 | 275836 |
| Belmont    | 02478 | MA    | Residential | 2017-03-17 00:00:00.000 | 392567 |
| Winchester | 01890 | MA    | Residential | 2017-03-21 00:00:00.000 | 389921 |
+------------+-------+-------+-------------+-------------------------+--------+
3 Row(s) produced. Time Elapsed: 0.308s


### Remove the successfully copied data files
### After you verify that you successfully copied data from your stage into the tables, you can remove data files from the internal stage using the REMOVE command to save on data storage.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> REMOVE @sf_tut_stage/sales.json.gz;
+----------------------------+---------+
| name                       | result  |
|----------------------------+---------|
| sf_tut_stage/sales.json.gz | removed |
+----------------------------+---------+
1 Row(s) produced. Time Elapsed: 0.148s


### Execute the following DROP <object> commands to return your system to its state before you began the tutorial:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP DATABASE IF EXISTS mydatabase;
+----------------------------------+
| status                           |
|----------------------------------|
| MYDATABASE successfully dropped. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.072s


### Execute the following DROP <object> commands to return your system to its state before you began the tutorial:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP WAREHOUSE IF EXISTS mywarehouse;
+-----------------------------------+
| status                            |
|-----------------------------------|
| MYWAREHOUSE successfully dropped. |
+-----------------------------------+
1 Row(s) produced. Time Elapsed: 0.119s


