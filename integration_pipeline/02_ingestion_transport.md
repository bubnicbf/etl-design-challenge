# Ingestion and Transport

## Establish Connectors

Use a JDBC connector to connect to the Databricks environment and extract data from the OMOP v5.3 schema using SQL queries.

1. Identify the JDBC driver: To use a JDBC connector, you need to identify the appropriate JDBC driver for your Databricks environment. Databricks provides a JDBC driver for Apache Spark that you can download from the Databricks website.

2. Set up the JDBC connector: Once you have the JDBC driver, you need to set up the JDBC connector. This involves configuring the driver and creating a connection to the Databricks environment. The exact steps will depend on the specific JDBC connector and ETL tool you're using, but generally involve specifying the JDBC driver class, the database URL, and the authentication credentials.

3. Connect to the OMOP v5.3 schema: Once the JDBC connector is set up, you can connect to the OMOP v5.3 schema in the Databricks environment using SQL queries. This can be done using a SQL client or an ETL tool that supports JDBC connectors, such as Apache NiFi or Azure Data Factory.

## Cohort selection 

Use a custom script or tool to select data only for the approved patient identifiers for each cohort defined by the Data Collaborator.

1. Obtain the cohort criteria: You need to obtain the cohort criteria from the Data Collaborator for each approved cohort. The cohort criteria define the conditions that patients must meet to be included in each cohort. This may include demographic information, clinical diagnoses, procedures, medications, or other criteria.

2. Create a script or tool: Once you have the cohort criteria, you can create a custom script or tool that applies the criteria as a filter before extracting any data. This can be done using a scripting language such as Python, or a tool such as Apache NiFi or Azure Data Factory.

3. Apply the cohort filter: The script or tool should apply the cohort filter to the SQL queries used to extract data from the Databricks environment. This can be done using a WHERE clause in the SQL query that includes the cohort criteria. For example, you can use the following SQL query to extract data from the "death" table for patients who meet the criteria for cohort 1:

````
SELECT *
FROM omop.death
WHERE person_id IN (SELECT person_id FROM approved_cohort_1)
  AND load_dt BETWEEN load_index AND NOW();
````
This will extract data only for the patient identifiers in the approved_cohort_1 table since the last data pull, indicated by the load_index. 

4. Schedule cohort updates: You need to schedule regular updates to the approved cohort lists based on the Data Collaborator's approval process. This may involve updating the cohort criteria, adding or removing patient identifiers, or creating new cohorts.

## Extractors

1. Configure NiFi: First, you need to configure NiFi with the appropriate JDBC driver for the Databricks environment and set up a connection to the OMOP schema. This involves creating a JDBC connection pool, configuring the driver class, the database URL, and the authentication credentials. Next you'll need to configured a DBCP connection pool in NiFi to connect to the Databricks environment.

2. Create a dataflow: Once NiFi is configured, you can create a dataflow to extract data from the OMOP tables and move it to a landing zone within your Azure VPC. The dataflow should include processors to extract data from each of the OMOP tables, apply any necessary data transformations, and move the data to a landing zone.

3. Use secure connectivity: To ensure that data is not distributed outside of your VPC, you need to use NiFi's secure connectivity features. This may include using HTTPS or SSL/TLS for data encryption, using authentication and authorization features to control access to the data, and using network security features such as firewalls or virtual private networks to control access to the NiFi server.

4. Examples for each OMOP table: Here are examples for extracting data from each of the OMOP tables using NiFi. The processors are configured to use a JDBC connection pool to connect to the Databricks environment, and to move the data to a specified directory in the Azure VPC using PutFile processors. The example SQL queries select all columns from each table and apply the cohort filter using a WHERE clause. Note that you will need to modify the queries and processor configurations to fit your specific environment and requirements. Use a QueryDatabaseTable processor to extract data from each table and move it to a landing zone within your Azure VPC.

