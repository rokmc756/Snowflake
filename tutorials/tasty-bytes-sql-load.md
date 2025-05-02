
### Place your cursor in the USE ROLE line.
jomoon#COMPUTE_WH@(no database).(no schema)> USE ROLE accountadmin;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 5.221s


### Place your cursor in the USE WAREHOUSE line, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> USE WAREHOUSE compute_wh;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.060s


### Place your cursor in the CREATE OR REPLACE DATABASE line, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> CREATE OR REPLACE DATABASE tasty_bytes_sample_data;
+--------------------------------------------------------+
| status                                                 |
|--------------------------------------------------------|
| Database TASTY_BYTES_SAMPLE_DATA successfully created. |
+--------------------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.118s


### Place your cursor in the CREATE OR REPLACE SCHEMA line, then select Run.
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.PUBLIC> CREATE OR REPLACE SCHEMA tasty_bytes_sample_data.raw_pos;
+--------------------------------------+
| status                               |
|--------------------------------------|
| Schema RAW_POS successfully created. |
+--------------------------------------+
1 Row(s) produced. Time Elapsed: 0.063s


### Place your cursor in the CREATE OR REPLACE TABLE lines, then select Run.
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> CREATE OR REPLACE TABLE tasty_bytes_sample_data.raw_pos.menu
                                                  (
                                                      menu_id NUMBER(19,0),
                                                      menu_type_id NUMBER(38,0),
                                                      menu_type VARCHAR(16777216),
                                                      truck_brand_name VARCHAR(16777216),
                                                      menu_item_id NUMBER(38,0),
                                                      menu_item_name VARCHAR(16777216),
                                                      item_category VARCHAR(16777216),
                                                      item_subcategory VARCHAR(16777216),
                                                      cost_of_goods_usd NUMBER(38,4),
                                                      sale_price_usd NUMBER(38,4),
                                                      menu_item_health_metrics_obj VARIANT
                                                  );
+----------------------------------+
| status                           |
|----------------------------------|
| Table MENU successfully created. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.145s


### To confirm that the table was created successfully, place your cursor in the SELECT line, then select Run.
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> SELECT * FROM tasty_bytes_sample_data.raw_pos.menu;
+---------+--------------+-----------+------------------+--------------+----------------+---------------+------------------+-------------------+----------------+------------------------------+
| MENU_ID | MENU_TYPE_ID | MENU_TYPE | TRUCK_BRAND_NAME | MENU_ITEM_ID | MENU_ITEM_NAME | ITEM_CATEGORY | ITEM_SUBCATEGORY | COST_OF_GOODS_USD | SALE_PRICE_USD | MENU_ITEM_HEALTH_METRICS_OBJ |
|---------+--------------+-----------+------------------+--------------+----------------+---------------+------------------+-------------------+----------------+------------------------------|
+---------+--------------+-----------+------------------+--------------+----------------+---------------+------------------+-------------------+----------------+------------------------------+
0 Row(s) produced. Time Elapsed: 0.063s


### Place your cursor in the CREATE OR REPLACE STAGE lines, then select Run.
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> CREATE OR REPLACE STAGE tasty_bytes_sample_data.public.blob_stage
                                                  url = 's3://sfquickstarts/tastybytes/'
                                                  file_format = (type = csv);
+---------------------------------------------+
| status                                      |
|---------------------------------------------|
| Stage area BLOB_STAGE successfully created. |
+---------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.674s


### To confirm that the stage was created successfully, place your cursor in the LIST line, then select Run.
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> LIST @tasty_bytes_sample_data.public.blob_stage/raw_pos/menu/;
+--------------------------------------------------------+------+----------------------------------+------------------------------+
| name                                                   | size | md5                              | last_modified                |
|--------------------------------------------------------+------+----------------------------------+------------------------------|
| s3://sfquickstarts/tastybytes/raw_pos/menu/menu.csv.gz | 3478 | 9dd9a858e141c2e1c11c0ba95117321c | Mon, 3 Apr 2023 15:10:40 GMT |
+--------------------------------------------------------+------+----------------------------------+------------------------------+
1 Row(s) produced. Time Elapsed: 1.041s


### To load the data into the table, place your cursor in the COPY INTO lines, then select Run.
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> COPY INTO tasty_bytes_sample_data.raw_pos.menu
                                                  FROM @tasty_bytes_sample_data.public.blob_stage/raw_pos/menu/;
+--------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
| file                                                   | status | rows_parsed | rows_loaded | error_limit | errors_seen | first_error | first_error_line | first_error_character | first_error_column_name |
|--------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------|
| s3://sfquickstarts/tastybytes/raw_pos/menu/menu.csv.gz | LOADED |         100 |         100 |           1 |           0 | NULL        |             NULL |                  NULL | NULL                    |
+--------------------------------------------------------+--------+-------------+-------------+-------------+-------------+-------------+------------------+-----------------------+-------------------------+
1 Row(s) produced. Time Elapsed: 2.257s


### For example, to return the number of rows in the table, run the following query:
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS>SELECT COUNT(*) AS row_count FROM tasty_bytes_sample_data.raw_pos.menu;
+-----------+
| ROW_COUNT |
|-----------|
|       100 |
+-----------+
1 Row(s) produced. Time Elapsed: 0.126s


### Run this query to return the top ten rows in the table:
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> SELECT TOP 10 * FROM tasty_bytes_sample_data.raw_pos.menu;
+---------+--------------+-----------+------------------+--------------+--------------------+---------------+------------------+-------------------+----------------+-----------------------------------+
| MENU_ID | MENU_TYPE_ID | MENU_TYPE | TRUCK_BRAND_NAME | MENU_ITEM_ID | MENU_ITEM_NAME     | ITEM_CATEGORY | ITEM_SUBCATEGORY | COST_OF_GOODS_USD | SALE_PRICE_USD | MENU_ITEM_HEALTH_METRICS_OBJ      |
|---------+--------------+-----------+------------------+--------------+--------------------+---------------+------------------+-------------------+----------------+-----------------------------------|
|   10001 |            1 | Ice Cream | Freezing Point   |           10 | Lemonade           | Beverage      | Cold Option      |            0.6500 |         3.5000 | {                                 |
|         |              |           |                  |              |                    |               |                  |                   |                |   "menu_item_health_metrics": [   |
~~ snip


### If the objects you created in this tutorial are no longer needed, you can remove them from the system with DROP <object> commands. To remove the database you created, run the following command:
jomoon#COMPUTE_WH@TASTY_BYTES_SAMPLE_DATA.RAW_POS> DROP DATABASE IF EXISTS tasty_bytes_sample_data;
+-----------------------------------------------+
| status                                        |
|-----------------------------------------------|
| TASTY_BYTES_SAMPLE_DATA successfully dropped. |
+-----------------------------------------------+
1 Row(s) produced. Time Elapsed: 0.103s


