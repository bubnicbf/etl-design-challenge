# Diagram steps

1. Extract the data from the source system using an Azure Data Factory pipeline, configured with a source dataset to read the raw data from the source system.

- Extract Data from Databricks using Azure Data Factory
	- Configure Source Dataset
	- Use Copy Activity to copy data to landing zone
- Move Data from Landing Zone to NiFi using GetAzureBlobStorage Processor

2. Validate the extracted data using custom Python scripts or built-in validation processors in Data Factory, such as the Validate Data Flow activity or the Azure Data Factory Data Flow expression builder.

- Transform Data using NiFi
	- QueryDatabaseTable processor to extract data from each table
	- ExecuteScript processor to apply custom transformations
	- ConvertRecord processor to change the format of data
	- SplitText processor to separate values within fields
- De-identify data as necessary
	- Apply data masking or encryption as necessary

3. Transform the validated data using the Data Flow activity in Azure Data Factory, which provides a visual interface to build data transformations using a drag-and-drop interface. This may include additional data transformations, de-identification, or data enrichment.

- Validate Data using NiFi
	- Develop validation scripts to verify data quality
	- Configure validation processors to execute validation logic
	- Configure logging and alerts for validation errors
- Ensure compliance with regulations and policies
	- Conduct regular reviews of de-identified data for potential re-identification risks
	- Automate reports for human review

4. Load the transformed data into your target data store or analytics platform using the appropriate sink connector in Azure Data Factory, such as the Azure Synapse Analytics connector or the Azure SQL Database connector.

- Load Data using Azure Data Factory
	- Configure Sink Dataset
	- Use Copy Activity to move data from landing zone to sink
- Configure Sink Processor to load transformed data into target data store or analytics platform
- Configure appropriate data loading settings in the sink processor
	- Column mapping
	- File format
	- Batch size
- Start the Azure Data Factory pipeline to load transformed data into target data store or analytics platform

5. Configure the appropriate data loading settings in the sink connector, such as the column mapping, file format, and batch size. You can also configure performance optimization settings, such as parallelism and partitioning, to optimize the data loading process.

- Automate Workflow using Apache Airflow
	- Create Airflow DAG
	- Configure tasks for each step of ETL solution
	- Schedule DAG to run on regular basis

6. Schedule the Azure Data Factory pipeline to run at regular intervals to keep the target data store or analytics platform up-to-date with the latest data from the source system.

- Use Airflow to automate the ETL workflow.
- Configure DAGs (Directed Acyclic Graphs) to define the sequence of tasks and dependencies.
- Use operators to define the tasks to be executed, such as PythonOperator or BashOperator.
- Schedule the DAGs to run at specific intervals or trigger them manually.

7. Monitor the ETL process for any errors or performance issues using Azure Data Factory's built-in monitoring and logging features, such as the Azure Data Factory Metrics and Activity Runs pages.