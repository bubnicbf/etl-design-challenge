# Transformation

## ETL using NiFi

There are several circumstances where using NiFi for data transformation may be more appropriate than using Azure Data Factory.

- Complex data transformations: NiFi has a rich set of built-in processors and allows for custom script execution, making it more flexible for complex data transformations. It also has support for multiple data formats, including binary data, which may be necessary for certain types of data transformations.

- Customization: NiFi is highly customizable, and users can create custom processors and plugins to extend its capabilities. This makes it a better choice for organizations that have unique data transformation requirements that cannot be met by off-the-shelf tools.

- Cost: NiFi is an open-source tool, which means that there are no licensing fees associated with its use. This can be a significant cost savings for organizations that are working with large volumes of data and need to scale their data transformation capabilities.

- Development agility: NiFi has a visual interface for designing dataflows, which allows for rapid development and iteration of data transformation pipelines. This makes it a good choice for organizations that need to rapidly iterate on their data transformation pipelines and test new ideas.

1. Use Apache NiFi to create a dataflow and configure a source processor to read the validated data from the landing zone in your Azure VPC. This can be done using a variety of supported processors, such as the GetAzureBlobStorage processor or the GetAzureDataLakeStorageV2 processor.

GetAzureDataLakeStorageV2 processor to read validated data from Azure Data Lake Storage:

````
<?xml version="1.0" encoding="UTF-8"?>
<template encoding-version="1.0">
    <description>Example NiFi flow for reading validated data from Azure Data Lake Storage</description>
    <snippet>
        <processors>
            <processor>
                <id>12345678-1234-1234-1234-123456789012</id>
                <class>org.apache.nifi.azure.datalake.storage.v2.GetAzureDataLakeStorageV2</class>
                <name>Get Validated Data</name>
                <property>
                    <name>Account Name</name>
                    <value>your_account_name</value>
                    <description>The name of the Azure Storage account to use.</description>
                    <sensitive>true</sensitive>
                </property>
                <property>
                    <name>Client ID</name>
                    <value>your_client_id</value>
                    <description>The ID of the Azure AD application used to authenticate.</description>
                    <sensitive>true</sensitive>
                </property>
                <property>
                    <name>Client Secret</name>
                    <value>your_client_secret</value>
                    <description>The client secret associated with the Azure AD application.</description>
                    <sensitive>true</sensitive>
                </property>
                <property>
                    <name>Tenant ID</name>
                    <value>your_tenant_id</value>
                    <description>The ID of the Azure AD tenant associated with the Azure Storage account.</description>
                    <sensitive>true</sensitive>
                </property>
                <property>
                    <name>Directory</name>
                    <value>/validated_data</value>
                    <description>The path to the directory in your Azure Data Lake Storage where the validated data files are located.</description>
                </property>
                <property>
                    <name>File Filter</name>
                    <value>*.csv</value>
                    <description>The filename pattern of the validated data files you want to read.</description>
                </property>
            </processor>
        </processors>
        <connections>
            <connection>
                <source>12345678-1234-1234-1234-123456789012</source>
                <destination>45678901-2345-6789-0123-456789012345</destination>
                <name>Get Validated Data to Next Processor</name>
            </connection>
        </connections>
    </snippet>
</template>
````

2. Use the data transformation capabilities built into Apache NiFi to transform the validated data as necessary to meet the requirements of your target data store or analytics platform. This may include additional data transformations, de-identification, or data enrichment. You can use a variety of transformation processors in Apache NiFi, such as the ExecuteScript processor, the QueryRecord processor, or the ConvertRecord processor, depending on your specific transformation requirements and preferred technology stack.

- Create a dataflow in Apache NiFi: Use the NiFi UI to create a dataflow and add processors to perform the necessary data transformations. The dataflow should begin with a source processor that reads the validated data from the landing zone in your Azure VPC.

- Use transformation processors: Apache NiFi offers a variety of built-in processors for data transformation, including the ExecuteScript processor, QueryRecord processor, and ConvertRecord processor. Use the appropriate processor(s) to perform the necessary data transformations, depending on your specific requirements.

- Configure transformation processors: Each processor should be configured with the appropriate properties, such as SQL queries, scripts, or mapping rules, to perform the necessary transformations.

