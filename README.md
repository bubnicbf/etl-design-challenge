# ETL Design Challenge

I am pleased to submit to you the design challenge solution for your ETL process that meets the requirements and constraints set forth by your data collaborators. My goal is to move data from their Databricks environment on Azure, apply custom transformations, validate and de-identify the data, and load it into your target data store and analytics platform. To achieve this, I have chosen to use NiFi, along with custom Python scripts, to perform the necessary data movement, validation, and de-identification tasks. I also discuss the use of Azure Data Factory in place of NiFi in several key areas. My design takes into account the need to ensure compliance with applicable regulations and policies, as well as the need to conduct regular reviews of the de-identified data. I believe my solution meets the requirements of your data collaborators while also providing a scalable and efficient process for managing large volumes of data.

<!-- ![pipeline image](img/pipeline.jpg) -->

1. [Data Sources](https://github.com/bubnicbf/etl-design-challenge/blob/main/integration_pipeline/01_data_sources.md)  
2. [Transformations](https://github.com/bubnicbf/etl-design-challenge/blob/main/integration_pipeline/02_transformations.md)  
3. [Load and Validation](https://github.com/bubnicbf/etl-design-challenge/blob/main/integration_pipeline/03_load_and_validation.md)  
4. [Transformation](https://github.com/bubnicbf/etl-design-challenge/blob/main/integration_pipeline/04_transformation.md)  
