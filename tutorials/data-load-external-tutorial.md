

# Creating the database, tables, and warehouse
# Execute the following statements to create a database, two tables (for csv and json data),
# and a virtual warehouse needed for this tutorial. After you complete the tutorial, you can drop these objects. )
jomoon#COMPUTE_WH@(no database).(no schema)> CREATE OR REPLACE DATABASE mydatabase;
+-------------------------------------------+
| status                                    |
|-------------------------------------------|
| Database MYDATABASE successfully created. |
+-------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.263s


### X
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
1 Row(s) produced. Time Elapsed: 0.154s




### X
jomoon#COMPUTE_WH@MYDATABASE.PUBLIC> CREATE OR REPLACE TEMPORARY TABLE myjsontable (
                                         json_data VARIANT);

+-----------------------------------------+
| status                                  |
|-----------------------------------------|
| Table MYJSONTABLE successfully created. |
+-----------------------------------------+
1 Row(s) produced. Time Elapsed: 0.125s


### X
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
1 Row(s) produced. Time Elapsed: 0.092s



# [ Note the following ]
# The CREATE DATABASE statement creates a database. The database automatically includes a schema named ‘public’.
# The CREATE TABLE statements create target tables for CSV and JSON data.
# The tables are temporary, that is, they persist only for the duration of the user session and are not visible to other users.
# The CREATE WAREHOUSE statement creates an initially suspended warehouse.
# The statement also sets AUTO_RESUME = true, which starts the warehouse automatically when you execute SQL statements that require compute resources.



### Creating a file format object for CSV data ( Execute the CREATE FILE FORMAT command to create the mycsvformat file format. )
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


### Creating a file format object for JSON data ( Execute the CREATE FILE FORMAT command to create the myjsonformat file format. )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE FILE FORMAT myjsonformat
                                       TYPE = 'JSON'
                                       STRIP_OUTER_ARRAY = TRUE;
+------------------------------------------------+
| status                                         |
|------------------------------------------------|
| File format MYJSONFORMAT successfully created. |
+------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.056s


### Creating a stage for CSV data files ( Execute CREATE STAGE to create the my_csv_stage stage: )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE STAGE my_csv_stage
                                       FILE_FORMAT = mycsvformat
                                       URL = 's3://snowflake-docs';


+-----------------------------------------------+
| status                                        |
|-----------------------------------------------|
| Stage area MY_CSV_STAGE successfully created. |
+-----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.841s


### Creating a stage for JSON data files ( Execute CREATE STAGE to create the my_json_stage stage: )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE STAGE my_json_stage
                                       FILE_FORMAT = myjsonformat
                                       URL = 's3://snowflake-docs';


+------------------------------------------------+
| status                                         |
|------------------------------------------------|
| Stage area MY_JSON_STAGE successfully created. |
+------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.884s


### In regular use, if you were creating a stage that pointed to your private data files, you would reference a storage integration created using CREATE STORAGE INTEGRATION by an account administrator (i.e. a user with the ACCOUNTADMIN role) or a role with the global CREATE INTEGRATION privilege:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE STAGE external_stage
                                       FILE_FORMAT = mycsvformat
                                       URL = 's3://private-bucket'
                                       STORAGE_INTEGRATION = myint;
002003 (02000): SQL compilation error:
Integration 'MYINT' does not exist or not authorized.


### To load the data from the sample CSV files:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO mycsvtable
                                       FROM @my_csv_stage/tutorials/dataloading/contacts1.csv
                                       ON_ERROR = 'skip_file';


+---------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                                                    | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|---------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| s3://snowflake-docs/tutorials/dataloading/contacts1.csv | LOADED |           5 |           5 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+---------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 2.878s



### Load the rest of the staged files in the mycsvtable table.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO mycsvtable
                                       FROM @my_csv_stage/tutorials/dataloading/
                                       PATTERN='.*contacts[1-5].csv'
                                       ON_ERROR = 'skip_file';



+---------------------------------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------+
| file                                                    | status      | rows_parsed | rows_loaded | error_limit | errors_seen | first_error                                                                                                                                                          | first_error_line | first_error_character | first_error_column_name |
|---------------------------------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------|
| s3://snowflake-docs/tutorials/dataloading/contacts5.csv | LOADED      |           6 |           6 |           1 |           0 | NULL                                                                                                                                                                 |             NULL |                  NULL | NULL                    |
| s3://snowflake-docs/tutorials/dataloading/contacts3.csv | LOAD_FAILED |           5 |           0 |           1 |           2 | Number of columns in file (11) does not match that of the corresponding table (10), use file format option error_on_column_count_mismatch=false to ignore this error |                3 |                     1 | "MYCSVTABLE"[11]        |
| s3://snowflake-docs/tutorials/dataloading/contacts2.csv | LOADED      |           5 |           5 |           1 |           0 | NULL                                                                                                                                                                 |             NULL |                  NULL | NULL                    |
| s3://snowflake-docs/tutorials/dataloading/contacts4.csv | LOADED      |           5 |           5 |           1 |           0 | NULL                                                                                                                                                                 |             NULL |                  NULL | NULL                    |
+---------------------------------------------------------+-------------+-------------+-------------+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+-----------------------+-------------------------+
4 Row(s) produced. Time Elapsed: 2.724s


