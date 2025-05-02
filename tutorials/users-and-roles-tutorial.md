
### Login to SnowSQL
```sh
jomoon@LAPTOP-OS28E8H5:~/Snowflake$ snowsql -a gnsjdpk-tj82189 -u jomoon
* SnowSQL * v1.3.3
Type SQL statements or !help
```


### Place your cursor in the USE ROLE line
```sql
jomoon#COMPUTE_WH@(no database).(no schema)> USE ROLE USERADMIN;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.084s
```


### Create a user with a name, a password, and some other properties.
``` sql
jomoon#COMPUTE_WH@(no database).(no schema)> CREATE OR REPLACE USER snowman
                                            PASSWORD = 'sn0wf@ll'
                                            LOGIN_NAME = 'snowstorm'
                                            FIRST_NAME = 'Snow'
                                            LAST_NAME = 'Storm'
                                            EMAIL = 'snow.storm@snowflake.com'
                                            MUST_CHANGE_PASSWORD = true
                                            DEFAULT_WAREHOUSE = COMPUTE_WH;
+------------------------------------+
| status                             |
|------------------------------------|
| User SNOWMAN successfully created. |
+------------------------------------+
1 Row(s) produced. Time Elapsed: 0.123s
```


### Place your cursor in the USE ROLE line, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> USE ROLE SECURITYADMIN;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.062s


### Place your cursor in the GRANT ROLE line, enter the name of the user you created, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> GRANT ROLE SYSADMIN TO USER snowman;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.061s


### Place your cursor in the GRANT USAGE line, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE SYSADMIN;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.059s


### In the open worksheet, place your cursor in the USE ROLE line, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> USE ROLE ACCOUNTADMIN;
+----------------------------------+
| status                           |
|----------------------------------|
| Statement executed successfully. |
+----------------------------------+
1 Row(s) produced. Time Elapsed: 0.044s


### Place your cursor in the SHOW USERS line, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> SHOW USERS;
+-----------+-------------------------------+------------+--------------+------------+-----------+--------------------------+----------------+---------------------+---------+----------+----------------------+----------------+-------------------+-------------------+--------------+-------------------------+---------------+---------------+--------------------+--------------+-------------------------------+-------------------------------+-------------------+--------------+--------------------+------+---------+
| name      | created_on                    | login_name | display_name | first_name | last_name | email                    | mins_to_unlock | days_to_expiry      | comment | disabled | must_change_password | snowflake_lock | default_warehouse | default_namespace | default_role | default_secondary_roles | ext_authn_duo | ext_authn_uid | mins_to_bypass_mfa | owner        | last_success_login            | expires_at_time               | locked_until_time | has_password | has_rsa_public_key | type | has_mfa |
|-----------+-------------------------------+------------+--------------+------------+-----------+--------------------------+----------------+---------------------+---------+----------+----------------------+----------------+-------------------+-------------------+--------------+-------------------------+---------------+---------------+--------------------+--------------+-------------------------------+-------------------------------+-------------------+--------------+--------------------+------+---------|
| JOMOON    | 2025-04-30 21:18:07.665 -0700 | JOMOON     | JOMOON       | Jack       | Moon      | rokmc756@gmail.com       | NULL           | NULL                | NULL    | false    | false                | false          | COMPUTE_WH        | NULL              | ACCOUNTADMIN | []                      | false         | NULL          | NULL               | ACCOUNTADMIN | 2025-05-01 20:01:21.001 -0700 | NULL                          | NULL              | true         | false              | NULL | false   |
| SNOWFLAKE | 2025-04-30 21:18:07.682 -0700 | SNOWFLAKE  | SNOWFLAKE    | NULL       | NULL      | NULL                     | NULL           | 0.02696759259259259 | NULL    | false    | false                | false          | NULL              | NULL              | NULL         | ["ALL"]                 | false         | NULL          | NULL               |              | 2025-04-30 21:18:11.192 -0700 | 2025-05-01 21:18:10.191 -0700 | NULL              | true         | false              | NULL | false   |
| SNOWMAN   | 2025-05-01 20:38:40.201 -0700 | SNOWSTORM  | SNOWMAN      | Snow       | Storm     | snow.storm@snowflake.com | NULL           | NULL                | NULL    | false    | true                 | false          | COMPUTE_WH        | NULL              | NULL         | ["ALL"]                 | false         | NULL          | NULL               | USERADMIN    | NULL                          | NULL                          | NULL              | true         | false              | NULL | false   |
+-----------+-------------------------------+------------+--------------+------------+-----------+--------------------------+----------------+---------------------+---------+----------+----------------------+----------------+-------------------+-------------------+--------------+-------------------------+---------------+---------------+--------------------+--------------+-------------------------------+-------------------------------+-------------------+--------------+--------------------+------+---------+
3 Row(s) produced. Time Elapsed: 0.060s


### Place your cursor in the SHOW ROLES line, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> SHOW ROLES;
+-------------------------------+---------------+------------+------------+--------------+-------------------+------------------+---------------+-------+-----------------------------------------------------------------------------------+
| created_on                    | name          | is_default | is_current | is_inherited | assigned_to_users | granted_to_roles | granted_roles | owner | comment                                                                           |
|-------------------------------+---------------+------------+------------+--------------+-------------------+------------------+---------------+-------+-----------------------------------------------------------------------------------|
| 2025-04-30 21:18:07.740 -0700 | ACCOUNTADMIN  | Y          | Y          | N            |                 1 |                0 |             2 |       | Account administrator can manage all aspects of the account.                      |
| 2025-04-30 21:18:08.032 -0700 | ORGADMIN      | N          | N          | N            |                 1 |                0 |             0 |       | Organization administrator can manage organizations and accounts in organizations |
| 2025-04-30 21:18:07.699 -0700 | PUBLIC        | N          | N          | Y            |                 0 |                0 |             0 |       | Public role is automatically available to every user in the account.              |
| 2025-04-30 21:18:07.725 -0700 | SECURITYADMIN | N          | N          | Y            |                 0 |                1 |             1 |       | Security administrator can manage security aspects of the account.                |
| 2025-04-30 21:18:07.719 -0700 | SYSADMIN      | N          | N          | Y            |                 1 |                1 |             0 |       | System administrator can create and manage databases and warehouses.              |
| 2025-04-30 21:18:07.734 -0700 | USERADMIN     | N          | N          | Y            |                 0 |                1 |             0 |       | User administrator can create and manage users and roles                          |
+-------------------------------+---------------+------------+------------+--------------+-------------------+------------------+---------------+-------+-----------------------------------------------------------------------------------+
6 Row(s) produced. Time Elapsed: 0.049s


### Place your cursor in the DROP USER line, enter the name of the user you created, then select Run.
jomoon#COMPUTE_WH@(no database).(no schema)> DROP USER snowman;
+-------------------------------+
| status                        |
|-------------------------------|
| SNOWMAN successfully dropped. |
+-------------------------------+
1 Row(s) produced. Time Elapsed: 0.090s



