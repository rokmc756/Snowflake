
### Place your cursor in the USE ROLE line.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> USE ROLE accountadmin;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 5.260s


### Place your cursor in the USE WAREHOUSE line, then select Run.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> USE WAREHOUSE compute_wh;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.038s


### Place your cursor in the CREATE OR REPLACE DATABASE line, enter a name for your database and an optional comment, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> CREATE OR REPLACE DATABASE cloud_data_db
                                          COMMENT = 'Database for loading cloud data';
+----------------------------------------------+
| status                                       |
|----------------------------------------------|
| Database CLOUD_DATA_DB successfully created. |
+----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.176s


### Place your cursor in the CREATE OR REPLACE SCHEMA line, enter a name for your schema and an optional comment, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.PUBLIC> CREATE OR REPLACE SCHEMA cloud_data_db.azure_data
                                         COMMENT = 'Schema for tables loaded from Azure';
+-----------------------------------------+
| status                                  |
|-----------------------------------------|
| Schema AZURE_DATA successfully created. |
+-----------------------------------------+
1 Row(s) produced. Time Elapsed: 0.058s



### Place your cursor in the CREATE OR REPLACE TABLE lines, complete the table definition, add an optional comment, and select Run. For example, the following table contains six columns:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> CREATE OR REPLACE TABLE cloud_data_db.azure_data.calendar
                                             (
                                             full_date DATE
                                             ,day_name VARCHAR(10)
                                             ,month_name VARCHAR(10)
                                             ,day_number VARCHAR(2)
                                             ,full_year VARCHAR(4)
                                             ,holiday BOOLEAN
                                             )
                                             COMMENT = 'Table to be loaded from Azure calendar data file';
+--------------------------------------+
| status                               |
|--------------------------------------|
| Table CALENDAR successfully created. |
+--------------------------------------+
1 Row(s) produced. Time Elapsed: 0.152s


### To confirm that the table was created successfully, place your cursor in the SELECT line, then select Run.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> SELECT * FROM cloud_data_db.azure_data.calendar;
+-----------+----------+------------+------------+-----------+---------+
| FULL_DATE | DAY_NAME | MONTH_NAME | DAY_NUMBER | FULL_YEAR | HOLIDAY |
|-----------+----------+------------+------------+-----------+---------|
+-----------+----------+------------+------------+-----------+---------+
0 Row(s) produced. Time Elapsed: 0.055s


### Place your cursor in the CREATE OR REPLACE STORAGE INTEGRATION lines, define the required parameters, and select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> CREATE OR REPLACE STORAGE INTEGRATION azure_data_integration
                                             TYPE = EXTERNAL_STAGE
                                             STORAGE_PROVIDER = 'AZURE'
                                             AZURE_TENANT_ID = '075f576e-6f9b-4955-8e99-4086736225d9'
                                             ENABLED = TRUE
                                             STORAGE_ALLOWED_LOCATIONS = ('azure://tutorial99.blob.core.windows.net/snow-tutorial-container/');
+----------------------------------------------------------+
| status                                                   |
|----------------------------------------------------------|
| Integration AZURE_DATA_INTEGRATION successfully created. |
+----------------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.122s


### Place your cursor in the DESCRIBE INTEGRATION line, specify the name of the storage integration you created, and select Run.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> DESCRIBE INTEGRATION azure_data_integration;
+-----------------------------+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+
| property                    | property_type | property_value                                                                                                                                            | property_default |
|-----------------------------+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+------------------|
| ENABLED                     | Boolean       | true                                                                                                                                                      | false            |
| STORAGE_PROVIDER            | String        | AZURE                                                                                                                                                     |                  |
| STORAGE_ALLOWED_LOCATIONS   | List          | azure://tutorial99.blob.core.windows.net/snow-tutorial-container/                                                                                         | []               |
| STORAGE_BLOCKED_LOCATIONS   | List          |                                                                                                                                                           | []               |
| USE_PRIVATELINK_ENDPOINT    | Boolean       | false                                                                                                                                                     | false            |
| AZURE_TENANT_ID             | String        | 075f576e-6f9b-4955-8e99-4086736225d9                                                                                                                      |                  |
| AZURE_CONSENT_URL           | String        | https://login.microsoftonline.com/075f576e-6f9b-4955-8e99-4086736225d9/oauth2/authorize?client_id=ab92526b-2ed7-475d-ae6c-dfed50ddd8fc&response_type=code |                  |
| AZURE_MULTI_TENANT_APP_NAME | String        | SnowflakePACInt3428_1746160441892                                                                                                                         |                  |
| COMMENT                     | String        |                                                                                                                                                           |                  |
+-----------------------------+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+
9 Row(s) produced. Time Elapsed: 0.040s


