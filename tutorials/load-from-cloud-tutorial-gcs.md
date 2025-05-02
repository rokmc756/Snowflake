
### Place your cursor in the USE ROLE line.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> USE ROLE accountadmin;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 5.262s


### Place your cursor in the USE WAREHOUSE line, then select Run.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> USE WAREHOUSE compute_wh;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.065s



### Place your cursor in the CREATE OR REPLACE DATABASE line, enter a name for your database and an optional comment, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> CREATE OR REPLACE DATABASE cloud_data_db
                                             COMMENT = 'Database for loading cloud data';
+----------------------------------------------+
| status                                       |
|----------------------------------------------|
| Database CLOUD_DATA_DB successfully created. |
+----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.134s


### Place your cursor in the CREATE OR REPLACE SCHEMA line, enter a name for your schema and an optional comment, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.PUBLIC> CREATE OR REPLACE SCHEMA cloud_data_db.gcs_data
                                         COMMENT = 'Schema for tables loaded from GCS';
+---------------------------------------+
| status                                |
|---------------------------------------|
| Schema GCS_DATA successfully created. |
+---------------------------------------+
1 Row(s) produced. Time Elapsed: 0.058s


### Place your cursor in the CREATE OR REPLACE TABLE lines, complete the table definition, add an optional comment, and select Run. For example, the following table contains six columns:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> CREATE OR REPLACE TABLE cloud_data_db.gcs_data.calendar
                                           (
                                           full_date DATE,
                                           day_name VARCHAR(10),
                                           month_name VARCHAR(10),
                                           day_number VARCHAR(2),
                                           full_year VARCHAR(4),
                                           holiday BOOLEAN
                                           )
                                           COMMENT = 'Table to be loaded from GCS calendar data file';
+--------------------------------------+
| status                               |
|--------------------------------------|
| Table CALENDAR successfully created. |
+--------------------------------------+
1 Row(s) produced. Time Elapsed: 0.157s


### To confirm that the table was created successfully, place your cursor in the SELECT line, then select Run.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> SELECT * FROM cloud_data_db.gcs_data.calendar;
+-----------+----------+------------+------------+-----------+---------+
| FULL_DATE | DAY_NAME | MONTH_NAME | DAY_NUMBER | FULL_YEAR | HOLIDAY |
|-----------+----------+------------+------------+-----------+---------|
+-----------+----------+------------+------------+-----------+---------+
0 Row(s) produced. Time Elapsed: 0.072s


### Place your cursor in the CREATE OR REPLACE STORAGE INTEGRATION lines, define the required parameters, and select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> CREATE OR REPLACE STORAGE INTEGRATION gcs_data_integration
                                           TYPE = EXTERNAL_STAGE
                                           STORAGE_PROVIDER = 'GCS'
                                           ENABLED = TRUE
                                           STORAGE_ALLOWED_LOCATIONS = ('gcs://tutorial24bucket/gcsdata/');
+--------------------------------------------------------+
| status                                                 |
|--------------------------------------------------------|
| Integration GCS_DATA_INTEGRATION successfully created. |
+--------------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.094s


### Place your cursor in the DESCRIBE INTEGRATION line, specify the name of the storage integration you created, and select Run. This command returns information about the storage integration you created, including the Service Account ID (STORAGE_GCP_SERVICE_ACCOUNT) that was created automatically for your Snowflake account. You will use this value to configure permissions for Snowflake in the GCS Console.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> DESCRIBE INTEGRATION gcs_data_integration;
+-----------------------------+---------------+-----------------------------------------------------------+------------------+
| property                    | property_type | property_value                                            | property_default |
|-----------------------------+---------------+-----------------------------------------------------------+------------------|
| ENABLED                     | Boolean       | true                                                      | false            |
| STORAGE_PROVIDER            | String        | GCS                                                       |                  |
| STORAGE_ALLOWED_LOCATIONS   | List          | gcs://tutorial24bucket/gcsdata/                           | []               |
| STORAGE_BLOCKED_LOCATIONS   | List          |                                                           | []               |
| STORAGE_GCP_SERVICE_ACCOUNT | String        | udilgpwdlx@awsapnortheast2-1-9975.iam.gserviceaccount.com |                  |
| COMMENT                     | String        |                                                           |                  |
+-----------------------------+---------------+-----------------------------------------------------------+------------------+
6 Row(s) produced. Time Elapsed: 0.046s


