# Snowflake - DBT project

## Snowflake Configuration

1. Create and login to Snoflake trial account
2. Create a database and a service accounts for dbt.

```sql
-------------------------------------------
-- dbt credentials
-------------------------------------------
USE ROLE securityadmin;
-- dbt roles
CREATE OR REPLACE ROLE dbt_dev_role;
CREATE OR REPLACE ROLE dbt_prod_role;
------------------------------------------- Please replace with your dbt user password
CREATE OR REPLACE USER dbt_user PASSWORD = "<mysecretpassword>";

GRANT ROLE dbt_dev_role,dbt_prod_role TO USER dbt_user;
GRANT ROLE dbt_dev_role,dbt_prod_role TO ROLE sysadmin;

-------------------------------------------
-- dbt objects
-------------------------------------------
USE ROLE sysadmin;

CREATE OR REPLACE WAREHOUSE dbt_dev_wh  WITH WAREHOUSE_SIZE = 'XSMALL' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 1 INITIALLY_SUSPENDED = TRUE;
CREATE OR REPLACE WAREHOUSE dbt_dev_heavy_wh  WITH WAREHOUSE_SIZE = 'LARGE' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 1 INITIALLY_SUSPENDED = TRUE;
CREATE OR REPLACE WAREHOUSE dbt_prod_wh WITH WAREHOUSE_SIZE = 'XSMALL' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 1 INITIALLY_SUSPENDED = TRUE;
CREATE OR REPLACE WAREHOUSE dbt_prod_heavy_wh  WITH WAREHOUSE_SIZE = 'LARGE' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 1 INITIALLY_SUSPENDED = TRUE;

GRANT ALL ON WAREHOUSE dbt_dev_wh  TO ROLE dbt_dev_role;
GRANT ALL ON WAREHOUSE dbt_dev_heavy_wh  TO ROLE dbt_dev_role;
GRANT ALL ON WAREHOUSE dbt_prod_wh TO ROLE dbt_prod_role;
GRANT ALL ON WAREHOUSE dbt_prod_heavy_wh  TO ROLE dbt_prod_role;

CREATE OR REPLACE DATABASE dbt_hol_dev; 
CREATE OR REPLACE DATABASE dbt_hol_prod; 
GRANT ALL ON DATABASE dbt_hol_dev  TO ROLE dbt_dev_role;
GRANT ALL ON DATABASE dbt_hol_prod TO ROLE dbt_prod_role;
GRANT ALL ON ALL SCHEMAS IN DATABASE dbt_hol_dev   TO ROLE dbt_dev_role;
GRANT ALL ON ALL SCHEMAS IN DATABASE dbt_hol_prod  TO ROLE dbt_prod_role;
```

As result of these steps, we should have:

two empty databases: PROD, DEV
two pair of virtual warehouses: two for prod, two for dev workloads
a pair of roles and one user

3. Create a new dbt project in any local folder by running the following commands:
```
$ dbt init dbt_hol
$ cd dbt_hol
```
Configure dbt/Snowflake profiles
Open `~/.dbt/profiles.yml` in text editor and add the following section
```
dbt_hol:
  target: dev
  outputs:
    dev:
      type: snowflake
      ######## Please replace with your Snowflake account name
      account: <your_snowflake_trial_account>
      
      user: dbt_user
      ######## Please replace with your Snowflake dbt user password
      password: <mysecretpassword>
      
      role: dbt_dev_role
      database: dbt_hol_dev
      warehouse: dbt_dev_wh
      schema: public
      threads: 200
    prod:
      type: snowflake
      ######## Please replace with your Snowflake account name
      account: <your_snowflake_trial_account>
      
      user: dbt_user
      ######## Please replace with your Snowflake dbt user password
      password: <mysecretpassword>
      
      role: dbt_prod_role
      database: dbt_hol_prod
      warehouse: dbt_prod_wh
      schema: public
      threads: 200
```
Validate the configuration
Run the following command (in dbt_hol folder):
```
$ dbt debug
$ dbt run
```

Congratulations! You just run your first dbt models on Snowflake!

In this lab, we are going to analyze historical trading performance of a company that has trading desks spread across different regions. As inputs, we are going to leverage datasets available in Knoema Economy Data Atlas that is available in Snowflake Data Marketplace, plus few manual uploads.

We are going to set up the environments from scratch, build scalable pipelines in dbt, establish data tests, and Snowflake and promote code to production. Finally we will use Snowsight to build a simple dashboard to visualize the results.

![image](https://github.com/nikvolynets/snowflake-dbt/assets/151893648/d7effe35-8bfc-4d2c-b7d9-2b4495f965bf)

Let's go to the Snowflake Data Marketplace and find what we need. Today we're going to grab a Knoema Economy Atlas Data.
