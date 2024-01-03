# ETL-Pipeline-S3_SnowPipe
python ETL pipeline to load file from s3 using snow pipe


![Screen Shot 2024-01-02 at 11 33 12 PM](https://github.com/devaa07/ETL-Pipeline-S3_SnowPipe/assets/126756574/25467bef-47da-4c4e-a91f-2bcd8be17488)


# Step 1: Create an EC2 instance and install the below dependencies
          sudo apt update
          sudo apt install python3-pip
          sudo apt install python3.10-venv
          python3 -m venv redfin_venv
          source redfin_venv/bin/activate
          pip install pandas
          pip install boto3
          pip install --upgrade awscli
          pip install apache-airflow

# Step 2: create s3 bucket and upload the spark code inside s3 bucket. Update the DAG code
          redfin_analytics.py

# Step 4: script loads the transformed data file into other S3 bucket 

# Step 5: Create the Snowflake Warehouse, database, schema, CSV file format, table and snow pipe. refer the below code

        DROP DATABASE IF EXISTS redfin_database_1;
        CREATE DATABASE redfin_database_1;
        -- CREATE WAREHOUSE redfin_warehouse;
        CREATE SCHEMA redfin_schema;
        // Create Table
        TRUNCATE TABLE redfin_database_1.redfin_schema.redfin_table;
        CREATE OR REPLACE TABLE redfin_database_1.redfin_schema.redfin_table (
        period_begin DATE,
        period_end DATE,
        period_duration INT,
        region_type STRING,
        region_type_id INT,
        table_id INT,
        is_seasonally_adjusted STRING,
        city STRING,
        state STRING,
        state_code STRING,
        property_type STRING,
        property_type_id INT,
        median_sale_price FLOAT,
        median_list_price FLOAT,
        median_ppsf FLOAT,
        median_list_ppsf FLOAT,
        homes_sold FLOAT,
        inventory FLOAT,
        months_of_supply FLOAT,
        median_dom FLOAT,
        avg_sale_to_list FLOAT,
        sold_above_list FLOAT,
        parent_metro_region_metro_code STRING,
        last_updated DATETIME,
        period_begin_in_years STRING,
        period_end_in_years STRING,
        period_begin_in_months STRING,
        period_end_in_months STRING
        );
        SELECT *
        FROM redfin_database_1.redfin_schema.redfin_table LIMIT 10;
        
        SELECT COUNT(*) FROM redfin_database_1.redfin_schema.redfin_table
        -- DESC TABLE redfin_database.redfin_schema.redfin_table;
        
        
        // Create file format object
        CREATE SCHEMA file_format_schema;
        CREATE OR REPLACE file format redfin_database_1.file_format_schema.format_csv
            type = 'CSV'
            field_delimiter = ','
            RECORD_DELIMITER = '\n'
            skip_header = 1
            -- error_on_column_count_mismatch = FALSE;
            
        // Create staging schema
        CREATE SCHEMA external_stage_schema;
        // Create staging
        -- DROP STAGE redfin_database.external_stage_schema.redfin_ext_stage_yml;
        CREATE OR REPLACE STAGE redfin_database_1.external_stage_schema.redfin_ext_stage_yml 
            url="s3://redfin-transform-zone-yml/"
            credentials=(aws_key_id='xxxx'
            aws_secret_key='xxxx')
            FILE_FORMAT = redfin_database_1.file_format_schema.format_csv;
        
        list @redfin_database_1.external_stage_schema.redfin_ext_stage_yml;
        
        // Create schema for snowpipe
        -- DROP SCHEMA redfin_database.snowpipe_schema;
        CREATE OR REPLACE SCHEMA redfin_database_1.snowpipe_schema;
        
        // Create Pipe
        CREATE OR REPLACE PIPE redfin_database_1.snowpipe_schema.redfin_snowpipe
        auto_ingest = TRUE
        AS 
        COPY INTO redfin_database_1.redfin_schema.redfin_table
        FROM @redfin_database_1.external_stage_schema.redfin_ext_stage_yml;
        
        DESC PIPE redfin_database_1.snowpipe_schema.redfin_snowpipe;
          
# Step 6: Create the S3 event notification and give the Snow pipe SqsARN into event settings

![Screen Shot 2024-01-02 at 11 42 18 PM](https://github.com/devaa07/ETL-Pipeline-S3_SnowPipe/assets/126756574/91d0b1f9-d82b-4eab-93d9-4038b8a24cc8)

![Screen Shot 2024-01-02 at 11 43 34 PM](https://github.com/devaa07/ETL-Pipeline-S3_SnowPipe/assets/126756574/3794279b-5dbf-47b1-a82d-ba0ce2ffd7da)

# Step 7: Trigger the DAG and see the results

![Screen Shot 2024-01-02 at 11 44 46 PM](https://github.com/devaa07/ETL-Pipeline-S3_SnowPipe/assets/126756574/688ff77c-4120-4f88-970e-89c8611e7f75)

# Step 8: create the snowflake connection in powerBI and peform some analytics

![Screen Shot 2024-01-02 at 11 45 42 PM](https://github.com/devaa07/ETL-Pipeline-S3_SnowPipe/assets/126756574/ff9cb5a9-42c1-4350-8853-7fe81e711b30)