Apache NiFi can be used to transform the data at this step by using a variety of built-in processors that allow for data transformation (ExecuteScript, ConvertRecord, JoltTransformJSON, etc.).  Additionally custom code can be implemented for any desired complex transformations.

````
import json
import requests
from nipyapi import canvas, nifi

# Get the NiFi processor group
pg = nifi.ProcessGroupApi().get_process_group(pg_id, 'id')
pg_name = pg.component.name

# Define the data transformation function
def transform_data(data):
    transformed_data = []
    for record in data:
        # Apply transformation logic
        transformed_record = {}
        transformed_record['field1'] = record['field1'] + '_transformed'
        transformed_record['field2'] = record['field2'] * 2
        transformed_data.append(transformed_record)
    return transformed_data

# Define the NiFi processors for the data transformation
get_data_processor = canvas.get_processor(processor_id=processor_id)
transform_data_processor = nifi.ProcessorBuilder('Transform Data').property('Script Engine', 'python').property('Script Body', '''
    import json
    
    def transform_data(data):
        transformed_data = []
        for record in data:
            # Apply transformation logic
            transformed_record = {}
            transformed_record['field1'] = record['field1'] + '_transformed'
            transformed_record['field2'] = record['field2'] * 2
            transformed_data.append(transformed_record)
        return transformed_data
    
    output_data = transform_data(json.loads(flowfile.getAttribute('data')))
    flowfile = session.putAttribute(flowfile, 'data', json.dumps(output_data))
    session.transfer(flowfile, REL_SUCCESS)
''').add_relationship('success', 'Transform Data', 'success')

# Connect the processors
canvas.create_connection(get_data_processor, transform_data_processor, 'success')

# Start the NiFi dataflow
canvas.schedule_process_group(pg, true)

````

3. Create a sink processor to load the transformed data into your target data store or analytics platform. This can be done using a variety of supported sink processors, such as the PutAzureSynapseAnalytics processor, the PutAzureSqlDatabase processor, or the PutAzureBlobStorage processor.

- Configure the sink processor to load the transformed data into the target data store or analytics platform using the appropriate loading mechanism, such as inserting data into a table or writing data to a file, depending on the specific connector and target data store being used.

    - Storage Account Name/Key
    - Container Name for each Cohort
    - Blob Name for each patient
    - Use Chunking if using batch processing
    - Chunk Size for batch size
    - Max Concurrent Request for optimization

- Configure the appropriate data loading settings in the sink processor, such as the column mapping, file format, and batch size. You can also configure performance optimization settings, such as parallelism and partitioning, to optimize the data loading process.

````
{
  "revision": {
    "version": 0
  },
  "id": "example-processor",
  "parentGroupId": "example-group",
  "position": {
    "x": 100,
    "y": 100
  },
  "component": {
    "type": "PutAzureBlobStorage",
    "bundle": {
      "group": "org.apache.nifi",
      "artifact": "nifi-azure-bundle",
      "version": "1.13.2"
    },
    "name": "PutAzureBlobStorage",
    "properties": {
      "Blob Storage Name": "example-blob-storage",
      "Account Name": "example-account-name",
      "Account Key": "example-account-key",
      "Container Name": "example-container-name",
      "Directory": "/example/directory",
      "Compress": "true",
      "Permissions": "0777"
    },
    "relationships": [
      {
        "name": "success",
        "autoTerminate": false
      },
      {
        "name": "failure",
        "autoTerminate": true
      }
    ],
    "supportsBatching": true
  }
}

````

4. Configure a connection between the source processor and the sink processor in your Apache NiFi dataflow to move the transformed data from the source to the sink. The connection will handle the data loading mechanism, such as inserting data into a table or writing data to a file, depending on the specific processor and target data store being used.

5. Configure the appropriate data loading settings in the sink processor, such as the column mapping, file format, and batch size. You can also configure performance optimization settings, such as parallelism and threading, to optimize the data loading process.

- Configure the settings according to the requirements of your target data store or analytics platform. This may involve mapping the columns in the source data to the columns in the target data store, specifying the file format and compression settings, and setting the batch size to optimize performance.