### Place your cursor in the SHOW INTEGRATIONS line and select Run. This command returns information about the storage integration you created.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> SHOW INTEGRATIONS;
+------------------------+----------------+----------+---------+---------+-------------------------------+
| name                   | type           | category | enabled | comment | created_on                    |
|------------------------+----------------+----------+---------+---------+-------------------------------|
| AZURE_DATA_INTEGRATION | EXTERNAL_STAGE | STORAGE  | true    | NULL    | 2025-05-01 21:34:01.862 -0700 |
| S3_DATA_INTEGRATION    | EXTERNAL_STAGE | STORAGE  | true    | NULL    | 2025-05-01 21:24:19.211 -0700 |
+------------------------+----------------+----------+---------+---------+-------------------------------+
2 Row(s) produced. Time Elapsed: 0.055s


### Place your cursor in the CREATE OR REPLACE STAGE lines, specify a name, the storage integration you created, the bucket URL, and the correct file format, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> CREATE OR REPLACE STAGE cloud_data_db.azure_data.azuredata_stage
                                             STORAGE_INTEGRATION = azure_data_integration
                                             URL = 'azure://tutorial99.blob.core.windows.net/snow-tutorial-container/'
                                             FILE_FORMAT = (TYPE = CSV);


+--------------------------------------------------+
| status                                           |
|--------------------------------------------------|
| Stage area AZUREDATA_STAGE successfully created. |
+--------------------------------------------------+
1 Row(s) produced. Time Elapsed: 10.103s


### Return information about the stage you created:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> SHOW STAGES;
+-------------------------------+-----------------+---------------+-------------+-------------------------------------------------------------------+-----------------+--------------------+--------------+---------+---------+----------+-------+----------------------+------------------------+----------+-----------------+-------------------+
| created_on                    | name            | database_name | schema_name | url                                                               | has_credentials | has_encryption_key | owner        | comment | region  | type     | cloud | notification_channel | storage_integration    | endpoint | owner_role_type | directory_enabled |
|-------------------------------+-----------------+---------------+-------------+-------------------------------------------------------------------+-----------------+--------------------+--------------+---------+---------+----------+-------+----------------------+------------------------+----------+-----------------+-------------------|
| 2025-05-01 21:34:21.244 -0700 | AZUREDATA_STAGE | CLOUD_DATA_DB | AZURE_DATA  | azure://tutorial99.blob.core.windows.net/snow-tutorial-container/ | N               | N                  | ACCOUNTADMIN |         | unknown | EXTERNAL | AZURE | NULL                 | AZURE_DATA_INTEGRATION | NULL     | ROLE            | N                 |
+-------------------------------+-----------------+---------------+-------------+-------------------------------------------------------------------+-----------------+--------------------+--------------+---------+---------+----------+-------+----------------------+------------------------+----------+-----------------+-------------------+
1 Row(s) produced. Time Elapsed: 0.049s


### To load the data into the table, place your cursor in the COPY INTO lines, specify the table name, the stage you created, and name of the file (or files) you want to load, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> COPY INTO cloud_data_db.azure_data.calendar
                                             FROM @cloud_data_db.azure_data.azuredata_stage
                                               FILES = ('calendar.txt');



091003 (22000): Failure using stage area. Cause: [Unable to retrieve Access Token (Status Code: AADSTS90002), please check your role assignment and retry]


### To run a query in the open worksheet, select the line or lines of the SELECT command, and then select Run. For example, run the following query:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> SELECT * FROM cloud_data_db.azure_data.calendar;
+-----------+----------+------------+------------+-----------+---------+
| FULL_DATE | DAY_NAME | MONTH_NAME | DAY_NUMBER | FULL_YEAR | HOLIDAY |
|-----------+----------+------------+------------+-----------+---------|
+-----------+----------+------------+------------+-----------+---------+
0 Row(s) produced. Time Elapsed: 0.125s


### Take a few minutes to review a short summary and the key points covered in the tutorial. You might also want to consider cleaning up by dropping any objects you created in the tutorial. For example, you might want to drop the table you created and loaded:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.AZURE_DATA> DROP TABLE calendar;
+--------------------------------+
| status                         |
|--------------------------------|
| CALENDAR successfully dropped. |
+--------------------------------+
1 Row(s) produced. Time Elapsed: 0.077s


