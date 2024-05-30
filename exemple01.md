### Steps:
1. **Data Extraction**: Data is extracted from the on-premises PostgreSQL database using `pg_dump` command line utility.
2. **Data Loading**: The extracted data is loaded into an AWS RDS PostgreSQL database using `psql` command line utility.
3. **Data Transformation**: Data cleaning and transformation are performed using dbt. The data is loaded into staging tables in the AWS RDS PostgreSQL database, where duplicates are removed, missing values are handled, and date formats are standardized.
4. **Final Data Load**: The transformed data is loaded into Snowflake data warehouse. Final tables such as `dim_order`, `dim_customer`, and `fact_sales` are created, and data from staging tables is inserted into these tables.

### Key Components:
- **Functional Approach**: The ETL pipeline is implemented using a series of modular functions, each responsible for a specific task such as data extraction, loading, and transformation. This approach ensures clarity, reusability, and maintainability of the code.
- **Error Handling**: Error handling is implemented using try-except blocks to catch and handle exceptions that may occur during data extraction, loading, or transformation. Error messages are printed to provide feedback on the execution status.
- **Configuration Management**: Configuration details such as database connections and credentials are stored in environment variables and YAML configuration files (`profiles.yml`). This allows for easy customization and adaptation of the ETL pipeline for different environments.
- **Lazy Evaluation**: Although not extensively utilized in this example, lazy evaluation principles can be applied to handle streams of data efficiently, especially in scenarios with large datasets.

You're right. Let's include access to the staging PostgreSQL database in the `profiles.yml` file for dbt, and also import necessary modules in the Python script. Here's the updated version:

```python
import os
import subprocess
import yaml

def set_on_prem_env():
    os.environ['PGHOST'] = 'onprem-db.example.com'
    os.environ['PGPORT'] = '5432'
    os.environ['PGDATABASE'] = 'onprem_db'
    os.environ['PGUSER'] = 'onprem_user'
    os.environ['PGPASSWORD'] = 'onprem_pwd'

def set_aws_rds_env():
    os.environ['PGHOST'] = 'aws-rds-db-instance.cfn3kr4lz2n4.us-east-1.rds.amazonaws.com'
    os.environ['PGPORT'] = '5432'
    os.environ['PGDATABASE'] = 'aws_rds_db'
    os.environ['PGUSER'] = 'aws_rds_user'
    os.environ['PGPASSWORD'] = 'aws_rds_pwd'

def install_dbt():
    subprocess.run(['pip', 'install', 'dbt'], check=True)

def init_dbt_project(project_name):
    subprocess.run(['dbt', 'init', project_name], check=True)

def configure_dbt_profile(profile_name, host, user, password, dbname, port, schema):
    profile = {
        profile_name: {
            'target': 'dev',
            'outputs': {
                'dev': {
                    'type': 'postgres',
                    'host': localhost,
                    'user': user,
                    'password': password,
                    'dbname': dbname,
                    'port': 5432,
                    'schema': public
                },
                'staging': {
                    'type': 'postgres',
                    'host': 'staging-db.example.com',
                    'user': 'your_staging_user',
                    'password': 'your_staging_password',
                    'dbname': 'retail_stg_db',
                    'port': '5432',
                    'schema': 'public'
                }
            }
        }
    }
    with open(os.path.expanduser('~/.dbt/profiles.yml'), 'w') as f:
        yaml.dump(profile, f)

def create_dbt_models(project_dir):
    order_sql = """
    -- models/stg_order.sql
    with cleaned_order as (
        select distinct
            id,
            customer_id,
            coalesce(date::date, '1970-01-01'::date) as date,
            amount
        from {{ source('staging', 'order') }}
    )
    select * from cleaned_order;
    """
    sales_sql = """
    -- models/stg_sales.sql
    with cleaned_sales as (
        select distinct
            id,
            order_id,
            coalesce(date::date, '1970-01-01'::date) as date,
            amount
        from {{ source('staging', 'sales') }}
    )
    select * from cleaned_sales;
    """
    with open(f'{project_dir}/models/stg_order.sql', 'w') as f:
        f.write(order_sql)
    with open(f'{project_dir}/models/stg_sales.sql', 'w') as f:
        f.write(sales_sql)

def run_dbt_models():
    try:
        subprocess.run(['dbt', 'run'], check=True)
    except subprocess.CalledProcessError as e:
        print(f"Error running dbt models: {e}")
        raise

def configure_snowflake_profile(profile_name, account, user, password, role, warehouse, database, schema):
    profile = {
        profile_name: {
            'target': 'dev',
            'outputs': {
                'dev': {
                    'type': 'snowflake',
                    'account': your_account.region.snowflakecomputing.com,
                    'user': user,
                    'password': password,
                    'role': retail_role,
                    'warehouse': warehouse,
                    'database': database,
                    'schema': public,
                    'threads': 1,
                    'client_session_keep_alive': False
                }
            }
        }
    }
    with open(os.path.expanduser('~/.dbt/profiles.yml'), 'w') as f:
        yaml.dump(profile, f)

def create_snowflake_tables(project_dir):
    dim_order_sql = """
    -- models/dim_order.sql
    insert into {{ ref('dim_order') }}
    select * from {{ ref('stg_order') }};
    """
    dim_customer_sql = """
    -- models/dim_customer.sql
    insert into {{ ref('dim_customer') }}
    select distinct customer_id as id, 'Unknown' as name, 'Unknown' as email
    from {{ ref('stg_order') }};
    """
    fact_sales_sql = """
    -- models/fact_sales.sql
    insert into {{ ref('fact_sales') }}
    select * from {{ ref('stg_sales') }};
    """
    with open(f'{project_dir}/models/dim_order.sql', 'w') as f:
        f.write(dim_order_sql)
    with open(f'{project_dir}/models/dim_customer.sql', 'w') as f:
        f.write(dim_customer_sql)
    with open(f'{project_dir}/models/fact_sales.sql', 'w') as f:
        f.write(fact_sales_sql)

def run_dbt_snowflake_models():
    try:
        subprocess.run(['dbt', 'run'], check=True)
    except subprocess.CalledProcessError as e:
        print(f"Error running dbt models: {e}")
        raise

def main():
    try:
        set_on_prem_env()
        extract_data()

        set_aws_rds_env()
        load_data()
        
        install_dbt()
        project_name = 'your_project'
        init_dbt_project(project_name)
        
        configure_dbt_profile(
            profile_name=project_name,
            host='your_aws_rds_host',
            user='your_aws_rds_user',
            password='your_aws_rds_password',
            dbname='your_aws_rds_db',
            port='your_aws_rds_port',
            schema='public'
        )
        create_dbt_models(project_name)
        run_dbt_models()
        
        configure_snowflake_profile(
            profile_name=project_name,
            account='your_snowflake_account',
            user='your_snowflake_user',
            password='your_snowflake_password',
            role='your_snowflake_role',
            warehouse='your_snowflake_warehouse',
            database='your_snowflake_db',
            schema='public'
        )
        create_snowflake_tables(project_name)
        run_dbt_snowflake_models()
    except Exception as e:
        print(f"ETL process failed: {e}")

if __name__ == "__main__":
    main()
```