### Place your cursor in the SHOW INTEGRATIONS line and select Run.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> SHOW INTEGRATIONS;
+------------------------+----------------+----------+---------+---------+-------------------------------+
| name                   | type           | category | enabled | comment | created_on                    |
|------------------------+----------------+----------+---------+---------+-------------------------------|
| AZURE_DATA_INTEGRATION | EXTERNAL_STAGE | STORAGE  | true    | NULL    | 2025-05-01 21:34:01.862 -0700 |
| GCS_DATA_INTEGRATION   | EXTERNAL_STAGE | STORAGE  | true    | NULL    | 2025-05-01 21:44:07.471 -0700 |
| S3_DATA_INTEGRATION    | EXTERNAL_STAGE | STORAGE  | true    | NULL    | 2025-05-01 21:24:19.211 -0700 |
+------------------------+----------------+----------+---------+---------+-------------------------------+
3 Row(s) produced. Time Elapsed: 0.047s


### Place your cursor in the CREATE OR REPLACE STAGE lines, specify a name, the storage integration you created, the bucket URL, and the correct file format, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> CREATE OR REPLACE STAGE cloud_data_db.gcs_data.gcsdata_stage
                                           STORAGE_INTEGRATION = gcs_data_integration
                                           URL = 'gcs://tutorial24bucket/gcsdata/'
                                           FILE_FORMAT = (TYPE = CSV);



+------------------------------------------------+
| status                                         |
|------------------------------------------------|
| Stage area GCSDATA_STAGE successfully created. |
+------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.786s


### Return information about the stage you created:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> SHOW STAGES;
+-------------------------------+---------------+---------------+-------------+---------------------------------+-----------------+--------------------+--------------+---------+--------+----------+-------+----------------------+----------------------+----------+-----------------+-------------------+
| created_on                    | name          | database_name | schema_name | url                             | has_credentials | has_encryption_key | owner        | comment | region | type     | cloud | notification_channel | storage_integration  | endpoint | owner_role_type | directory_enabled |
|-------------------------------+---------------+---------------+-------------+---------------------------------+-----------------+--------------------+--------------+---------+--------+----------+-------+----------------------+----------------------+----------+-----------------+-------------------|
| 2025-05-01 21:44:28.772 -0700 | GCSDATA_STAGE | CLOUD_DATA_DB | GCS_DATA    | gcs://tutorial24bucket/gcsdata/ | N               | N                  | ACCOUNTADMIN |         |        | EXTERNAL | GCP   | NULL                 | GCS_DATA_INTEGRATION | NULL     | ROLE            | N                 |
+-------------------------------+---------------+---------------+-------------+---------------------------------+-----------------+--------------------+--------------+---------+--------+----------+-------+----------------------+----------------------+----------+-----------------+-------------------+
1 Row(s) produced. Time Elapsed: 0.048s


### To load the data into the table, place your cursor in the COPY INTO lines, specify the table name, the stage you created, and name of the file (or files) you want to load, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> COPY INTO cloud_data_db.gcs_data.calendar
                                           FROM @cloud_data_db.gcs_data.gcsdata_stage
                                             FILES = ('calendar.txt');



091003 (22000): Failure using stage area. Cause: [The billing account for the owning project is disabled in state closed (Status Code: 403)]


### To run a query in the open worksheet, select the line or lines of the SELECT command, and then select Run. For example, run the following query:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.GCS_DATA> SELECT * FROM cloud_data_db.gcs_data.calendar;
+-----------+----------+------------+------------+-----------+---------+
| FULL_DATE | DAY_NAME | MONTH_NAME | DAY_NUMBER | FULL_YEAR | HOLIDAY |
|-----------+----------+------------+------------+-----------+---------|
+-----------+----------+------------+------------+-----------+---------+
0 Row(s) produced. Time Elapsed: 0.049s


