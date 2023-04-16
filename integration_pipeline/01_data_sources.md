# Data Sources

## Discovery

The data is stored in a Databricks environment on Azure. Access to the Databricks environment can be established using a virtual private network (VPN) or Azure ExpressRoute to ensure that data remains within your Azure VPC.

1. Access the Databricks environment: First, you need to access the Databricks environment on Azure. This can be done by logging into the Azure portal, navigating to the Databricks resource, and opening the Databricks workspace.

2. Identify the OMOP v5.3 schema: Once you're in the Databricks workspace, you need to identify the OMOP v5.3 schema where the data is stored. This can be done by navigating to the SQL tab and running a query to list the available schemas. For example, you can run the following SQL query to list all the schemas:

````
SHOW SCHEMAS;
````
Look for the schema named "omop" to identify the OMOP v5.3 schema.

3. Identify the tables: Once you've identified the OMOP v5.3 schema, you need to identify the tables where the data is stored. This can be done by running a query to list the tables in the schema. For example, you can run the following SQL query to list all the tables in the "omop" schema:

````
SHOW TABLES IN omop;
````
This will list all the tables in the OMOP v5.3 schema, including the 9 clinical data tables listed in your requirements.

4. Understand the data model: Finally, you need to understand the data model used in the OMOP v5.3 schema to be able to extract and transform the data. The OMOP v5.3 data model is based on a set of clinical data tables that capture information about patients, their diagnoses, procedures, medications, and other clinical events. Each table has a set of columns that capture specific attributes of the event, such as the patient ID, the event date, the diagnosis code, and the medication name.