### Load the contacts.json staged data file into the myjsontable table.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> COPY INTO myjsontable
                                       FROM @my_json_stage/tutorials/dataloading/contacts.json
                                       ON_ERROR = 'skip_file';
+---------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                                                    | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|---------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| s3://snowflake-docs/tutorials/dataloading/contacts.json | LOADED |           3 |           3 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+---------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 2.650s


### In SnowSQL, execute the following command. Replace query_id with the Query ID value.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> CREATE OR REPLACE TABLE save_copy_errors AS SELECT * FROM TABLE(VALIDATE(mycsvtable, JOB_ID=>'<query_id>'));
002018 (22023): SQL compilation error:
Invalid argument [Invalid Job UUID provided.] for table function. Table function argument is required to be a constant.


### Query the save_copy_errors table.
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * FROM SAVE_COPY_ERRORS;
002003 (42S02): SQL compilation error:
Object 'SAVE_COPY_ERRORS' does not exist or not authorized.


### Verify the loaded data ( Execute a SELECT statement to verify that the data was loaded successfully. )
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> SELECT * FROM mycsvtable;
+----+-----------+------------+----------------------------------+----------------------------------------+----------------+----------------+--------------------------------+------------------+------------+
| ID | LAST_NAME | FIRST_NAME | COMPANY                          | EMAIL                                  | WORKPHONE      | CELLPHONE      | STREETADDRESS                  | CITY             | POSTALCODE |
|----+-----------+------------+----------------------------------+----------------------------------------+----------------+----------------+--------------------------------+------------------+------------|
| 20 | Michelle  | Dickson    | Ut Limited                       | Duis.dignissim.tempor@cursuset.org     | 1-202-490-0151 | 1-129-553-7398 | 6752 Eros. St.                 | Stornaway        | 61290      |
| 21 | Lance     | Harper     | Rutrum Lorem Limited             | Sed.neque@risus.com                    | 1-685-778-6726 | 1-494-188-6168 | 663-7682 Et St.                | Gisborne         | 73449      |
| 22 | Keely     | Pace       | Eleifend Limited                 | ante.bibendum.ullamcorper@necenim.edu  | 1-312-381-5244 | 1-432-225-9226 | P.O. Box 506, 5233 Aliquam Av. | Woodlands County | 61213      |
| 23 | Sage      | Leblanc    | Egestas A Consulting             | dapibus@elementum.org                  | 1-630-981-0327 | 1-301-287-0495 | 4463 Lorem Road                | Woodlands County | 33951      |
| 24 | Marny     | Holt       | Urna Nec Luctus Associates       | ornare@vitaeorci.ca                    | 1-522-364-3947 | 1-460-971-8360 | P.O. Box 311, 4839 Nulla Av.   | Port Coquitlam   | 36733      |
| 25 | Holly     | Park       | Mauris PC                        | Vestibulum.ante@Maecenasliberoest.org  | 1-370-197-9316 | 1-411-413-4602 | P.O. Box 732, 8967 Eu Avenue   | Provost          | 45507      |
|  1 | Imani     | Davidson   | At Ltd                           | nec@sem.net                            | 1-243-889-8106 | 1-730-771-0412 | 369-6531 Molestie St.          | Russell          | 74398      |
|  2 | Kelsie    | Abbott     | Neque Sed Institute              | lacus@pede.net                         | 1-467-506-9933 | 1-441-508-7753 | P.O. Box 548, 1930 Pede. Road  | Campbellton      | 27022      |
|  3 | Hilel     | Durham     | Pede Incorporated                | eu@Craspellentesque.net                | 1-752-108-4210 | 1-391-449-8733 | Ap #180-2360 Nisl. Street      | Etalle           | 84025      |
|  4 | Graiden   | Molina     | Sapien Institute                 | sit@fermentum.net                      | 1-130-156-6666 | 1-269-605-7776 | 8890 A, Rd.                    | Dundee           | 70504      |
|  5 | Karyn     | Howard     | Pede Ac Industries               | sed.hendrerit@ornaretortorat.edu       | 1-109-166-5492 | 1-506-782-5089 | P.O. Box 902, 5398 Et, St.     | Saint-Hilarion   | 26232      |
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
21 Row(s) produced. Time Elapsed: 0.169s


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
3 Row(s) produced. Time Elapsed: 0.337s



### Tutorial clean up (optional) - Execute the following DROP <object> commands to return your system to its state before you began the tutorial:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP DATABASE IF EXISTS mydatabase;
+----------------------------------+
| status                           |
|----------------------------------|
| MYDATABASE successfully dropped. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.069s


### Tutorial clean up (optional) - Execute the following DROP <object> commands to return your system to its state before you began the tutorial:
jomoon#MYWAREHOUSE@MYDATABASE.PUBLIC> DROP WAREHOUSE IF EXISTS mywarehouse;
+-----------------------------------+
| status                            |
|-----------------------------------|
| MYWAREHOUSE successfully dropped. |
+-----------------------------------+
1 Row(s) produced. Time Elapsed: 0.116s


