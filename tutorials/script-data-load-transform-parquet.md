
### Creating the database, table, and virtual warehouse
### The following commands create objects specifically for use with this tutorial. When you have completed the tutorial, you can drop these objects.

jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> create or replace database mydatabase;
+-------------------------------------------+
| status                                    |
|-------------------------------------------|
| Database MYDATABASE successfully created. |
+-------------------------------------------+
1 Row(s) produced. Time Elapsed: 5.370s


jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> use schema mydatabase.public;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.037s


jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC>  create or replace temporary table cities (
                                         continent varchar default null,
                                         country varchar default null,
                                         city variant default null
                                       );

+------------------------------------+
| status                             |
|------------------------------------|
| Table CITIES successfully created. |
+------------------------------------+
1 Row(s) produced. Time Elapsed: 0.177s


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
1 Row(s) produced. Time Elapsed: 0.092s


### x
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> use warehouse mywarehouse;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.040s

Note these commands create a temporary table. Temporary tables persist only for the duration of the user session and is not visible to other users.



### Execute the CREATE FILE FORMAT command to create the sf_tut_parquet_format file format.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE FILE FORMAT sf_tut_parquet_format
                                       TYPE = parquet;
+---------------------------------------------------------+
| status                                                  |
|---------------------------------------------------------|
| File format SF_TUT_PARQUET_FORMAT successfully created. |
+---------------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.064s


### Execute the CREATE STAGE command to create the internal sf_tut_stage stage.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE TEMPORARY STAGE sf_tut_stage
                                     FILE_FORMAT = sf_tut_parquet_format;
+-----------------------------------------------+
| status                                        |
|-----------------------------------------------|
| Stage area SF_TUT_STAGE successfully created. |
+-----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.089s


### Execute the PUT command to upload the parquet file from your local file system to the named stage.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> PUT file:///tmp/load/cities.parquet @sf_tut_stage;
+----------------+----------------+-------------+-------------+--------------------+--------------------+----------+---------+
| source         | target         | source_size | target_size | source_compression | target_compression | status   | message |
|----------------+----------------+-------------+-------------+--------------------+--------------------+----------+---------|
| cities.parquet | cities.parquet |         866 |         880 | PARQUET            | PARQUET            | UPLOADED |         |
+----------------+----------------+-------------+-------------+--------------------+--------------------+----------+---------+
1 Row(s) produced. Time Elapsed: 5.334s


### Copy the cities.parquet staged data file into the CITIES table.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> copy into cities
                                      from (select $1:continent::varchar,
                                                   $1:country:name::varchar,
                                                   $1:country:city::variant
                                           from @sf_tut_stage/cities.parquet);
+-----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                        | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|-----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| sf_tut_stage/cities.parquet | LOADED |           3 |           3 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+-----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 1.070s


### Execute the following query to verify data is copied.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * from cities;
+---------------+---------+-----------------+
| CONTINENT     | COUNTRY | CITY            |
|---------------+---------+-----------------|
| Europe        | France  | [               |
|               |         |   "Paris",      |
|               |         |   "Nice",       |
|               |         |   "Marseilles", |
|               |         |   "Cannes"      |
|               |         | ]               |
| Europe        | Greece  | [               |
|               |         |   "Athens",     |
|               |         |   "Piraeus",    |
|               |         |   "Hania",      |
|               |         |   "Heraklion",  |
|               |         |   "Rethymnon",  |
|               |         |   "Fira"        |
|               |         | ]               |
| North America | Canada  | [               |
|               |         |   "Toronto",    |
|               |         |   "Vancouver",  |
|               |         |   "St. John's", |
|               |         |   "Saint John", |
|               |         |   "Montreal",   |
|               |         |   "Halifax",    |
|               |         |   "Winnipeg",   |
|               |         |   "Calgary",    |
|               |         |   "Saskatoon",  |
|               |         |   "Ottawa",     |
|               |         |   "Yellowknife" |
|               |         | ]               |
+---------------+---------+-----------------+
3 Row(s) produced. Time Elapsed: 0.427s


### Unload the CITIES table into another Parquet file.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> copy into @sf_tut_stage/out/parquet_
                                     from (select continent,
                                                  country,
                                                  c.value::string as city
                                          from cities,
                                               lateral flatten(input => city) c)
                                       file_format = (type = 'parquet')
                                       header = true;
+---------------+-------------+--------------+
| rows_unloaded | input_bytes | output_bytes |
|---------------+-------------+--------------|
|            21 |        1127 |         1127 |
+---------------+-------------+--------------+
1 Row(s) produced. Time Elapsed: 0.348s


[ Note the following ]
The file_format = (type = 'parquet') specifies parquet as the format of the data file on the stage. When the Parquet file type is specified, the COPY INTO <location> command unloads data to a single column by default.
The header=true option directs the command to retain the column names in the output file.
In the nested SELECT query:
The FLATTEN function first flattens the city column array elements into separate columns.
The LATERAL modifier joins the output of the FLATTEN function with information outside of the object - in this example, the continent and country.




### Execute the following query to verify data is copied into staged Parquet file.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> select t.$1 from @sf_tut_stage/out/ t;
+---------------------------------+
| $1                              |
|---------------------------------|
| {                               |
|   "CITY": "Paris",              |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "France"           |
| }                               |
| {                               |
|   "CITY": "Nice",               |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "France"           |
| }                               |
| {                               |
|   "CITY": "Marseilles",         |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "France"           |
| }                               |
| {                               |
|   "CITY": "Cannes",             |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "France"           |
| }                               |
| {                               |
|   "CITY": "Athens",             |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "Greece"           |
| }                               |
| {                               |
|   "CITY": "Piraeus",            |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "Greece"           |
| }                               |
| {                               |
|   "CITY": "Hania",              |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "Greece"           |
| }                               |
| {                               |
|   "CITY": "Heraklion",          |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "Greece"           |
| }                               |
| {                               |
|   "CITY": "Rethymnon",          |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "Greece"           |
| }                               |
| {                               |
|   "CITY": "Fira",               |
|   "CONTINENT": "Europe",        |
|   "COUNTRY": "Greece"           |
| }                               |
| {                               |
|   "CITY": "Toronto",            |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Vancouver",          |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "St. John's",         |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Saint John",         |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Montreal",           |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Halifax",            |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Winnipeg",           |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Calgary",            |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Saskatoon",          |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Ottawa",             |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
| {                               |
|   "CITY": "Yellowknife",        |
|   "CONTINENT": "North America", |
|   "COUNTRY": "Canada"           |
| }                               |
+---------------------------------+
21 Row(s) produced. Time Elapsed: 0.405s


### Remove the successfully copied data files
### After you verify that you successfully copied data from your stage into the tables,
### you can remove data files from the internal stage using the REMOVE command to save on data storage.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> REMOVE @sf_tut_stage/cities.parquet;
+-----------------------------+---------+
| name                        | result  |
|-----------------------------+---------|
| sf_tut_stage/cities.parquet | removed |
+-----------------------------+---------+
1 Row(s) produced. Time Elapsed: 0.100s


### Execute the following DROP <object> commands to return your system to its state before you began the tutorial:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP DATABASE IF EXISTS mydatabase;
+----------------------------------+
| status                           |
|----------------------------------|
| MYDATABASE successfully dropped. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.319s


### Execute the following DROP <object> commands to return your system to its state before you began the tutorial:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP WAREHOUSE IF EXISTS mywarehouse;
+-----------------------------------+
| status                            |
|-----------------------------------|
| MYWAREHOUSE successfully dropped. |
+-----------------------------------+
1 Row(s) produced. Time Elapsed: 0.126s


