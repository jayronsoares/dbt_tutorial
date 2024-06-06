### Steps:
1. **Data Extraction**: Data is extracted from the on-premises PostgreSQL database using `pg_dump` command line utility.
2. **Data Loading**: The extracted data is loaded into an AWS RDS PostgreSQL database using `psql` command line utility.
3. **Data Transformation**: Data cleaning and transformation are performed using dbt. The data is loaded into staging tables in the AWS RDS PostgreSQL database, where duplicates are removed, missing values are handled, and date formats are standardized.
4. **Final Data Load**: The transformed data is loaded into Snowflake data warehouse. Final tables such as `dim_order`, `dim_customer`, and `fact_sales` are created, and data from staging tables is inserted into these tables.

### Directory Structure:
```
etl_pipeline/
│
├── main.py
├── dbt_utils.py
├── etl_utils.py
├── dbt_profiles/
│   └── profiles.yml
├── models/
│   ├── dim_order.sql
│   ├── dim_customer.sql
│   ├── fact_sales.sql
│   ├── stg_order.sql
│   └── stg_sales.sql
└── scripts/
    ├── extract_data.sh
    └── load_data.sh
```

### 1. `main.py`
The main orchestration script.

```python
import os
from etl_utils import set_on_prem_env, set_aws_rds_env, run_shell_script
from dbt_utils import (
    install_dbt,
    init_dbt_project,
    configure_dbt_profile,
    run_dbt_models,
    configure_snowflake_profile,
    run_dbt_snowflake_models
)

def main():
    try:
        # Data Extraction
        set_on_prem_env()
        run_shell_script('scripts/extract_data.sh')

        # Data Loading
        set_aws_rds_env()
        run_shell_script('scripts/load_data.sh')
        
        # Data Transformation
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
        run_dbt_models()
        
        # Final Data Load
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
        run_dbt_snowflake_models()
    except Exception as e:
        print(f"ETL process failed: {e}")

if __name__ == "__main__":
    main()
```

### 2. `etl_utils.py`
Utility functions for ETL tasks.

```python
import os
import subprocess

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

def run_shell_script(script_path):
    try:
        subprocess.run(['bash', script_path], check=True)
    except subprocess.CalledProcessError as e:
        print(f"Error running shell script {script_path}: {e}")
        raise
```

### 3. `dbt_utils.py`
Utility functions for dbt tasks.

```python
import subprocess
import yaml
import os

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
                    'host': host,
                    'user': user,
                    'password': password,
                    'dbname': dbname,
                    'port': port,
                    'schema': schema
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
                    'account': account,
                    'user': user,
                    'password': password,
                    'role': role,
                    'warehouse': warehouse,
                    'database': database,
                    'schema': schema,
                    'threads': 1,
                    'client_session_keep_alive': False
                }
            }
        }
    }
    with open(os.path.expanduser('~/.dbt/profiles.yml'), 'w') as f:
        yaml.dump(profile, f)

def run_dbt_snowflake_models():
    try:
        subprocess.run(['dbt', 'run'], check=True)
    except subprocess.CalledProcessError as e:
        print(f"Error running dbt models: {e}")
        raise
```

### 4. `dbt_profiles/profiles.yml`

```yaml
your_project:
  target: dev
  outputs:
    dev:
      type: postgres
      host: your_aws_rds_host
      user: your_aws_rds_user
      password: your_aws_rds_password
      dbname: your_aws_rds_db
      port: your_aws_rds_port
      schema: public
    staging:
      type: postgres
      host: staging-db.example.com
      user: your_staging_user
      password: your_staging_password
      dbname: retail_stg_db
      port: 5432
      schema: public
    snowflake:
      type: snowflake
      account: your_account.region.snowflakecomputing.com
      user: your_snowflake_user
      password: your_snowflake_password
      role: your_snowflake_role
      warehouse: your_snowflake_warehouse
      database: your_snowflake_db
      schema: public
      threads: 1
      client_session_keep_alive: False
```

### 5. `models/`

#### `models/stg_order.sql`
```sql
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
```

#### `models/stg_sales.sql`
```sql
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
```

#### `models/dim_order.sql`
```sql
-- models/dim_order.sql
insert into {{ ref('dim_order') }}
select * from {{ ref('stg_order') }};
```

#### `models/dim_customer.sql`
```sql
-- models/dim_customer.sql
insert into {{ ref('dim_customer') }}
select distinct customer_id as id, 'Unknown' as name, 'Unknown' as email
from {{ ref('stg_order') }};
```

#### `models/fact_sales.sql`
```sql
-- models/fact_sales.sql
insert into {{ ref('fact_sales') }}
select * from {{ ref('stg_sales') }};
```

### 6. `scripts/extract_data.sh`
Shell script to extract data from on-premises PostgreSQL.

```bash
#!/bin/bash
pg_dump -h $PGHOST -U $PGUSER -d $PGDATABASE -F c -b -v -f /path/to/backup.dump
```

### 7. `scripts/load_data.sh`
Shell script to load data into AWS RDS PostgreSQL.

```bash
#!/bin/bash
pg_restore -h $PGHOST -U $PGUSER -d $PGDATABASE -v /path/to/backup.dump
```
