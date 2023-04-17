# ETL Design Challenge

I am pleased to submit to you the design challenge solution for your ETL process that meets the requirements and constraints set forth by your data collaborators. My goal is to move data from their Databricks environment on Azure, apply custom transformations, validate and de-identify the data, and load it into your target data store and analytics platform. To achieve this, I have chosen to use a combination of NiFi and Azure Data Factory, along with custom Python scripts, to perform the necessary data movement, validation, and de-identification tasks. I also discuss the differences between NiFi and Azure Data Factory in several key areas. My design takes into account the need to ensure compliance with applicable regulations and policies, as well as the need to conduct regular reviews of the de-identified data. I believe my solution meets the requirements of your data collaborators while also providing a scalable and efficient process for managing large volumes of data.

![pipeline image](https://github.com/bubnicbf/etl-design-challenge/blob/main/ingest_data_flow/ingest_data_flow.jpg)

1. [Data Sources](https://github.com/bubnicbf/etl-design-challenge/blob/main/integration_pipeline/01_data_sources.md)  
2. [Ingestion & Transport](https://github.com/bubnicbf/etl-design-challenge/blob/main/integration_pipeline/02_ingestion_transport.md)  
3. [Landing Zone](https://github.com/bubnicbf/etl-design-challenge/blob/main/integration_pipeline/03_landing_zone.md)  
4. [Transformation](https://github.com/bubnicbf/etl-design-challenge/blob/main/integration_pipeline/04_transformation.md)  


## Automation

Both NiFi and Azure Data Factory allow you to schedule the pipeline to run at specified intervals. This can be done through the built-in scheduling features or using external scheduling tools such as Airflow.

Apache Airflow can be used to automate the OMOP ETL workflow, allowing for more efficient and reliable processing of large data volumes. With Airflow, you can create a DAG (Directed Acyclic Graph) that represents the data pipeline, with each task representing a step in the ETL process.

Airflow is an open-source platform to programmatically author, schedule, and monitor workflows. It is designed to orchestrate complex workflows and data pipelines.

You can also define a DAG (Directed Acyclic Graph) that represents the workflow. The DAG can be defined using Python code and can include tasks that correspond to each step in the ETL process, including data extraction, transformation, and loading. Each task can be associated with an operator, which is a Python class that defines the behavior of the task.

Overall, Airflow provides a powerful and flexible framework for automating complex ETL workflows. It allows you to define workflows using Python code, integrate with a wide range of data sources and targets, and monitor and manage workflows using a web-based UI or a command-line interface.

## Modularization

Modularization is an important aspect of software engineering, and it applies to the design of ETL solutions as well. In a modularized ETL solution, the various components of the solution are designed and implemented as independent modules that can be easily reused in other ETL processes or even in other software applications. This approach makes the solution more flexible, scalable, and maintainable.

1. The solution presented here is modularized for reuse in several ways. First, the ETL process is divided into distinct phases, each with its own set of tools and technologies. This allows for easy substitution of different tools or technologies as needed for specific use cases.

2. The various components of each phase are designed and implemented as independent modules. For example, the NiFi dataflow for data extraction is a separate module that can be easily reused in other data extraction processes. Similarly, the Python scripts for data validation can be reused in other validation processes.

3. The solution is designed to be configurable and extensible. For example, the SQL queries used for data extraction and the Python scripts used for validation can be easily modified to fit different data sources or validation criteria.

4. The solution includes documentation and examples to facilitate reuse. The documentation provides a clear explanation of the ETL process and the purpose of each component, while the examples demonstrate how to use each component in a practical setting.

By modularizing the ETL solution for reuse, the solution becomes more flexible and adaptable to changing requirements and data sources, while also reducing the amount of redundant code and effort required for development and maintenance.

## Cost 

The cost difference between using NiFi vs Azure Data Factory vs a combination of NiFi and Azure Data Factory depends on a variety of factors, such as the size and complexity of the data, the number of data sources and sinks involved, and the specific requirements of the data processing and transformation tasks. Here are some considerations to keep in mind:

NiFi:

- NiFi is open-source and free to use, so there are no licensing costs.
- However, if you are using NiFi on a cloud provider such as Azure, you will still need to pay for the underlying compute resources and storage that NiFi is running on.
- NiFi is designed for real-time processing and low-latency data movement, which means it may be more performant than Azure Data Factory in certain use cases.
- NiFi is highly customizable and can be configured to work with a wide variety of data sources and sinks.

Azure Data Factory:

- Azure Data Factory is a cloud-based ETL service offered by Microsoft, and it has a usage-based pricing model.
- The cost of using Azure Data Factory depends on factors such as the number of activities executed, the volume of data processed, and the frequency of pipeline runs.
- Azure Data Factory has built-in connectors for a variety of data sources and sinks, which can simplify the ETL process.
- Azure Data Factory has built-in support for data movement and transformation at scale, which can make it more suitable for large-scale ETL jobs.

Combination of NiFi and Azure Data Factory:

- A combination of NiFi and Azure Data Factory can offer the benefits of both tools.
- For example, NiFi can be used for real-time data processing and low-latency data movement, while Azure Data Factory can be used for large-scale batch processing and integration with Azure services.
- However, using both tools may increase the complexity of the ETL solution and require additional expertise to manage and maintain.

A combination of NiFi and Azure Data Factory can help save costs by leveraging the strengths of each tool and reducing the need for highly skilled and expensive data engineers. NiFi provides a user-friendly interface for designing data flows and transformations, allowing for less experienced developers to easily create and maintain data pipelines. This can reduce the need for highly skilled developers, who are typically more expensive to hire and retain.

Azure Data Factory, on the other hand, provides a more powerful and scalable ETL solution with the ability to handle larger volumes of data and more complex data transformations. By utilizing Azure Data Factory for the more complex ETL tasks, and NiFi for simpler tasks, the workload can be distributed more efficiently and cost-effectively. 

Overall, the combination of NiFi and Azure Data Factory can help save costs by reducing the need for highly skilled SQL developers, leveraging the strengths of each tool for specific tasks, and improving overall efficiency in data pipeline development and maintenance.

## Speed

For ingesting a large cohort of 700,000,000+ records, a combination of NiFi and Azure Data Factory would likely be the best solution.

NiFi can handle the high-volume data ingestion and provide data flow orchestration capabilities, while Azure Data Factory can provide advanced transformation capabilities and seamlessly integrate with Azure data services. This combination can provide a scalable, flexible, and cost-effective solution for ingesting large amounts of data.

By using a combination of both tools, the data engineering team can leverage the strengths of each platform to create a more efficient and effective data ingestion and processing pipeline. Additionally, the use of NiFi and Azure Data Factory can provide a more modular and reusable solution, which can help reduce development time and costs over the long term.

## Multiple Cohorts

If a second cohort needs to be ingested in addition to the large cohort, the same solution can be used. The difference would be in the configuration of the data extraction step, where a separate query would be created for the smaller cohort and a separate connection could be established in either NiFi or Azure Data Factory for the source data.

In NiFi, a separate GetJDBC or QueryDatabaseTable processor can be added to the existing dataflow to extract data from the second cohort. In Azure Data Factory, a separate source dataset can be created to read the data from the second cohort.

Once the data is extracted, it can be processed and transformed using the same data transformation steps that were used for the first cohort. This could include additional data transformations, de-identification, or data enrichment, depending on the specific requirements for the second cohort.

The transformed data can be loaded into the target data store or analytics platform using a separate sink processor in NiFi or a separate sink dataset in Azure Data Factory. The same data loading settings can be used for the second cohort as were used for the first cohort.

By reusing the existing data transformation and data loading processes, the solution can efficiently handle the ingestion of multiple cohorts, regardless of their size.

- Speed: as the volume of data increases, the time required to process the data will also increase. This can result in longer processing times and potential delays in data availability for downstream applications. Additionally, if the processing workload is too large, it may overwhelm the processing resources, leading to performance issues or failures.

- Cost: processing large volumes of data can be expensive in terms of computing resources and storage. The cost of using cloud-based solutions such as NiFi and Azure Data Factory will depend on the amount of data processed, the duration of processing, and the amount of storage used. Therefore, processing multiple cohorts of varying sizes can result in a significant increase in cost, especially if the processing is not optimized.

To minimize the impact of multiple cohorts on speed and cost, it is important to optimize the data processing pipeline. This may involve strategies such as scaling resources up or down based on the size of the cohort being processed, or using techniques such as partitioning to distribute the workload across multiple processing resources. It is also important to continuously monitor and optimize the processing pipeline to ensure that it remains efficient and cost-effective.

## FAQ

Q: Can you walk me through the data transformation process?
A: Sure, the data transformation process involves several steps. First, the data is extracted using tools like Apache NiFi or Azure Data Factory. Next, the data is transformed using custom python scripts or tools like Apache NiFi. The transformed data is then loaded into a target data store or analytics platform using tools like Apache NiFi or Azure Data Factory. Finally, the data is validated to ensure that it is accurate and consistent.

Q: How is the solution modularized for reuse?
A: The solution is modularized by breaking it down into discrete components, each of which can be reused as needed. For example, the data extraction process can be reused for multiple cohorts of data, and the validation scripts can be reused to check for quality issues across different data sets.

Q: How does the solution handle automation?
A: The solution can be automated using tools like Apache NiFi, Azure Data Factory, and Apache Airflow. These tools can be used to schedule and orchestrate the various components of the ETL process, making it more efficient and less error-prone.

Q: How does the solution handle de-identification of sensitive data?
A: The solution includes custom python scripts to de-identify data, which are integrated into the data transformation process. This helps to ensure that sensitive data is protected and that the solution is compliant with applicable regulations and policies.

Q: Can you explain how the solution handles large cohorts of data?
A: The solution is designed to handle large volumes of data by using tools like Apache NiFi and Azure Data Factory, which are optimized for processing large amounts of data quickly and efficiently. Additionally, the solution is modularized, which makes it easier to scale and adapt to changing needs over time.

Q: How does the solution ensure data quality?
A: The solution includes a validation step, which uses custom python scripts and tools like Apache NiFi to check for data quality issues and ensure that the data is consistent and accurate. This helps to reduce errors and improve the overall quality of the data being processed.

Q: What are the benefits of using Apache NiFi for data extraction?
A: Apache NiFi is a good choice for data extraction due to its ease of use, flexibility, and ability to handle large volumes of data. It also has built-in support for a wide range of data sources, making it a versatile tool for extracting data from a variety of environments.

Q: Can you explain the benefits of using Azure Data Factory for data extraction?
A: Azure Data Factory is a good choice for data extraction because it is designed to work with a wide range of data sources and offers built-in support for many common data connectors. It is also a cloud-based service, which can help to reduce costs and improve scalability.

Q: How does the solution handle performance optimization?
A: The solution uses a variety of techniques to optimize performance, such as parallelism, partitioning, and threading. These techniques help to ensure that the ETL process runs quickly and efficiently, even when processing large volumes of data.

Q: What tools are used for data validation?
A: Custom Python scripts are used for data validation. These scripts are executed by NiFi processors, such as the ExecuteScript processor or the InvokeHTTP processor, to check for data quality issues, data consistency, and data integrity.

Q: How is the ETL pipeline monitored and maintained?
A: The ETL pipeline is monitored and maintained using various tools and techniques, such as logging and alerting, performance optimization, and periodic reviews. NiFi and Data Factory both have built-in monitoring and maintenance features, such as logging processors, flowfile repository management, and health monitoring. Additionally, custom scripts and tools can be used to monitor specific aspects of the pipeline, such as data quality, performance, and security.

Q: How does the solution handle data quality issues?
A: The solution includes validation steps at various points in the process to ensure data quality. Additionally, data engineers can develop custom scripts to perform more detailed validation on the transformed data.

Q: Can the solution handle incremental updates to the data, or only full reloads?
A: The solution can handle both incremental updates. For example, a scheduled Airflow DAG could be set up to run the ETL process on a daily basis, only pulling in new or updated data since the last run.

Q: What types of monitoring and logging are in place to detect and troubleshoot errors?
A: The solution includes logging and monitoring mechanisms to track the progress of the ETL process and detect errors. Both NiFi and Data Factory provide detailed logs and metrics, and custom scripts can be developed to perform additional monitoring and alerting.

Q: Is the solution scalable to handle future growth and increased data volume?
A: The solution is designed to be scalable and can handle increased data volume as needed. For example, additional processing nodes can be added to the Databricks cluster to handle larger data sets, and NiFi and Data Factory can be configured to handle parallel processing for increased speed and efficiency.

Q: What security measures are in place to protect sensitive data during the ETL process?
A: The solution includes various security measures, such as using Azure VPCs to isolate the data and limiting access to authorized personnel only. Additionally, the solution includes de-identification mechanisms to protect sensitive data during transformation and a validation process to ensure compliance with privacy regulations.