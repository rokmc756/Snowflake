
Execute the following statements to create a database, two tables (for csv and json data), and a virtual warehouse needed for this tutorial. After you complete the tutorial, you can drop these objects.

### Create a database. A database automatically includes a schema named 'public'.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> CREATE OR REPLACE DATABASE mydatabase;

+-------------------------------------------+
| status                                    |
|-------------------------------------------|
| Database MYDATABASE successfully created. |
+-------------------------------------------+
1 Row(s) produced. Time Elapsed: 5.342s


### Create target tables for CSV and JSON data. The tables are temporary, meaning they persist only for the duration of the user session and are not visible to other users.
jomoon#COMPUTE_WH@MYDATABASE.PUBLIC> CREATE OR REPLACE TEMPORARY TABLE mycsvtable (
                                      id INTEGER,
                                      last_name STRING,
                                      first_name STRING,
                                      company STRING,
                                      email STRING,
                                      workphone STRING,
                                      cellphone STRING,
                                      streetaddress STRING,
                                      city STRING,
                                      postalcode STRING);
+----------------------------------------+
| status                                 |
|----------------------------------------|
| Table MYCSVTABLE successfully created. |
+----------------------------------------+
1 Row(s) produced. Time Elapsed: 0.133s


### X
jomoon#COMPUTE_WH@MYDATABASE.PUBLIC> CREATE OR REPLACE TEMPORARY TABLE myjsontable (
                                      json_data VARIANT);
+-----------------------------------------+
| status                                  |
|-----------------------------------------|
| Table MYJSONTABLE successfully created. |
+-----------------------------------------+
1 Row(s) produced. Time Elapsed: 0.125s


### Create a warehouse
jomoon#COMPUTE_WH@MYDATABASE.PUBLIC> CREATE OR REPLACE WAREHOUSE mywarehouse WITH
                                      WAREHOUSE_SIZE='X-SMALL'
                                      AUTO_SUSPEND = 120
                                      AUTO_RESUME = TRUE
                                      INITIALLY_SUSPENDED=TRUE;
+---------------------------------------------+
| status                                      |
|---------------------------------------------|
| Warehouse MYWAREHOUSE successfully created. |
+---------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.090s


### Creating a file format object for CSV data ( Execute the CREATE FILE FORMAT command to create the mycsvformat file format )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE FILE FORMAT mycsvformat
                                       TYPE = 'CSV'
                                       FIELD_DELIMITER = '|'
                                       SKIP_HEADER = 1;
+-----------------------------------------------+
| status                                        |
|-----------------------------------------------|
| File format MYCSVFORMAT successfully created. |
+-----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.055s


### Creating a file format object for JSON data ( Execute the CREATE FILE FORMAT command to create the myjsonformat file format )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE FILE FORMAT myjsonformat
                                       TYPE = 'JSON'
                                       STRIP_OUTER_ARRAY = TRUE;
+------------------------------------------------+
| status                                         |
|------------------------------------------------|
| File format MYJSONFORMAT successfully created. |
+------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.063s


### Creating a stage for CSV data files ( Execute CREATE STAGE to create the my_csv_stage stage )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE STAGE my_csv_stage
                                       FILE_FORMAT = mycsvformat;
+-----------------------------------------------+
| status                                        |
|-----------------------------------------------|
| Stage area MY_CSV_STAGE successfully created. |
+-----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.090s


### Creating a stage for JSON data files ( Execute CREATE STAGE to create the my_json_stage stage )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE STAGE my_json_stage
                                       FILE_FORMAT = myjsonformat;
+------------------------------------------------+
| status                                         |
|------------------------------------------------|
| Stage area MY_JSON_STAGE successfully created. |
+------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.106s


### Staging the CSV sample data files ( Execute the PUT command to upload the CSV files from your local file system )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> PUT file:///tmp/load/contacts*.csv @my_csv_stage AUTO_COMPRESS=TRUE;
+---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------+
| source        | target           | source_size | target_size | source_compression | target_compression | status   | message |
|---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------|
| contacts1.csv | contacts1.csv.gz |         694 |         512 | NONE               | GZIP               | UPLOADED |         |
| contacts2.csv | contacts2.csv.gz |         763 |         576 | NONE               | GZIP               | UPLOADED |         |
| contacts3.csv | contacts3.csv.gz |         771 |         576 | NONE               | GZIP               | UPLOADED |         |
| contacts4.csv | contacts4.csv.gz |         750 |         576 | NONE               | GZIP               | UPLOADED |         |
| contacts5.csv | contacts5.csv.gz |         887 |         624 | NONE               | GZIP               | UPLOADED |         |
+---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------+
5 Row(s) produced. Time Elapsed: 15.552s


