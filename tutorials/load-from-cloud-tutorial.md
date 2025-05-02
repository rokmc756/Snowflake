
### Place your cursor in the USE ROLE line.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> USE ROLE accountadmin;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.046s


### Place your cursor in the USE WAREHOUSE line, then select Run.
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> USE WAREHOUSE compute_wh;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.036s


### Place your cursor in the CREATE OR REPLACE DATABASE line, enter a name for your database and an optional comment, then select Run. For example:
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> CREATE OR REPLACE DATABASE cloud_data_db
                                                    COMMENT = 'Database for loading cloud data';
+----------------------------------------------+
| status                                       |
|----------------------------------------------|
| Database CLOUD_DATA_DB successfully created. |
+----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.121s


### Place your cursor in the CREATE OR REPLACE SCHEMA line, enter a name for your schema and an optional comment, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.PUBLIC> CREATE OR REPLACE SCHEMA cloud_data_db.s3_data
                                         COMMENT = 'Schema for tables loaded from S3';
+--------------------------------------+
| status                               |
|--------------------------------------|
| Schema S3_DATA successfully created. |
+--------------------------------------+
1 Row(s) produced. Time Elapsed: 0.061s


### Place your cursor in the CREATE OR REPLACE TABLE lines, complete the table definition, add an optional comment, and select Run. For example, the following table contains six columns:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> CREATE OR REPLACE TABLE cloud_data_db.s3_data.calendar
                                          (
                                          full_date DATE
                                          ,day_name VARCHAR(10)
                                          ,month_name VARCHAR(10)
                                          ,day_number VARCHAR(2)
                                          ,full_year VARCHAR(4)
                                          ,holiday BOOLEAN
                                          )
                                          COMMENT = 'Table to be loaded from S3 calendar data file';
+--------------------------------------+
| status                               |
|--------------------------------------|
| Table CALENDAR successfully created. |
+--------------------------------------+
1 Row(s) produced. Time Elapsed: 0.145s


### To confirm that the table was created successfully, place your cursor in the SELECT line, then select Run.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> SELECT * FROM cloud_data_db.s3_data.calendar;
+-----------+----------+------------+------------+-----------+---------+
| FULL_DATE | DAY_NAME | MONTH_NAME | DAY_NUMBER | FULL_YEAR | HOLIDAY |
|-----------+----------+------------+------------+-----------+---------|
+-----------+----------+------------+------------+-----------+---------+
0 Row(s) produced. Time Elapsed: 0.073s


### Place your cursor in the CREATE OR REPLACE STORAGE INTEGRATION lines, define the required parameters, and select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> CREATE OR REPLACE STORAGE INTEGRATION s3_data_integration
                                          TYPE = EXTERNAL_STAGE
                                          STORAGE_PROVIDER = 'S3'
                                          STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::631373164455:role/tutorial_role'
                                          ENABLED = TRUE
                                          STORAGE_ALLOWED_LOCATIONS = ('s3://snow-tutorial-bucket/s3data/');
+-------------------------------------------------------+
| status                                                |
|-------------------------------------------------------|
| Integration S3_DATA_INTEGRATION successfully created. |
+-------------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.166s


### Place your cursor in the DESCRIBE INTEGRATION line, specify the name of the storage integration you created, and select Run.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> DESCRIBE INTEGRATION s3_data_integration;
+---------------------------+---------------+------------------------------------------------+------------------+
| property                  | property_type | property_value                                 | property_default |
|---------------------------+---------------+------------------------------------------------+------------------|
| ENABLED                   | Boolean       | true                                           | false            |
| STORAGE_PROVIDER          | String        | S3                                             |                  |
| STORAGE_ALLOWED_LOCATIONS | List          | s3://snow-tutorial-bucket/s3data/              | []               |
| STORAGE_BLOCKED_LOCATIONS | List          |                                                | []               |
| STORAGE_AWS_IAM_USER_ARN  | String        | arn:aws:iam::246272778318:user/x4rv0000-s      |                  |
| STORAGE_AWS_ROLE_ARN      | String        | arn:aws:iam::631373164455:role/tutorial_role   |                  |
| STORAGE_AWS_EXTERNAL_ID   | String        | UL74497_SFCRole=5_o6uSy2KIwYQuTiPmL+qzijabX90= |                  |
| USE_PRIVATELINK_ENDPOINT  | Boolean       | false                                          | false            |
| COMMENT                   | String        |                                                |                  |
+---------------------------+---------------+------------------------------------------------+------------------+
9 Row(s) produced. Time Elapsed: 0.628s


