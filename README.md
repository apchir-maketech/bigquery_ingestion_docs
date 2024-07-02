# Automated Airflow DAG Generation for BigQuery Data Ingestion: A Configuration-Driven Approach

## Abstract

A dynamic and flexible system designed to automate the generation of Airflow DAGs for Google BigQuery data ingestion tasks. The goal is to transform manual DAG creation processes into a dynamic, configuration-driven model. This system allows for the definition of data ingestion workflows through simple configurations stored in a MySQL database, supporting a wide variety of parameters that cater to various aspects of data extraction, transformation, and loading (ETL) processes.

### Workflow Overview: From Configuration to Execution
> **Step 1**: Define ingestion config <br/>
> **Step 2**: Review and Copy the generated "Release Candidate" Dag into the live dags folder

This conceptual framework not only significantly accelerates the deployment of new data pipelines but also democratizes the process of DAG creation, making it accessible to users with minimal coding expertise by abstracting the complexities of DAG scripting into user-friendly configurations.


## Motivation

In data engineering, the process of creating and managing Airflow DAGs for data ingestion can often become repetitive and time-consuming, especially when dealing with a multitude of tables across various databases. Recognizing this challenge, the motivation behind developing an automated Airflow DAG generator was to streamline the development process and centralize the logic involved in data ingestion workflows. Instead of manually writing DAGs, users can now simply update a configuration in a MySQL database and execute the generator DAG. This approach not only simplifies the development process but also ensures consistency and scalability in managing data ingestion pipelines for Google BigQuery.


## Configuration Management

To further enhance the efficiency and manageability of the DAG generation process, a CI/CD pipeline has been integrated into the system. This pipeline leverages Azure Repos for storing and versioning the configuration files, with each table's configuration specified in a separate YAML file. When changes are pushed to the main branch of the repository, the pipeline automatically triggers a process that parses these YAML configurations and inserts them into the MySQL table designated for DAG configurations. This setup not only centralizes the management of configurations, making it easier to track and audit changes over time but also ensures that any updates to the configurations are seamlessly and automatically reflected in the data ingestion workflows. This CI/CD integration represents a significant step towards automating the entire lifecycle of Airflow DAG management, from configuration through deployment, thereby further reducing manual intervention and enhancing overall system reliability and traceability.


## Workflow

![Architecture](/images/architecture.png)

## Codebase and repos

- [Airflow repo] REDACTED
- [Config repo] REDACTED
- [Control DB] REDACTED


## GCP resources

- [Composer (Airflow) environment] REDACTED
- [GCS Bucket for composer] REDACTED
- [CLoudSQL (ControlDB) instance] REDACTED


## Configuration Options

Template yaml file 
```yaml
# PIPELINE_IGNORE 
# Use the above comment in the first line if you want the pipeline to ignore this yml file.

# The location of the yml file needs to follow the following pattern:
# "source_type/source_name/database/optional_schema.table.yml"
# where source_type must be a "name" in the "controldb.source_type" table,
# source_name must be a "name" in the "controldb.source" table
# database is the source database name
# optional_schema is the source schema name 
#     -- applicable for databases that use schemas, like mssql etc. 
#     -- not applicable for those that don't, like mysql etc.
# and table is the source table name
# Example: "dbtable/ddr_dev/DemandDataRepository/dbo.Bin.yml"


bq_tables:

  # -------------------------------------------------------------------
  # begin bq table config info
  
  - destination_name: ddr_dev                     # must be a "name" in the "controldb.bq_destination" table
    dataset: dbo                                  # target bigquery dataset
    table: Bin                                    # target bigquery table 
    landing_dataset: dbo_stage                    # dataset for the intermediate landing table 
    dag_unit: small_tables                        # determines which dag this goes into
    
    # job config. all fields are optional and if not provided then default values will be used
    job_config: 
      enabled: x                                  # default value: null; possible values: x, null
      source_table_columns: null                  # default value: null; possible values: free text for comma separated column names
      source_table_where_condition: null          # default value: null; possible values: free text for where clause predicate; ex: COUNTRY='US'
      source_extract_type: full                   # default value: full; possible values: full, delta
      extract_query_split_count: null             # default value: null; possible values: any integer; ex: 3
      split_by_column: null                       # default value: null; applicable if extract_query_split_count > 0; will be auto-detected if left null; possible values: name of the source date or datetime column to split on; ex: create_date
      split_lower_limit_value: null               # default value: null; applicable if extract_query_split_count > 0; will be auto-detected if left null; possible values: lower limit value for date or datetime column to split on; ex: '2020-01-01'
      split_upper_limit_value: null               # default value: null; applicable if extract_query_split_count > 0; will be auto-detected if left null; possible values: upper limit value for date or datetime column to split on; ex: '2023-12-31'
      target_create_disposition: CREATE_IF_NEEDED # default value: CREATE_IF_NEEDED; possible values: CREATE_IF_NEEDED, CREATE_NEVER
      target_write_disposition: WRITE_TRUNCATE    # default value: WRITE_MERGE; possible values: WRITE_MERGE, WRITE_APPEND, WRITE_TRUNCATE, WRITE_EMPTY
      target_table_schema_json: null              # default value: null; will be auto-detected if left null; possible values: json schema for bigquery table
      target_table_property_options: null         # default value: null; possible values: free text for bigquery table options; ex: PARTITION BY create_date CLUSTER BY customer_id
      target_table_delta_pointer_column: null     # default value: null; applicable if source_extract_type = delta; possible values: name of the source column to use for delta offset; ex: modified_timestamp
    
    # the folowing section is optional and just for extra info. 
    # it will be ignored by pipeline and wont be loaded into control db
    non_config_notes: 
      any_field: 'any info here' # example: comments: 'my comments'
  
  # end of table config info
  # -------------------------------------------------------------------



  # optional -- add more bq tables if you want to load the same source table into more than 1 bq tables
  - destination_name: ...

```


## Example config and the generated DAG
![Example config and the generated DAG](/images/example_config_and_generated_dag.png)


## Utils

#### Airflow

- [Generate YAML files for config defined in an excel spreadsheet] REDACTED
- [Generate release candidates for a list of Dag Units defined in a csv file] REDACTED
- [Deploy the latest release candidates for a list of Dag IDs defined in a csv file] REDACTED

#### BigQuery

- [Generate bigquery de-dupe script for a list of source mssql tables in a csv file] REDACTED


## Current Challenges

- **CloudSQL Connectivity Issues**: For the creation of new configurations or DAGs, the [CloudSQL instance] REDACTED serving as the control database currently requires enabling public access by allowing the IP CIDR address 0.0.0.0/0. This step is necessary to facilitate connectivity from both Azure and Airflow/Composer environments. Consider implementing a more secure and permanent solution, potentially involving the use of a CloudSQL Proxy. It is crucial to remember that, following the temporary measure of opening the network for public access, it must be promptly restricted again to ensure the system's security.

- **Cloud Composer Infrastructure Limitations**: The existing setup for the Cloud Composer utilizes machines with limited computing capabilities, which may lead to occasional delays in Airflow's DAG parsing process. Although these delays are generally short-lived, resolving spontaneously within a few minutes, enhancing the computational resources of the Composer instances could mitigate these issues more effectively.
  - Note: Also consider upgrading the Composer to a newer Airflow version


## Future Directions

Expanding the scope of the existing framework to encompass file ingestion is a feasible and valuable enhancement. It's worth exploring the integration of this capability into the system.