### Staging the JSON sample data files ( Execute the PUT command to upload the JSON file from your local file system to the named stage )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> PUT file:///tmp/load/contacts.json @my_json_stage AUTO_COMPRESS=TRUE;
+---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------+
| source        | target           | source_size | target_size | source_compression | target_compression | status   | message |
|---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------|
| contacts.json | contacts.json.gz |         965 |         448 | NONE               | GZIP               | UPLOADED |         |
+---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------+
1 Row(s) produced. Time Elapsed: 5.403s


### List the staged files (optional) - List the staged files by using the LIST command
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> LIST @my_csv_stage;
+-------------------------------+------+----------------------------------+------------------------------+
| name                          | size | md5                              | last_modified                |
|-------------------------------+------+----------------------------------+------------------------------|
| my_csv_stage/contacts1.csv.gz |  512 | 5f2b6f9c94b4082d7ef101edbda2bb83 | Fri, 2 May 2025 04:55:35 GMT |
| my_csv_stage/contacts2.csv.gz |  576 | a131d539be31147a2ded51e9ef49ef03 | Fri, 2 May 2025 04:55:25 GMT |
| my_csv_stage/contacts3.csv.gz |  576 | 71476c6ff1a58bc9618f61a39e38d0d4 | Fri, 2 May 2025 04:55:35 GMT |
| my_csv_stage/contacts4.csv.gz |  576 | 260cc19a67a2b997ad73f12e8925a4f0 | Fri, 2 May 2025 04:55:35 GMT |
| my_csv_stage/contacts5.csv.gz |  624 | 08d82b08ad05bd532070b48eec96a1e9 | Fri, 2 May 2025 04:55:35 GMT |
+-------------------------------+------+----------------------------------+------------------------------+
5 Row(s) produced. Time Elapsed: 0.079s


### List the staged files (optional) - List the staged files by using the LIST command
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> LIST @my_json_stage;
+--------------------------------+------+----------------------------------+------------------------------+
| name                           | size | md5                              | last_modified                |
|--------------------------------+------+----------------------------------+------------------------------|
| my_json_stage/contacts.json.gz |  448 | b84805a648a588b498078f8e1c7efb7f | Fri, 2 May 2025 04:55:57 GMT |
+--------------------------------+------+----------------------------------+------------------------------+
1 Row(s) produced. Time Elapsed: 0.073s


### Start by loading the data from one of the files (contacts1.csv.gz). Execute the following:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO mycsvtable
                                       FROM @my_csv_stage/contacts1.csv.gz
                                       FILE_FORMAT = (FORMAT_NAME = mycsvformat)
                                       ON_ERROR = 'skip_file';
+-------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                          | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|-------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| my_csv_stage/contacts1.csv.gz | LOADED |           5 |           5 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+-------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 1.029s


### Load the rest of the staged files in the mycsvtable table. The following example uses pattern matching to load data from all files that match the regular expression .*contacts[1-5].csv.gz into the mycsvtable table.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO mycsvtable
                                       FROM @my_csv_stage
                                       FILE_FORMAT = (FORMAT_NAME = mycsvformat)
                                       PATTERN='.*contacts[1-5].csv.gz'
                                       ON_ERROR = 'skip_file';



+-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------+
| file                          | status      | rows_parsed | rows_loaded | error_limit | errors_seen | first_error                                                                                                                                                          | first_error_line | first_error_character | first_error_column_name |
|-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------|
| my_csv_stage/contacts5.csv.gz | LOADED      |           6 |           6 |           1 |           0 | NULL                                                                                                                                                                 |             NULL |                  NULL | NULL                    |
| my_csv_stage/contacts2.csv.gz | LOADED      |           5 |           5 |           1 |           0 | NULL                                                                                                                                                                 |             NULL |                  NULL | NULL                    |
| my_csv_stage/contacts4.csv.gz | LOADED      |           5 |           5 |           1 |           0 | NULL                                                                                                                                                                 |             NULL |                  NULL | NULL                    |
| my_csv_stage/contacts3.csv.gz | LOAD_FAILED |           5 |           0 |           1 |           2 | Number of columns in file (11) does not match that of the corresponding table (10), use file format option error_on_column_count_mismatch=false to ignore this error |                3 |                     1 | "MYCSVTABLE"[11]        |
+-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------+
4 Row(s) produced. Time Elapsed: 0.842s