````
{
  "revision": {
    "version": 0
  },
  "component": {
    "name": "PutAzureBlobStorage",
    "type": "org.apache.nifi.azure.storage.PutAzureBlobStorage",
    "properties": {
      "Blob Storage Connection": "your_connection",
      "Blob Storage Container": "your_container",
      "File Name": "your_file_name",
      "Content Type": "text/plain",
      "Storage Strategy": "COMPOSED",
      "Compression Format": "NONE",
      "Conflict Resolution Strategy": "REPLACE_ON_NO_CONFLICT",
      "Overwrite Existing": "true",
      "Block Size": "512 MB",
      "Max Wait Time": "1 min",
      "Minimum File Age": "0 sec",
      "Proxy Configuration Service": "",
      "Put Mode": "STREAMING"
    },
    "relationships": {
      "success": {
        "name": "success",
        "description": ""
      },
      "failure": {
        "name": "failure",
        "description": ""
      }
    }
  }
}

````

6. Start the Apache NiFi dataflow to load the transformed data into your target data store or analytics platform.

### Data Factory ETL example

There are several circumstances in which you might want to use Azure Data Factory instead of NiFi for transformation:

- Integration with Azure Services: If you are using a lot of Azure services, Azure Data Factory might be a better fit because it has tight integration with many Azure services, making it easier to use them in your data pipeline.

- Familiarity with SQL: If you are more comfortable working with SQL for your transformations, Azure Data Factory has built-in support for SQL transformations, which may make it a more natural choice for you.

- Data Volume: If you are dealing with a large volume of data, Azure Data Factory can handle the scale-out process better than NiFi, which is more suitable for smaller data volumes.

- Budget: If cost is a significant concern, Azure Data Factory might be more cost-effective because it uses a pay-as-you-go pricing model, which allows you to scale up and down as needed.

- Simplicity: If your ETL process is relatively straightforward, and you do not require a lot of customization, Azure Data Factory might be a simpler and more straightforward option to use.

1. Create an Azure Data Factory pipeline and configure a source dataset to read the validated data from the landing zone in your Azure VPC. This can be done using a variety of supported data source connectors, such as the Azure Blob Storage connector or the Azure Data Lake Storage Gen1 connector.
2. Use the data transformation capabilities built into Azure Data Factory to transform the validated data as necessary to meet the requirements of your target data store or analytics platform. This may include additional data transformations, de-identification, or data enrichment. You can use a variety of transformation activities in Azure Data Factory, such as the Data Flow activity, the HDInsight Hive activity, or the Databricks Notebook activity, depending on your specific transformation requirements and preferred technology stack.
3. Create a sink dataset to load the transformed data into your target data store or analytics platform. This can be done using a variety of supported data sink connectors, such as the Azure Synapse Analytics connector, the Azure SQL Database connector, or the Azure Blob Storage connector.
4. Configure a copy activity in your Azure Data Factory pipeline to move the transformed data from the source dataset to the sink dataset. The copy activity will handle the data loading mechanism, such as inserting data into a table or writing data to a file, depending on the specific connector and target data store being used.
5. Configure the appropriate data loading settings in the copy activity, such as the column mapping, file format, and batch size. You can also configure performance optimization settings, such as parallelism and partitioning, to optimize the data loading process.
6. Start the Azure Data Factory pipeline to load the transformed data into your target data store or analytics platform.

## De-identification

1. Identify the fields that contain personally identifiable information (PHI) and are subject to de-identification. These may include fields such as names, addresses, dates of birth, social security numbers, and medical record numbers.

- death: 
	- date of death
	- cause of death
- person: 
	- name
	- date of birth
	- gender
	- race
	- ethnicity
	- address
	- social security number
- observation: 
	- test results
	- measurements
		- height
		- weight
		- blood pressure
	- dates of service
- procedure_occurrence: procedure codes
	- dates of service
- visit_occurrence: dates of service
	- type of visit
		- inpatient
		- outpatient
	- provider information
- specimen: specimen ID
	- dates of service
	- provider information
- drug_exposure: drug codes
	- dose
	- dates of service
	- provider information
- condition_occurrence: diagnosis codes
	- dates of service
	- provider information
- measurement: measurements
		- height
		- weight
		- blood pressure
	- dates of service

Note that the specific fields that contain PII may vary depending on the specific data sources used and the regulations and policies that apply to your project. It's important to carefully review the data fields in each table to identify any fields that may need to be de-identified to ensure compliance with applicable regulations and policies.

