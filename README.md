# ETL Design Challenge

CuriMeta firmly believes that engineers are designers that happen to implement their designs into functional technologies. With that in mind, we would like you to design an approach to the following data ingestion use case drawing on your experience and knowledge of the data engineering tool landscape.

- Technology recommendation, evaluation and selection
- Authoring and testing data pipelines and related software
	- Data ingest
	- Data transformation
	- De-Identification
	- Extraction of datasets for data sales
	- Creation of tests and test data
- Integrating solutions to meet the data processing needs of all possible modalities of medical data
	- relational, narrative text, imaging and genomics data.

## Constraints

- Must operate on Azure cloud
- Must avoid use of SAAS solutions that would require us to distribute PHI data outside of our Azure VPC.
	- Tools that can be deployed in our VPC are acceptable.
- Contributor Data is stored in a Databricks environment on Azure on a Subscription owned by our Data Collaborator.
	- The data schema is OMOP v5.3 and the schema is named “omop”
		- https://ohdsi.github.io/CommonDataModel/cdm53.html
		- Focus on only the 9 Clinical Data Tables listed below.
	- Data is loaded to this schema once a week on Sunday and each data row has a “load_dt” timestamp column indicating when it was written to Databricks.
		- New data rows are added at a rate of 0.25% per week increase in row count per table per cohort
	- Databricks provides an SQL interface to support execution of select queries.
	- You have a dedicated “sandbox” schema in which to create tables and views as needed.
- You cannot replicate the entire repository
	- CuriMeta is required to ingest data based on defined patient ID lists for each data project (cohort definitions) defined during negotiation with our data consumers and approved by our data collaborators.
	- These cohorts contain an exact list of patient identifiers for whom we are approved to ingest data.
	- New cohorts are approved by data collaborators on ongoing basis

# The Challenge

Define a data ingest approach that allows us to ingest all data for new cohorts while ingesting only newly loaded data for existing cohorts.

- Focus on A-C on the example workflow.
- Recommend tools / languages / data stores.
	- Tell us why you selected them and list one alternative that you might consider
	- Any Azure service or Azure Marketplace vendor can be considered
- Diagram your recommended ingest data flow
- Define how we would automate/orchestrate this flow
- Define how you would handle new cohorts vs newly loaded data for existing cohorts
	- This is very important.