### Load the contacts.json.gz staged data file into the myjsontable table.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO myjsontable
                                       FROM @my_json_stage/contacts.json.gz
                                       FILE_FORMAT = (FORMAT_NAME = myjsonformat)
                                       ON_ERROR = 'skip_file';


+--------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                           | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|--------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| my_json_stage/contacts.json.gz | LOADED |           3 |           3 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+--------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 0.850s


### Validate the COPY INTO command execution, represented by the query ID, and save errors to a new table named save_copy_errors. In SnowSQL, execute the following command. Replace query_id with the Query ID value.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE TABLE save_copy_errors AS SELECT * FROM TABLE(VALIDATE(mycsvtable, JOB_ID=>'<query_id>'));
002018 (22023): SQL compilation error:
Invalid argument [Invalid Job UUID provided.] for table function. Table function argument is required to be a constant.


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO mycsvtable
                                       FROM @my_csv_stage/contacts1.csv.gz
                                       FILE_FORMAT = (FORMAT_NAME = mycsvformat)
                                       ON_ERROR = 'skip_file';

+---------------------------------------+
| status                                |
|---------------------------------------|
| Copy executed with 0 files processed. |
+---------------------------------------+
1 Row(s) produced. Time Elapsed: 6.055s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO mycsvtable
                                       FROM @my_csv_stage
                                       FILE_FORMAT = (FORMAT_NAME = mycsvformat)
                                       PATTERN='.*contacts[1-5].csv.gz'
                                       ON_ERROR = 'skip_file';
+-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------+
| file                          | status      | rows_parsed | rows_loaded | error_limit | errors_seen | first_error                                                                                                                                                          | first_error_line | first_error_character | first_error_column_name |
|-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------|
| my_csv_stage/contacts3.csv.gz | LOAD_FAILED |           5 |           0 |           1 |           2 | Number of columns in file (11) does not match that of the corresponding table (10), use file format option error_on_column_count_mismatch=false to ignore this error |                3 |                     1 | "MYCSVTABLE"[11]        |
+-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 0.713s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO myjsontable
                                       FROM @my_json_stage/contacts.json.gz
                                       FILE_FORMAT = (FORMAT_NAME = myjsonformat)
                                       ON_ERROR = 'skip_file';
+---------------------------------------+
| status                                |
|---------------------------------------|
| Copy executed with 0 files processed. |
+---------------------------------------+
1 Row(s) produced. Time Elapsed: 0.369s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE TABLE save_copy_errors AS SELECT * FROM TABLE(VALIDATE(mycsvtable, JOB_ID=>'<query_id>'));
002018 (22023): SQL compilation error:
Invalid argument [Invalid Job UUID provided.] for table function. Table function argument is required to be a constant.


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * FROM SAVE_COPY_ERRORS;
002003 (42S02): SQL compilation error:
Object 'SAVE_COPY_ERRORS' does not exist or not authorized.


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> PUT file:///tmp/load/contacts3.csv @my_csv_stage AUTO_COMPRESS=TRUE OVERWRITE=TRUE;
+---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------+
| source        | target           | source_size | target_size | source_compression | target_compression | status   | message |
|---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------|
| contacts3.csv | contacts3.csv.gz |         771 |         576 | NONE               | GZIP               | UPLOADED |         |
+---------------+------------------+-------------+-------------+--------------------+--------------------+----------+---------+
1 Row(s) produced. Time Elapsed: 5.310s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO mycsvtable
                                       FROM @my_csv_stage/contacts3.csv.gz
                                       FILE_FORMAT = (FORMAT_NAME = mycsvformat)
                                       ON_ERROR = 'skip_file';

