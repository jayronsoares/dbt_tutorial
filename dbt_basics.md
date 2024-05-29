
### 1. Setting Up Your dbt Project

Before you begin using dbt, you need to set up a project. Here’s how you can do it:

1. **Install dbt**: First, install dbt using pip.
   ```sh
   pip install dbt
   ```
2. **Initialize a New Project**: Create a new dbt project.
   ```sh
   dbt init my_dbt_project
   ```
   This command sets up the directory structure for your project, including folders for models, tests, and more.

3. **Configure Your Profile**: Create a `profiles.yml` file to store database connection details.
   ```yaml
   my_profile:
     outputs:
       dev:
         type: postgres
         host: localhost
         user: my_user
         password: my_password
         dbname: my_db
         schema: public
     target: dev
   ```
   This file allows dbt to connect to your database. Replace the placeholder values with your actual database credentials.

### 2. Creating Your First Model

In dbt, a model is simply a SQL file that contains a SELECT statement. Here’s an example of creating a simple model:

1. **Create a SQL File**: Create a new file in the `models` directory called `my_first_model.sql`.
   ```sql
   -- models/my_first_model.sql
   SELECT
     id,
     name,
     created_at
   FROM
     source_table
   ```
2. **Run the Model**: Execute the model to create a table or view in your database.
   ```sh
   dbt run --models my_first_model
   ```
   This command runs the SQL in `my_first_model.sql` and materializes the results in your data warehouse.

### 3. Using Jinja for Templating

dbt uses the Jinja templating language to add programmatic logic to your SQL. Here’s a simple example:

1. **Create a Model with Jinja**: Use Jinja to dynamically create a column.
   ```sql
   -- models/jinja_example.sql
   SELECT
     id,
     name,
     created_at,
     CASE
       WHEN created_at >= '{{ var("cutoff_date") }}' THEN 'Recent'
       ELSE 'Old'
     END AS status
   FROM
     source_table
   ```
2. **Run the Model with Variables**: Pass variables to your model when running it.
   ```sh
   dbt run --models jinja_example --vars '{"cutoff_date": "2023-01-01"}'
   ```

### 4. Testing Your Data

dbt provides built-in testing capabilities to ensure data quality. Here’s how to add tests to your project:

1. **Define a Test**: Create a YAML file to define your tests.
   ```yaml
   -- models/schema.yml
   version: 2
   models:
     - name: my_first_model
       tests:
         - unique:
             column_name: id
         - not_null:
             column_name: id
   ```
2. **Run the Tests**: Execute your tests to check for data issues.
   ```sh
   dbt test --models my_first_model
   ```

### 5. Documenting Your Models

Documentation is crucial for maintaining and understanding your dbt project. Here’s how to add documentation:

1. **Add Documentation**: Create doc blocks in your model SQL files.
   ```sql
   -- models/my_first_model.sql
   {{
     config(
       description="This model selects data from the source table and adds a status column."
     )
   }}
   SELECT
     id,
     name,
     created_at
   FROM
     source_table
   ```
2. **Generate and View Documentation**: Generate and view the documentation using dbt.
   ```sh
   dbt docs generate
   dbt docs serve
   ```
