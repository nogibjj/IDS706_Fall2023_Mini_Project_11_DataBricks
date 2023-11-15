[![install](https://github.com/nogibjj/IDS706_Fall2023_Mini_Project_11_DataBricks/actions/workflows/install.yml/badge.svg)](https://github.com/nogibjj/IDS706_Fall2023_Mini_Project_11_DataBricks/actions/workflows/install.yml)
[![lint](https://github.com/nogibjj/IDS706_Fall2023_Mini_Project_11_DataBricks/actions/workflows/lint.yml/badge.svg)](https://github.com/nogibjj/IDS706_Fall2023_Mini_Project_11_DataBricks/actions/workflows/lint.yml)
[![format](https://github.com/nogibjj/IDS706_Fall2023_Mini_Project_11_DataBricks/actions/workflows/format.yml/badge.svg)](https://github.com/nogibjj/IDS706_Fall2023_Mini_Project_11_DataBricks/actions/workflows/format.yml)
[![test](https://github.com/nogibjj/IDS706_Fall2023_Mini_Project_11_DataBricks/actions/workflows/test.yml/badge.svg)](https://github.com/nogibjj/IDS706_Fall2023_Mini_Project_11_DataBricks/actions/workflows/test.yml)
# IDS706_Fall2023_Mini_Project_11_DataBricks

IDS706 week 11 mini project: Data Pipeline with Databricks

It contains:

- ``IDS706_Mini_Project_11.ipynb`` an Azure Databricks-based Jupyter Notebook that makes use of ``Pandas`` to perform descriptive statistics related to the revenue data of fortune 500 companies over 50 years

- ``.devcontainer`` includes a `Dockerfile` that specifies the configurations of container, and a `devcontainer.json` which is a configuration file used in the context of Visual Studio Code

- ``workflows`` includes `GitHub Actions`, enables automated build, test and deployment for the project

- ``Makefile`` specifies build automation on Linux

- ``requirements.txt`` lists the dependencies, libraries, and specific versions of Python packages required for the project

It also includes ``main.py`` and ``test_main.py`` as sample files to show the functionality of the CI pipeline.

## Databricks data pipeline

The Azure Databricks notebook contains a data pipeline that contains a **data source**, a data processing script and a **data sink**.

### Data source configuration
Configure the data source as below to read data from Delta Table.
```Python
# Define variables used in code below
file_path = "/databricks-datasets/structured-streaming/events"
username = spark.sql("SELECT regexp_replace(current_user(), '[^a-zA-Z0-9]', '_')").first()[0]
table_name = f"{username}_etl_quickstart"
checkpoint_path = f"/tmp/{username}/_checkpoint/etl_quickstart"

# Clear out data from previous demo execution
spark.sql(f"DROP TABLE IF EXISTS {table_name}")
dbutils.fs.rm(checkpoint_path, True)

# Configure Auto Loader to ingest JSON data to a Delta table
(spark.readStream
  .format("cloudFiles")
  .option("cloudFiles.format", "json")
  .option("cloudFiles.schemaLocation", checkpoint_path)
  .load(file_path)
  .select("*", col("_metadata.file_path").alias("source_file"), current_timestamp().alias("processing_time"))
  .writeStream
  .option("checkpointLocation", checkpoint_path)
  .trigger(availableNow=True)
  .toTable(table_name))
```

### Data Processing
Performs descriptive statistics related to the revenue data of fortune 500 companies over 50 years.

### Data sink
Write the Spark DataFrame (converted from Pandas) to the Delta table.

```Python
spark_df.write.format("delta").mode("overwrite").save(delta_table_path)
```