### Place your cursor in the SHOW INTEGRATIONS line and select Run. This command returns information about the storage integration you created.
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> SHOW INTEGRATIONS;
+---------------------+----------------+----------+---------+---------+-------------------------------+
| name                | type           | category | enabled | comment | created_on                    |
|---------------------+----------------+----------+---------+---------+-------------------------------|
| S3_DATA_INTEGRATION | EXTERNAL_STAGE | STORAGE  | true    | NULL    | 2025-05-01 21:19:35.159 -0700 |
+---------------------+----------------+----------+---------+---------+-------------------------------+
1 Row(s) produced. Time Elapsed: 0.089s


### Place your cursor in the CREATE OR REPLACE STAGE lines, specify a name, the storage integration you created, the bucket URL, and the correct file format, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> CREATE OR REPLACE STAGE cloud_data_db.s3_data.s3data_stage
                                          STORAGE_INTEGRATION = s3_data_integration
                                          URL = 's3://snow-tutorial-bucket/s3data/'
                                          FILE_FORMAT = (TYPE = CSV);
+-----------------------------------------------+
| status                                        |
|-----------------------------------------------|
| Stage area S3DATA_STAGE successfully created. |
+-----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.930s


### Return information about the stage you created:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> SHOW STAGES;
+-------------------------------+--------------+---------------+-------------+-----------------------------------+-----------------+--------------------+--------------+---------+-----------+----------+-------+----------------------+---------------------+----------+-----------------+-------------------+
| created_on                    | name         | database_name | schema_name | url                               | has_credentials | has_encryption_key | owner        | comment | region    | type     | cloud | notification_channel | storage_integration | endpoint | owner_role_type | directory_enabled |
|-------------------------------+--------------+---------------+-------------+-----------------------------------+-----------------+--------------------+--------------+---------+-----------+----------+-------+----------------------+---------------------+----------+-----------------+-------------------|
| 2025-05-01 21:19:59.787 -0700 | S3DATA_STAGE | CLOUD_DATA_DB | S3_DATA     | s3://snow-tutorial-bucket/s3data/ | N               | N                  | ACCOUNTADMIN |         | us-east-2 | EXTERNAL | AWS   | NULL                 | S3_DATA_INTEGRATION | NULL     | ROLE            | N                 |
+-------------------------------+--------------+---------------+-------------+-----------------------------------+-----------------+--------------------+--------------+---------+-----------+----------+-------+----------------------+---------------------+----------+-----------------+-------------------+
1 Row(s) produced. Time Elapsed: 0.073s


### To load the data into the table, place your cursor in the COPY INTO lines, specify the table name, the stage you created, and name of the file (or files) you want to load, then select Run. For example:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> COPY INTO cloud_data_db.s3_data.calendar
                                          FROM @cloud_data_db.s3_data.s3data_stage
                                            FILES = ('calendar.txt');
003167 (42601): Error assuming AWS_ROLE:
User: arn:aws:iam::246272778318:user/x4rv0000-s is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::631373164455:role/tutorial_role


### To run a query in the open worksheet, select the line or lines of the SELECT command, and then select Run. For example, run the following query:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> SELECT * FROM cloud_data_db.s3_data.calendar;
+-----------+----------+------------+------------+-----------+---------+
| FULL_DATE | DAY_NAME | MONTH_NAME | DAY_NUMBER | FULL_YEAR | HOLIDAY |
|-----------+----------+------------+------------+-----------+---------|
+-----------+----------+------------+------------+-----------+---------+
0 Row(s) produced. Time Elapsed: 0.115s


### Take a few minutes to review a short summary and the key points covered in the tutorial. You might also want to consider cleaning up by dropping any objects you created in the tutorial. For example, you might want to drop the table you created and loaded:
jomoon#COMPUTE_WH@CLOUD_DATA_DB.S3_DATA> DROP TABLE calendar;
+--------------------------------+
| status                         |
|--------------------------------|
| CALENDAR successfully dropped. |
+--------------------------------+
1 Row(s) produced. Time Elapsed: 0.099s


