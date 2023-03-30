# Data Ingest

- Identify the different sources of data and assess their compatibility with platform 
- Determine the data ingestion method for each data source, whether it be 
	- file transfer
	- API connection
	- database connection
	- other?
- Essential to make sure we can reliably and securely collect data from each source and ensure that the data is in a standard format.

## Tools

Needs to deploy on VPC to avoid moving PHI data outside VPC

- Azure Data Factory
	- built-in mapping, transformation
	- Custom code
	 	- Functions for event response
		 	- Languages: C#, Java, Python
	 	- Databricks for transformation & ML workloads
		 	- Spark/Scala, supports Python
		 	- Need to manage cluster configs
	- Loading Zone (?)
		- Blob Storage
		- SQL
		- Cosmos DB
		- Data Lake
- NiFi
- Talend
- Informatica 
- Google Cloud Dataflow
- AWS Glue


# Data Transformation

- Determine the best approach to transform the data to a standardized format that is compatible with our analytics platform. 
- Depending on the source data's format, we may need to use various tools:
	- Scripting languages (Python)
	- ETL software 
		- open-source (Apache Nifi or Kafka)
		- 3rd party (Talend/Informatica)
- Ensure that the data transformation process is customizable and can handle a variety of data types.

# De-identification

- Determine PHI regulations (HIPAA or GDPR) 
- Eval. de-id techniques
	- masking vs. anonymization
- De-id process
	- configurable
	- auditable

# Extraction of datasets for data sales

- Determine use case to extract specific datasets from integrated data
	- automated
	- auditable
	- secure

# Creation of tests and test data
- ETL pipeline tests
	- create test data
	- create test cases for different data sources and transformations