
### X
```sh
export SNOWSQL_PWD=

snowsql -a gnsjdpk-tj82189 -u jomoon
```


### X
```sql
* SnowSQL * v1.3.3
Type SQL statements or !help
jomoon#COMPUTE_WH@(no database).(no schema)> ^D
Goodbye!


jomoon#COMPUTE_WH@(no database).(no schema)>CREATE OR REPLACE DATABASE sf_tuts;
+----------------------------------------+
| status                                 |
|----------------------------------------|
| Database SF_TUTS successfully created. |
+----------------------------------------+
1 Row(s) produced. Time Elapsed: 0.115s
```


### X
```sql
jomoon#COMPUTE_WH@SF_TUTS.PUBLIC>SELECT CURRENT_DATABASE(), CURRENT_SCHEMA();
+--------------------+------------------+
| CURRENT_DATABASE() | CURRENT_SCHEMA() |
|--------------------+------------------|
| SF_TUTS            | PUBLIC           |
+--------------------+------------------+
1 Row(s) produced. Time Elapsed: 0.036s
```


### X
```sql
jomoon#COMPUTE_WH@SF_TUTS.PUBLIC>CREATE OR REPLACE TABLE emp_basic (
                                    first_name STRING ,
                                    last_name STRING ,
                                    email STRING ,
                                    streetaddress STRING ,
                                    city STRING ,
                                    start_date DATE
                                    );


+---------------------------------------+
| status                                |
|---------------------------------------|
| Table EMP_BASIC successfully created. |
+---------------------------------------+
1 Row(s) produced. Time Elapsed: 0.131s
```