2. Determine the de-identification method(s) that will be used. There are two primary methods of de-identification: 1) removing or masking the PHI, and 2) encrypting the PHI. Depending on the specific requirements and constraints of your project, you may need to use one or both methods.

Encryption example:
````
from cryptography.fernet import Fernet

def encrypt_pii_data(pii_data):
    # Generate a new encryption key
    key = Fernet.generate_key()

    # Create a Fernet object using the key
    f = Fernet(key)

    # Convert the PII data to bytes
    bytes_pii_data = bytes(pii_data, 'utf-8')

    # Encrypt the PII data using the Fernet object
    encrypted_pii_data = f.encrypt(bytes_pii_data)

    # Return the encrypted PII data and encryption key as a tuple
    return encrypted_pii_data, key

````

3. Implement the de-identification logic using Azure Data Factory. This may involve writing custom scripts or using built-in processors to perform the necessary transformations.

- Create an Azure Data Factory pipeline to ingest the source data into a landing zone within your Azure VPC.
- Use an Azure Function to call the de-identification script that you have written in Python or another language. This script should read in the source data, identify any fields containing PII data, and perform the necessary transformations to de-identify the data.
- Use a Mapping Data Flow activity in Azure Data Factory to transform the data after it has been de-identified. You can use the Mapping Data Flow to perform operations such as filtering, aggregating, joining, and pivoting the data to prepare it for loading into your target data store.
- Use an Azure Data Lake Storage Sink to load the transformed and de-identified data into a target data store such as Azure Synapse Analytics or another analytics platform.

````
import os
import json
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:
    # Read in the request body as a JSON object
    req_body = req.get_json()

    # Get the path to the source data file from the request body
    source_data_path = req_body.get('source_data_path')

    # Call the de-identification script using the path to the source data file
    os.system('python deidentification_script.py ' + source_data_path)

    # Return a response indicating that the script was run successfully
    return func.HttpResponse("De-identification script executed successfully")
````

4. Test the de-identification process to ensure that it is working correctly and has not introduced any errors or data inconsistencies.

- Data consistency: Check the de-identified data against the original data to ensure that all identifiable information has been removed or encrypted.
Test for completeness: Ensure that all required fields have been de-identified and that no fields have been accidentally left unmodified.
- Data quality: Check that the de-identified data is still usable and meaningful for the intended purpose. This may involve running some analytics or data mining algorithms on the de-identified data.
- Performance: Check that the de-identification process can handle large volumes of data and does not introduce significant processing delays.

Basic data check:
````
# Import necessary libraries and modules
import pandas as pd
import hashlib

# Define a function to de-identify PHI data using SHA-256 hashing algorithm
def encrypt_phi(data):
    hashed_data = hashlib.sha256(str.encode(data)).hexdigest()
    return hashed_data

# Load data into a pandas DataFrame
df = pd.read_csv('deidentified_data.csv')

# Iterate over each row in the DataFrame and apply the de-identification function to the PHI data
for index, row in df.iterrows():
    row['name'] = encrypt_phi(row['name'])
    row['address'] = encrypt_phi(row['address'])
    row['phone'] = encrypt_phi(row['phone'])
    row['email'] = encrypt_phi(row['email'])

# Print the first 5 rows of the de-identified DataFrame to verify that the PHI data has been encrypted
print(df.head())
````

5. Monitor the de-identified data to ensure that it remains compliant with applicable regulations and policies. This may involve periodically reviewing the de-identified data to ensure that it still contains no PHI or other sensitive information.

- Set up automated monitoring processes: Use tools such as Azure Monitor to set up automated alerts and notifications to flag any potential re-identification risks or breaches of privacy.

- Conduct regular reviews of the de-identified data: Schedule regular reviews of the de-identified data to check for any potential re-identification risks or breaches of privacy. These reviews should be conducted by authorized personnel who have the appropriate expertise and training to identify potential risks.

Automated report:
````
import pandas as pd

# Load data into a Pandas DataFrame
df = pd.read_csv('deidentified_data.csv')

# Calculate the number of records processed and reviewed
num_records_processed = len(df)
num_records_reviewed = len(df[df['reviewed'] == True])

# Create a report DataFrame
report_df = pd.DataFrame({'Metric': ['Number of Records Processed', 'Number of Records Reviewed'],
                          'Value': [num_records_processed, num_records_reviewed]})