````
{
  "name": "OMOP Data Extraction",
  "comment": "Extracts data from the OMOP tables and moves it to a landing zone within the Azure VPC.",
  "processors": [
    {
      "id": "01",
      "type": "org.apache.nifi.processors.standard.QueryDatabaseTable",
      "name": "Extract Death Data",
      "config": {
        "Scheduling Strategy": "timer-driven",
        "Scheduling Period": "1 day",
        "Query": "SELECT * FROM omop.death WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
        "Maximum-value Columns": "load_dt",
        "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
        "DBCP Connection Pool": "Databricks Connection Pool"
      },
      "relationships": {
        "success": ["02"]
      }
    },
    {
      "id": "02",
      "type": "org.apache.nifi.processors.standard.PutFile",
      "name": "Save Death Data",
      "config": {
        "Directory": "/landing_zone/death/",
        "Conflict Resolution Strategy": "ignore"
      },
      "relationships": {}
    },
    {
      "id": "03",
      "type": "org.apache.nifi.processors.standard.QueryDatabaseTable",
      "name": "Extract Person Data",
      "config": {
        "Scheduling Strategy": "timer-driven",
        "Scheduling Period": "1 day",
        "Query": "SELECT * FROM omop.person WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
        "Maximum-value Columns": "load_dt",
        "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
        "DBCP Connection Pool": "Databricks Connection Pool"
      },
      "relationships": {
        "success": ["04"]
      }
    },
    {
      "id": "04",
      "type": "org.apache.nifi.processors.standard.PutFile",
      "name": "Save Person Data",
      "config": {
        "Directory": "/landing_zone/person/",
        "Conflict Resolution Strategy": "ignore"
      },
      "relationships": {}
    },
    {
      "id": "05",
      "type": "org.apache.nifi.processors.standard.QueryDatabaseTable",
      "name": "Extract Observation Data",
      "config": {
        "Scheduling Strategy": "timer-driven",
        "Scheduling Period": "1 day",
        "Query": "SELECT * FROM omop.observation WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
        "Maximum-value Columns": "load_dt",
        "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
        "DBCP Connection Pool": "Databricks Connection Pool"
      },
      "relationships": {
        "success": ["06"]
      }
    },
    {
      "id": "06",
	  "type": "org.apache.nifi.processors.standard.PutFile",
	  "name": "Save Observation Data",
	  "config": {
	    "Directory": "/landing_zone/observation/",
	    "Conflict Resolution Strategy": "ignore"
	  },
	  "relationships": {}
	},
	{
	  "id": "07",
	  "type": "org.apache.nifi.processors.standard.QueryDatabaseTable",
	  "name": "Extract Procedure_occurrence Data",
	  "config": {
	    "Scheduling Strategy": "timer-driven",
	    "Scheduling Period": "1 day",
	    "Query": "SELECT * FROM omop.procedure_occurrence WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
	    "Maximum-value Columns": "load_dt",
	    "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
	    "DBCP Connection Pool": "Databricks Connection Pool"
	  },
	  "relationships": {
	    "success": ["08"]
	  }
	},
	{
	  "id": "08",
	  "type": "org.apache.nifi.processors.standard.PutFile",
	  "name": "Save Procedure_occurrence Data",
	  "config": {
	    "Directory": "/landing_zone/procedure_occurrence/",
	    "Conflict Resolution Strategy": "ignore"
	  },
	  "relationships": {}
	},
	{
	  "id": "09",
	  "type": "org.apache.nifi.processors.standard.QueryDatabaseTable",
	  "name": "Extract Visit_occurrence Data",
	  "config": {
	    "Scheduling Strategy": "timer-driven",
	    "Scheduling Period": "1 day",
	    "Query": "SELECT * FROM omop.visit_occurrence WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
	    "Maximum-value Columns": "load_dt",
	    "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
	    "DBCP Connection Pool": "Databricks Connection Pool"
	  },
	  "relationships": {
	    "success": ["10"]
	  }
	},
	{
	  "id": "10",
	  "type": "org.apache.nifi.processors.standard.PutFile",
	  "name": "Save Visit_occurrence Data",
	  "config": {
	    "Directory": "/landing_zone/visit_occurrence/",
	    "Conflict Resolution Strategy": "ignore"
	  },
	  "relationships": {}
	},
	{
	  "id": "11",
	  "type": "org.apache.nifi.processors.standard.QueryDatabaseTable",
	  "name": "Extract Specimen Data",
	  "config": {
	    "Scheduling Strategy": "timer-driven",
	    "Scheduling Period": "1 day",
	    "Query": "SELECT * FROM omop.specimen WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
	    "Maximum-value Columns": "load_dt",
	    "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
	    "DBCP Connection Pool": "Databricks Connection Pool"
	  },
	  "relationships": {
	    "success": ["12"]
	  }
	},
	{
	  "id": "12",
	  "type": "org.apache.nifi.processors.standard.PutFile",
	  "name": "Save Specimen Data",
	  "config": {
	    "Directory": "/landing_zone/specimen/",
	    "Conflict Resolution Strategy": "ignore"
	  },
	  "relationships": {}
	},
	{
	  "id": "13",
	  "type": "org.apache.nifi.processors.standard.QueryDatabaseTable
	  "name": "Extract Drug_exposure Data",
	  "config": {
	    "Scheduling Strategy": "timer-driven",
	    "Scheduling Period": "1 day",
	    "Query": "SELECT * FROM omop.drug_exposure WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
	    "Maximum-value Columns": "load_dt",
	    "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
	    "DBCP Connection Pool": "Databricks Connection Pool"
	  },
	  "relationships": {
	    "success": ["14"]
	  }
	},
	{
	  "id": "14",
	  "type": "org.apache.nifi.processors.standard.PutFile",
	  "name": "Save Drug_exposure Data",
	  "config": {
	    "Directory": "/landing_zone/drug_exposure/",
	    "Conflict Resolution Strategy": "ignore"
	  },
	  "relationships": {}
	},
	{
	  "id": "15",
	  "type": "org.apache.nifi.processors.standard.QueryDatabaseTable",
	  "name": "Extract Condition_occurrence Data",
	  "config": {
	    "Scheduling Strategy": "timer-driven",
	    "Scheduling Period": "1 day",
	    "Query": "SELECT * FROM omop.condition_occurrence WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
	    "Maximum-value Columns": "load_dt",
	    "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
	    "DBCP Connection Pool": "Databricks Connection Pool"
	  },
	  "relationships": {
	    "success": ["16"]
	  }
	},
	{
	  "id": "16",
	  "type": "org.apache.nifi.processors.standard.PutFile",
	  "name": "Save Condition_occurrence Data",
	  "config": {
	    "Directory": "/landing_zone/condition_occurrence/",
	    "Conflict Resolution Strategy": "ignore"
	  },
	  "relationships": {}
	},
	{
	  "id": "17",
	  "type": "org.apache.nifi.processors.standard.QueryDatabaseTable",
	  "name": "Extract Measurement Data",
	  "config": {
	    "Scheduling Strategy": "timer-driven",
	    "Scheduling Period": "1 day",
	    "Query": "SELECT * FROM omop.measurement WHERE person_id IN (SELECT person_id FROM approved_cohort_1)",
	    "Maximum-value Columns": "load_dt",
	    "Maximum-value Time": "${now():toNumber():minus(86400000):format('yyyy-MM-dd HH:mm:ss')}",
	    "DBCP Connection Pool": "Databricks Connection Pool"
	  },
	  "relationships": {
	    "success": ["18"]
	  }
	},
	{
	  "id": "18",
	  "type": "org.apache.nifi.processors.standard.PutFile",
	  "name": "Save Measurement Data",
	  "config": {
	    "Directory": "/landing_zone/measurement/",
	    "Conflict Resolution Strategy": "ignore"
	  },
	  "relationships": {}
	}
````

- Note: this example assumes you have configured a DBCP connection pool in NiFi to connect to the Databricks environment, and that you have created a directory in the landing zone for each OMOP table.

### Azura Data Factory example

1. Create a new pipeline in Azure Data Factory.
2. Add a Query activity to the pipeline for each OMOP table you want to extract.
3. Configure the Query activities to use the appropriate SQL query to extract data from the corresponding OMOP table. Here are example queries for each table:
- death: 
````
SELECT * 
FROM omop.death 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

- person: 
````
SELECT * 
FROM omop.person 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

- observation: 
````
SELECT * 
FROM omop.observation 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

- procedure_occurrence: 
````
SELECT * 
FROM omop.procedure_occurrence 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

- visit_occurrence: 
````
SELECT * 
FROM omop.visit_occurrence 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

- specimen: 
````
SELECT * 
FROM omop.specimen 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

- drug_exposure: 
````
SELECT * 
FROM omop.drug_exposure 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

- condition_occurrence: 
````
SELECT * 
FROM omop.condition_occurrence 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

- measurement: 
````
SELECT * 
FROM omop.measurement 
WHERE person_id IN (SELECT person_id FROM approved_cohort_1) 
	AND load_dt BETWEEN @{variables('load_index')} AND GETDATE();
````

4. Configure the Query activities to use a JDBC connection to connect to the Databricks environment. This can be done by creating a linked service for the Databricks environment in Azure Data Factory.
5. Add a Copy activity to the pipeline after each Query activity to move the extracted data to a landing zone within your Azure VPC. Configure the Copy activity to use an Azure Blob Storage dataset or an Azure Data Lake Storage Gen2 dataset to write the extracted data to the target directory within your VPC.
6. Save and publish the pipeline.
7. Create a trigger to run the pipeline on a regular schedule, such as once a week on Sunday.

Note that you will need to modify the SQL queries and configuration settings to fit your specific environment and requirements. Additionally, you will need to implement the cohort selection process separately as described earlier in this conversation.

## NiFi or Data Factory?

- Complexity of the ETL workflow: If the ETL workflow is complex and requires custom scripting or transformation, NiFi might be a better choice due to its flexibility and ease of use in developing custom scripts.

- Integration with other Azure services: If you are already using other Azure services such as Azure Synapse Analytics or Azure Machine Learning, Azure Data Factory may be a better choice due to its seamless integration with these services.

- Cost: Both NiFi and Data Factory have different pricing models, and the cost can vary depending on the data volume and the specific use case. Therefore, the cost should also be considered when choosing between the two tools.

- Skillset of the team: The choice between NiFi and Data Factory can also depend on the skillset of the team responsible for managing the ETL workflow. If the team has experience with one tool over the other, it may be better to stick with that tool to minimize training and learning curve.

### Can NiFi handle batch processing?

NiFi has no problem ingesting files of any type or size provided sufficient space exists in the content repository to store that data.

For performance, Nifi only passes a FlowFile references between processors within the NiFi dataflow. Even if you "clone" a large file down two or more dataflow path, this only results in an additional reference FlowFile to the same content in the content repository. All rFlowFile references to the same content must be resolved before the actual content is removed from the repository.

That being said, NiFi provides a multitude of processors for manipulating the content of FlowFiles. Anytime you modify/change the content of a FlowFile, a new FlowFile is created along with the new content. This is important because following this new content creation, you still have the original as well as your new version of the content in your content repository. So you must plan accordingly if manipulation of the content is to be done to make sure you have sufficient repository storage.

JVM memory comes in to the mix most noticeably when doing any splitting of large content in to many smaller content. So if you plan on producing more then say 10,000 individual FlowFiles for a single Large FlowFile, you will likely need additional JVM memory allocation to your NiFi.

As you can see a lot more needs to be considered beyond just the size of teh data being ingested when planning out your NiFi needs.