# Plan

1. Identify the source systems: The data is stored in a Databricks environment on Azure. Access to the Databricks environment can be established using a virtual private network (VPN) or Azure ExpressRoute to ensure that data remains within your Azure VPC.

1. Establish connectivity: Use a JDBC connector to connect to the Databricks environment and extract data from the OMOP v5.3 schema using SQL queries.

1. Cohort selection: Use a custom script or tool to select data only for the approved patient identifiers for each cohort defined by the Data Collaborator.

1. Extract data: Use an ETL tool such as Apache NiFi to extract data from the Databricks environment and move it to a landing zone within your Azure VPC. Use NiFi's secure connectivity features to ensure that data is not distributed outside of your VPC.

1. Data transformation: Once the data is in the landing zone, perform the necessary data transformations to conform to the desired data model and de-identify the data as required. Use an ETL tool such as Apache NiFi or Azure Data Factory to perform this step.

1. Data validation: Validate the transformed data to ensure that it meets the required quality and accuracy standards. This may include running data validation scripts or performing data profiling. Use tools such as Apache NiFi or Databricks to perform this step.

1. Load data: Finally, load the transformed and validated data into the target data store or analytics platform. This may include loading the data into a data warehouse such as Azure Synapse Analytics or a data lake such as Azure Data Lake Storage. Use an ETL tool such as Apache NiFi or Azure Data Factory to perform this step.

1. Incremental updates: Schedule the ETL pipeline to run once a week on Sunday and perform incremental updates to the data by identifying new rows based on the "load_dt" timestamp column. Use a custom script or tool to calculate the rate of increase in row count per table per cohort and adjust the pipeline accordingly.

1. Sandbox schema: Use a dedicated sandbox schema in your target data store or analytics platform to create tables and views as needed for the data transformations.