+-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------+
| file                          | status      | rows_parsed | rows_loaded | error_limit | errors_seen | first_error                                                                                                                                                          | first_error_line | first_error_character | first_error_column_name |
|-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------|
| my_csv_stage/contacts3.csv.gz | LOAD_FAILED |           5 |           0 |           1 |           2 | Number of columns in file (11) does not match that of the corresponding table (10), use file format option error_on_column_count_mismatch=false to ignore this error |                3 |                     1 | "MYCSVTABLE"[11]        |
+-------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 0.873s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * FROM mycsvtable;
+----+-----------+------------+----------------------------------+----------------------------------------+----------------+----------------+--------------------------------+------------------+------------+
| ID | LAST_NAME | FIRST_NAME | COMPANY                          | EMAIL                                  | WORKPHONE      | CELLPHONE      | STREETADDRESS                  | CITY             | POSTALCODE |
|----+-----------+------------+----------------------------------+----------------------------------------+----------------+----------------+--------------------------------+------------------+------------|
|  1 | Imani     | Davidson   | At Ltd                           | nec@sem.net                            | 1-243-889-8106 | 1-730-771-0412 | 369-6531 Molestie St.          | Russell          | 74398      |
|  2 | Kelsie    | Abbott     | Neque Sed Institute              | lacus@pede.net                         | 1-467-506-9933 | 1-441-508-7753 | P.O. Box 548, 1930 Pede. Road  | Campbellton      | 27022      |
|  3 | Hilel     | Durham     | Pede Incorporated                | eu@Craspellentesque.net                | 1-752-108-4210 | 1-391-449-8733 | Ap #180-2360 Nisl. Street      | Etalle           | 84025      |
|  4 | Graiden   | Molina     | Sapien Institute                 | sit@fermentum.net                      | 1-130-156-6666 | 1-269-605-7776 | 8890 A, Rd.                    | Dundee           | 70504      |
|  5 | Karyn     | Howard     | Pede Ac Industries               | sed.hendrerit@ornaretortorat.edu       | 1-109-166-5492 | 1-506-782-5089 | P.O. Box 902, 5398 Et, St.     | Saint-Hilarion   | 26232      |
| 20 | Michelle  | Dickson    | Ut Limited                       | Duis.dignissim.tempor@cursuset.org     | 1-202-490-0151 | 1-129-553-7398 | 6752 Eros. St.                 | Stornaway        | 61290      |
| 21 | Lance     | Harper     | Rutrum Lorem Limited             | Sed.neque@risus.com                    | 1-685-778-6726 | 1-494-188-6168 | 663-7682 Et St.                | Gisborne         | 73449      |
| 22 | Keely     | Pace       | Eleifend Limited                 | ante.bibendum.ullamcorper@necenim.edu  | 1-312-381-5244 | 1-432-225-9226 | P.O. Box 506, 5233 Aliquam Av. | Woodlands County | 61213      |
| 23 | Sage      | Leblanc    | Egestas A Consulting             | dapibus@elementum.org                  | 1-630-981-0327 | 1-301-287-0495 | 4463 Lorem Road                | Woodlands County | 33951      |
| 24 | Marny     | Holt       | Urna Nec Luctus Associates       | ornare@vitaeorci.ca                    | 1-522-364-3947 | 1-460-971-8360 | P.O. Box 311, 4839 Nulla Av.   | Port Coquitlam   | 36733      |
| 25 | Holly     | Park       | Mauris PC                        | Vestibulum.ante@Maecenasliberoest.org  | 1-370-197-9316 | 1-411-413-4602 | P.O. Box 732, 8967 Eu Avenue   | Provost          | 45507      |
|  6 | Reed      | Moses      | Neque Corporation                | eget.lacus@facilisis.com               | 1-449-871-0780 | 1-454-964-5318 | Ap #225-4351 Dolor Ave         | Titagarh         | 62631      |
|  7 | Audrey    | Franks     | Arcu Eu Limited                  | eu.dui@aceleifendvitae.org             | 1-527-945-8935 | 1-263-127-1173 | Ap #786-9241 Mauris Road       | Bergen           | 81958      |
|  8 | Jakeem    | Erickson   | A Ltd                            | Pellentesque.habitant@liberoProinmi.ca | 1-381-591-9386 | 1-379-391-9490 | 319-1703 Dis Rd.               | Pangnirtung      | 62399      |
|  9 | Xaviera   | Brennan    | Bibendum Ullamcorper Limited     | facilisi.Sed.neque@dictum.edu          | 1-260-757-1919 | 1-211-651-0925 | P.O. Box 146, 8385 Vel Road    | Béziers          | 13082      |
| 10 | Francis   | Ortega     | Vitae Velit Egestas Associates   | egestas.rhoncus.Proin@faucibus.com     | 1-257-584-6487 | 1-211-870-2111 | 733-7191 Neque Rd.             | Chatillon        | 33081      |
| 16 | Aretha    | Sykes      | Lobortis Tellus Justo Foundation | eget@Naminterdumenim.net               | 1-670-849-1866 | 1-283-783-3710 | Ap #979-2481 Dui. Av.          | Thurso           | 66851      |
| 17 | Akeem     | Casey      | Pharetra Quisque Ac Institute    | dictum.eu@magna.edu                    | 1-277-657-0361 | 1-623-630-8848 | Ap #363-6074 Ullamcorper, Rd.  | Idar-Oberstei    | 30848      |
| 18 | Keelie    | Mendez     | Purus In Foundation              | Nulla.eu.neque@Aeneanegetmetus.co.uk   | 1-330-370-8231 | 1-301-568-0413 | 3511 Tincidunt Street          | Lanklaar         | 73942      |
| 19 | Lane      | Bishop     | Libero At PC                     | non@dapibusligula.ca                   | 1-340-862-4623 | 1-513-820-9039 | 7459 Pede. Street              | Linkebeek        | 89252      |
| 20 | Michelle  | Dickson    | Ut Limited                       | Duis.dignissim.tempor@cursuset.org     | 1-202-490-0151 | 1-129-553-7398 | 6752 Eros. St.                 | Stornaway        | 61290      |
+----+-----------+------------+----------------------------------+----------------------------------------+----------------+----------------+--------------------------------+------------------+------------+
21 Row(s) produced. Time Elapsed: 0.139s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * FROM myjsontable;
+-----------------------------------------------------------------+
| JSON_DATA                                                       |
|-----------------------------------------------------------------|
| {                                                               |
|   "customer": {                                                 |
|     "_id": "5730864df388f1d653e37e6f",                          |
|     "address": "509 Kings Hwy, Comptche, Missouri, 4848",       |
|     "company": "ORBIN",                                         |
|     "email": "blankenship.patrick@orbin.ca",                    |
|     "name": {                                                   |
|       "first": "Blankenship",                                   |
|       "last": "Patrick"                                         |
|     },                                                          |
|     "phone": "+1 (999) 407-2274"                                |
|   }                                                             |
| }                                                               |
| {                                                               |
|   "customer": {                                                 |
|     "_id": "5730864d4d8523c8baa8baf6",                          |
|     "address": "290 Lefferts Avenue, Malott, Delaware, 1575",   |
|     "company": "SNIPS",                                         |
|     "email": "anna.glass@snips.name",                           |
|     "name": {                                                   |
|       "first": "Anna",                                          |
|       "last": "Glass"                                           |
|     },                                                          |
|     "phone": "+1 (958) 411-2876"                                |
|   }                                                             |
| }                                                               |
| {                                                               |
|   "customer": {                                                 |
|     "_id": "5730864e375e08523150fc04",                          |
|     "address": "756 Randolph Street, Omar, Rhode Island, 3310", |
|     "company": "ESCHOIR",                                       |
|     "email": "sparks.ramos@eschoir.co.uk",                      |
|     "name": {                                                   |
|       "first": "Sparks",                                        |
|       "last": "Ramos"                                           |
|     },                                                          |
|     "phone": "+1 (962) 436-2519"                                |
|   }                                                             |
| }                                                               |
+-----------------------------------------------------------------+
3 Row(s) produced. Time Elapsed: 0.149s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> REMOVE @my_csv_stage PATTERN='.*.csv.gz';
+-------------------------------+---------+
| name                          | result  |
|-------------------------------+---------|
| my_csv_stage/contacts1.csv.gz | removed |
| my_csv_stage/contacts4.csv.gz | removed |
| my_csv_stage/contacts3.csv.gz | removed |
| my_csv_stage/contacts2.csv.gz | removed |
| my_csv_stage/contacts5.csv.gz | removed |
+-------------------------------+---------+
5 Row(s) produced. Time Elapsed: 0.172s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> REMOVE @my_json_stage PATTERN='.*.json.gz';
+--------------------------------+---------+
| name                           | result  |
|--------------------------------+---------|
| my_json_stage/contacts.json.gz | removed |
+--------------------------------+---------+
1 Row(s) produced. Time Elapsed: 0.112s


jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP DATABASE IF EXISTS mydatabase;
+----------------------------------+
| status                           |
|----------------------------------|
| MYDATABASE successfully dropped. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.090s


### X
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP WAREHOUSE IF EXISTS mywarehouse;
+-----------------------------------+
| status                            |
|-----------------------------------|
| MYWAREHOUSE successfully dropped. |
+-----------------------------------+
1 Row(s) produced. Time Elapsed: 0.137s
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC>