### X
```sql
jomoon#COMPUTE_WH@SF_TUTS.PUBLIC>CREATE OR REPLACE WAREHOUSE sf_tuts_wh WITH
                                    WAREHOUSE_SIZE='X-SMALL'
                                    AUTO_SUSPEND = 180
                                    AUTO_RESUME = TRUE
                                    INITIALLY_SUSPENDED=TRUE;
+--------------------------------------------+
| status                                     |
|--------------------------------------------|
| Warehouse SF_TUTS_WH successfully created. |
+--------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.093s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>SELECT CURRENT_WAREHOUSE();
+---------------------+
| CURRENT_WAREHOUSE() |
|---------------------|
| SF_TUTS_WH          |
+---------------------+
1 Row(s) produced. Time Elapsed: 0.058s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>PUT file:///home/jomoon/Snowflake/data/*.csv @sf_tuts.public.%emp_basic;
+--------------------------+-----------------------------+-------------+-------------+--------------------+--------------------+----------+---------+
| source                   | target                      | source_size | target_size | source_compression | target_compression | status   | message |
|--------------------------+-----------------------------+-------------+-------------+--------------------+--------------------+----------+---------|
| sample_customer_data.csv | sample_customer_data.csv.gz |          94 |         112 | NONE               | GZIP               | UPLOADED |         |
| somedata.csv             | somedata.csv.gz             |          94 |         128 | NONE               | GZIP               | UPLOADED |         |
+--------------------------+-----------------------------+-------------+-------------+--------------------+--------------------+----------+---------+
2 Row(s) produced. Time Elapsed: 20.766s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>LIST @sf_tuts.public.%emp_basic;
+-----------------------------+------+----------------------------------+------------------------------+
| name                        | size | md5                              | last_modified                |
|-----------------------------+------+----------------------------------+------------------------------|
| sample_customer_data.csv.gz |  112 | 6636abfc3071b4c9fb956c357c0b86dd | Thu, 1 May 2025 16:28:23 GMT |
| somedata.csv.gz             |  128 | 72f31297bb8bc0477f216e9b507990f9 | Thu, 1 May 2025 16:28:13 GMT |
+-----------------------------+------+----------------------------------+------------------------------+
2 Row(s) produced. Time Elapsed: 0.136s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>COPY INTO emp_basic
                                   FROM @%emp_basic
                                   FILE_FORMAT = (type = csv field_optionally_enclosed_by='"')
                                   PATTERN = '.*.csv.gz'
                                   ON_ERROR = 'skip_file';
+-----------------------------+-------------+-------------+-------------+-------------+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+--------------------------------+
| file                        | status      | rows_parsed | rows_loaded | error_limit | errors_seen | first_error                                                                                                                                                        | first_error_line | first_error_character | first_error_column_name        |
|-----------------------------+-------------+-------------+-------------+-------------+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+--------------------------------|
| somedata.csv.gz             | LOAD_FAILED |           4 |           0 |           1 |           4 | Number of columns in file (4) does not match that of the corresponding table (6), use file format option error_on_column_count_mismatch=false to ignore this error |                2 |                     1 | "EMP_BASIC"["STREETADDRESS":4] |
| sample_customer_data.csv.gz | LOAD_FAILED |           5 |           0 |           1 |           5 | Number of columns in file (2) does not match that of the corresponding table (6), use file format option error_on_column_count_mismatch=false to ignore this error |                2 |                     1 | "EMP_BASIC"["LAST_NAME":2]     |
+-----------------------------+-------------+-------------+-------------+-------------+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+--------------------------------+
2 Row(s) produced. Time Elapsed: 1.012s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>REMOVE @sf_tuts.public.%emp_basic;
+-----------------------------+---------+
| name                        | result  |
|-----------------------------+---------|
| sample_customer_data.csv.gz | removed |
| somedata.csv.gz             | removed |
+-----------------------------+---------+
2 Row(s) produced. Time Elapsed: 0.134s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>LIST @sf_tuts.public.%emp_basic;
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>PUT file:///home/jomoon/Snowflake/data/*.csv @sf_tuts.public.%emp_basic;
+--------------------------+-----------------------------+-------------+-------------+--------------------+--------------------+----------+---------+
| source                   | target                      | source_size | target_size | source_compression | target_compression | status   | message |
|--------------------------+-----------------------------+-------------+-------------+--------------------+--------------------+----------+---------|
| sample_customer_data.csv | sample_customer_data.csv.gz |         174 |         128 | NONE               | GZIP               | UPLOADED |         |
| somedata.csv             | somedata.csv.gz             |         110 |         128 | NONE               | GZIP               | UPLOADED |         |
+--------------------------+-----------------------------+-------------+-------------+--------------------+--------------------+----------+---------+
2 Row(s) produced. Time Elapsed: 15.457s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>LIST @sf_tuts.public.%emp_basic;
+-----------------------------+------+----------------------------------+------------------------------+
| name                        | size | md5                              | last_modified                |
|-----------------------------+------+----------------------------------+------------------------------|
| sample_customer_data.csv.gz |  128 | e5a835ae73286a3b7879e9c28598813a | Thu, 1 May 2025 17:30:14 GMT |
| somedata.csv.gz             |  128 | b1bb9807127aaae90df3230814c2df47 | Thu, 1 May 2025 17:30:03 GMT |
+-----------------------------+------+----------------------------------+------------------------------+
2 Row(s) produced. Time Elapsed: 0.108s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>COPY INTO emp_basic
                                                                    FROM @%emp_basic
                                                                    FILE_FORMAT = (type = csv field_optionally_enclosed_by='"')
                                                                    PATTERN = '.*.csv.gz'
                                                                    ON_ERROR = 'skip_file';


+-----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                        | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|-----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| somedata.csv.gz             | LOADED |           4 |           4 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
| sample_customer_data.csv.gz | LOADED |           5 |           5 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+-----------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
2 Row(s) produced. Time Elapsed: 0.905s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>SELECT * FROM emp_basic;
+------------+-----------+-------+---------------+------+------------+
| FIRST_NAME | LAST_NAME | EMAIL | STREETADDRESS | CITY | START_DATE |
|------------+-----------+-------+---------------+------+------------|
| Prague     | Jan       | 101   | 4875.33       | 1    | 1970-01-01 |
| Rome       | Mar       | 87    | 1557.39       | 1    | 1970-01-01 |
| Bangalore  | May       | 317   | 8936.99       | 1    | 1970-01-01 |
| Beijing    | Jul       | 411   | 11600.67      | 1    | 1970-01-01 |
| 1313131    | 1000.00   | 1     | 2             | 3    | 1970-01-01 |
| 4444444    | 99.13     | 1     | 2             | 3    | 1970-01-01 |
| 1515151    | 500.05    | 1     | 2             | 3    | 1970-01-01 |
| 6666666    | 1.12      | 1     | 2             | 3    | 1970-01-01 |
| 1717171    | 3000.03   | 1     | 2             | 3    | 1970-01-01 |
+------------+-----------+-------+---------------+------+------------+
9 Row(s) produced. Time Elapsed: 5.696s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>INSERT INTO emp_basic VALUES
                                    ('Clementine','Adamou','cadamou@sf_tuts.com','10510 Sachs Road','Klenak','2017-9-22') ,
                                    ('Marlowe','De Anesy','madamouc@sf_tuts.co.uk','36768 Northfield Plaza','Fangshan','2017-1-26');
+-------------------------+
| number of rows inserted |
|-------------------------|
|                       2 |
+-------------------------+
2 Row(s) produced. Time Elapsed: 0.616s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>SELECT * FROM emp_basic;
+------------+-----------+------------------------+------------------------+----------+------------+
| FIRST_NAME | LAST_NAME | EMAIL                  | STREETADDRESS          | CITY     | START_DATE |
|------------+-----------+------------------------+------------------------+----------+------------|
| Prague     | Jan       | 101                    | 4875.33                | 1        | 1970-01-01 |
| Rome       | Mar       | 87                     | 1557.39                | 1        | 1970-01-01 |
| Bangalore  | May       | 317                    | 8936.99                | 1        | 1970-01-01 |
| Beijing    | Jul       | 411                    | 11600.67               | 1        | 1970-01-01 |
| 1313131    | 1000.00   | 1                      | 2                      | 3        | 1970-01-01 |
| 4444444    | 99.13     | 1                      | 2                      | 3        | 1970-01-01 |
| 1515151    | 500.05    | 1                      | 2                      | 3        | 1970-01-01 |
| 6666666    | 1.12      | 1                      | 2                      | 3        | 1970-01-01 |
| 1717171    | 3000.03   | 1                      | 2                      | 3        | 1970-01-01 |
| Clementine | Adamou    | cadamou@sf_tuts.com    | 10510 Sachs Road       | Klenak   | 2017-09-22 |
| Marlowe    | De Anesy  | madamouc@sf_tuts.co.uk | 36768 Northfield Plaza | Fangshan | 2017-01-26 |
+------------+-----------+------------------------+------------------------+----------+------------+
11 Row(s) produced. Time Elapsed: 0.106s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>SELECT email FROM emp_basic WHERE email LIKE '%.uk';
+------------------------+
| EMAIL                  |
|------------------------|
| madamouc@sf_tuts.co.uk |
+------------------------+
1 Row(s) produced. Time Elapsed: 0.142s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>SELECT first_name, last_name, DATEADD('day',90,start_date) FROM emp_basic WHERE start_date <= '2017-01-01';
+------------+-----------+------------------------------+
| FIRST_NAME | LAST_NAME | DATEADD('DAY',90,START_DATE) |
|------------+-----------+------------------------------|
| Prague     | Jan       | 1970-04-01                   |
| Rome       | Mar       | 1970-04-01                   |
| Bangalore  | May       | 1970-04-01                   |
| Beijing    | Jul       | 1970-04-01                   |
| 1313131    | 1000.00   | 1970-04-01                   |
| 4444444    | 99.13     | 1970-04-01                   |
| 1515151    | 500.05    | 1970-04-01                   |
| 6666666    | 1.12      | 1970-04-01                   |
| 1717171    | 3000.03   | 1970-04-01                   |
+------------+-----------+------------------------------+
9 Row(s) produced. Time Elapsed: 0.068s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>DROP DATABASE IF EXISTS sf_tuts;
+-------------------------------+
| status                        |
|-------------------------------|
| SF_TUTS successfully dropped. |
+-------------------------------+
1 Row(s) produced. Time Elapsed: 0.111s
```


### X
```sql
jomoon#SF_TUTS_WH@SF_TUTS.PUBLIC>DROP WAREHOUSE IF EXISTS sf_tuts_wh;
+----------------------------------+
| status                           |
|----------------------------------|
| SF_TUTS_WH successfully dropped. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.110s
```