# Print the report
print(report_df.to_string(index=False))

````

- Ensure compliance with applicable regulations and policies: Ensure that your de-identification process is compliant with all applicable regulations and policies, such as HIPAA, GDPR, and other relevant data privacy laws.

- Maintain documentation of the de-identification process: Maintain detailed documentation of the de-identification process, including all tools and methods used, to ensure that the process can be audited and validated as needed.

1. Introduction
Overview of the de-identification process
	- Purpose of the document
	- Scope and limitations of the de-identification process
2. Regulatory Framework
	- Overview of the regulatory framework governing the de-identification process
	- Explanation of applicable regulations and policies
3. De-identification Methodology
	- Description of the de-identification methodology
	- Explanation of the data fields that contain identifiable information in each OMOP table
	- Overview of the de-identification techniques used to protect PHI
4. Implementation
	- Overview of the tools and technologies used to implement the de-identification process
	- Explanation of how the de-identification logic was implemented using Azure Data Factory
	- Description of any custom scripts used in the de-identification process
5. Testing and Validation
Explanation of the testing and validation process used to ensure the de-identification process is working correctly and has not introduced any errors or data inconsistencies
	- Overview of the test cases and test data used in the testing process
6. Monitoring and Maintenance
	- Description of the monitoring and maintenance process used to ensure that the de-identified data remains compliant with applicable regulations and policies
	- Explanation of how the automated report is generated and reviewed by authorized personnel
7. Conclusion
	- Summary of the de-identification process
	- Conclusion on the effectiveness of the de-identification process
	- Future recommendations for improvements or enhancements to the de-identification process.

- Update the de-identification process as needed: As regulations and policies change, update your de-identification process as needed to ensure ongoing compliance and data privacy.

6. Document the de-identification process and make it available to relevant stakeholders, including data collaborators and regulatory bodies. This documentation should include a detailed description of the de-identification methods used, as well as any assumptions or limitations of the process.

## Data Load: Azure Synapse Analytics

1. Create a Blob Storage dataset that points to the validated data files in the landing zone.
2. Create an Azure Synapse Analytics dataset that points to the target table in Azure Synapse Analytics.
3. Create a Data Flow activity in your Azure Data Factory pipeline that reads the validated data from the Blob Storage dataset, transforms it as necessary using the Data Flow transformation capabilities, and writes it to the Azure Synapse Analytics dataset.
4. Configure the appropriate transformation settings in the Data Flow activity, such as column mapping and data type conversions, to meet the requirements of Azure Synapse Analytics.
5. Configure the appropriate performance optimization settings in the Data Flow activity, such as parallelism and partitioning, to optimize the data transformation process.
6. Configure a copy activity in your Azure Data Factory pipeline that moves the transformed data from the Blob Storage dataset to the Azure Synapse Analytics dataset using the appropriate SQL insert statements.
7. Configure the appropriate data loading settings in the copy activity, such as batch size and retry behavior, to optimize the data loading process.
8. Start the Azure Data Factory pipeline to load the transformed data into Azure Synapse Analytics.

````
{
    "name": "LoadValidatedDataToSynapseAnalytics",
    "properties": {
        "activities": [
            {
                "name": "TransformValidatedData",
                "type": "DataFlow",
                "dependsOn": [],
                "policy": {
                    "timeout": "7.00:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": {}
                },
                "typeProperties": {
                    "source": {
                        "type": "DelimitedTextSource",
                        "location": {
                            "type": "AzureBlobStorageLocation",
                            "container": "validated-data-container",
                            "folderPath": "validated-data-folder"
                        },
                        "delimiter": ","
                    },
                    "transformation": {
                        "name": "dataflow-name"
                    },
                    "sink": {
                        "type": "AzureSqlSink",
                        "writeBatchSize": 10000,
                        "writeBatchTimeout": "0.00:00:30",
                        "sqlWriterCleanupScript": "",
                        "sqlWriterStoredProcedureName": "storedProcedureName",
                        "storedProcedureParameters": {
                            "parameterName": "@{item().parameterName}"
                        },
                        "storedProcedureTableTypeParameterName": "storedProcedureTableTypeParameterName",
                        "allowPolyBase": false
                    }
                }
            }
        ],
        "parameters": {
            "parameterName": {
                "type": "String"
            }
        },
        "annotations": []
    }
}
````