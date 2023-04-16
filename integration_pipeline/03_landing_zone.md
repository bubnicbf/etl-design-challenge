# Landing Zone

## Data Validation: Extraction

1. Validation criteria
- Record count: Verify that the number of records extracted matches the expected count based on the source data and selection criteria.
	- Use the QueryRecord processor to run a custom SQL query to count the number of records extracted. For example: `SELECT COUNT(*) FROM landing_zone.person;`
- Data types: Check that the data types of specific fields match the expected types, such as dates, numeric values, or character strings.
	- Use the ValidateRecord processor to validate the data types of specific fields. For example, you can use a CSVReader controller service to read a CSV file and validate that the "age" field contains numeric values: `${field.value:toNumber():equals(field.value)}`
- Data formats: Ensure that the data conforms to the expected format, such as the date format or the format of social security numbers or phone numbers.
	- Use the ExecuteScript processor to run a custom script to validate the data formats. For example, you can use a Groovy script to validate that the date format of the "birthdate" field in the "person" table is yyyy-MM-dd: `def birthdate = record.get('/birthdate').getValue() assert birthdate.matches('\\d{4}-\\d{2}-\\d{2}')`
- Required fields: Check that all required fields are present and contain valid values. For example, you may need to verify that all patient identifiers, such as patient IDs or medical record numbers, are present and accurate.
	- Use the ValidateRecord processor to validate that all required fields are present. For example, you can use a CSVReader controller service to read a CSV file and validate that the "patient_id" field is present: `${field.patient_id:notEmpty()}`
- Business rules: Validate the data against external sources or business rules, such as checking the validity of medical codes, verifying the consistency of data across different sources, or ensuring that the data meets specific regulatory or compliance requirements.
	- Use the QueryRecord processor to run a custom SQL query to validate the data against external sources or business rules. For example, you can use an external reference dataset to validate the ICD-10 codes in the "condition_occurrence" table: `SELECT COUNT(*) FROM landing_zone.condition_occurrence c JOIN reference.icd10_codes r ON c.condition_code = r.icd10_code;`
- Data completeness: Verify that all relevant data has been extracted and that there are no missing or incomplete data elements.
	- Use the ExecuteScript processor to run a custom script to check for missing or incomplete data elements. For example, you can use a Python script to validate that the "gender" field in the "person" table is not missing: `gender = record.get('/gender').getValue() assert gender is not None`
- Data consistency: Check for data consistency across different data sources, such as ensuring that patient data is consistent across all tables and sources.
	- Use the QueryRecord processor to run a custom SQL query to check for data consistency across different sources. For example, you can join the "person" and "visit_occurrence" tables to validate that the "person_id" field is consistent across both tables: `SELECT COUNT(*) FROM landing_zone.person p JOIN landing_zone.visit_occurrence v ON p.person_id = v.person_id WHERE p.gender = 'F';`
- Data accuracy: Validate the accuracy of the data against external sources or known benchmarks, such as comparing the data to a known reference dataset or validating the data against clinical guidelines or standards.
	- Use the ExecuteScript processor to run a custom script to validate the accuracy of the data against external sources or benchmarks. For example, you can use an external reference dataset to validate the "weight" field in the "measurement" table: `weight = record.get('/weight').getValue() assert weight > 0 and weight < 1000`

## Validation Processors

After configuring the validation processors, you can run the validation process to validate the extracted data in the landing zone. Depending on the size of the data and complexity of the validation criteria, this process may take some time to complete.

### Validation Process

1. Ensure that the data has been extracted from the Databricks environment and landed in the landing zone within your Azure VPC.
2. Start the NiFi dataflow that contains the validation processors.
3. Monitor the dataflow to ensure that the validation processors are processing the flow files and executing the validation logic. You can use NiFi's data provenance feature to track the progress of the flow files as they move through the dataflow and identify any validation errors or failures.
4. Once the validation process has completed, review the validation results and take any necessary action to address any validation errors or failures. Depending on the severity of the issues, you may need to rerun the extraction and validation processes to ensure that the data is accurate and complete.

### Configuration

1. QueryRecord Processor:
Configure the processor to use the JDBC connection pool to connect to the landing zone database.
- Set the SQL query to count the number of records extracted or to validate the data against external sources or business rules.
- Set the output format to be a RecordSet.
	- Database Connection Pooling Service: landing_zone_pool
	- SQL Query: `SELECT COUNT(*) FROM landing_zone.person;`
	- Output Format: RecordSet

2. ValidateRecord Processor:
Configure the processor to validate the data types, formats, and required fields using custom validation rules.
- Use one or more RecordPath properties to specify the fields to validate.
- Use the Failure Condition property to determine what happens when a validation error occurs (e.g., stop the pipeline, route to an error flow, etc.).
	- Record Reader: CSVReader
	- Record Writer: CSVRecordSetWriter
	- Failure Condition: Fail if any validation errors
	- Validation Rules:
		- /patient_id: `${field.value:notEmpty()}`
		- /birthdate: `${field.value:matches('\\d{4}-\\d{2}-\\d{2}')}`

3. ExecuteScript Processor:
Configure the processor to use the appropriate scripting language for your validation logic (e.g., Groovy, Python, JavaScript).
- Use the Script Body property to specify the validation script.
- Use the Failure Strategy property to determine what happens when a validation error occurs (e.g., stop the pipeline, route to an error flow, etc.).
	- Script Engine: Python
	- Failure Strategy: Fail on script evaluation error
	- Script Body:
````
import json

# Define the validation function
def validate_person(record):
    gender = record['/gender']
    if gender not in ['M', 'F']:
        return False
    else:
        return True

# Get the incoming flow file
flowFile = session.get()
if flowFile is not None:
    try:
        # Parse the flow file as JSON
        record = json.loads(flowFile.read().decode('utf-8'))
        
        # Validate the person record
        if not validate_person(record):
            # If the validation fails, transfer the flow file to the 'validation_failure' relationship
            session.transfer(flowFile, REL_VALIDATION_FAILURE)
        else:
            # If the validation passes, transfer the flow file to the 'validated' relationship
            session.transfer(flowFile, REL_VALIDATED)
        
        # Commit the flow file
        session.commit()
    except Exception as e:
        # If there is an error, transfer the flow file to the 'failure' relationship
        log.error(e)
        session.transfer(flowFile, REL_FAILURE)
        session.commit()
````

4. LogAttribute Processor:
Configure the processor to log the validation errors to a log file or other destination.
- Use one or more RecordPath properties to specify the fields to log.
	- Log Level: info
	- Log Filename: /logs/validation_errors.log
	- Log Format: `${now():format('yyyy-MM-dd HH:mm:ss.SSS')},${record.path},${field.path},${field.value},${field.reason}`

5. Notify Processor:
Configure the processor to send notifications to the appropriate stakeholders when a validation error occurs.
- Use the appropriate notification method (e.g., email, Slack, SMS, etc.).
- Use the Notification Text property to specify the message to send.
	- Notification Type: Email
	- Mail Hostname: mail.example.com
	- Mail Port: 25
	- Mail Subject: Data Validation Error
	- Mail Body: Validation error on `${now():format('yyyy-MM-dd HH:mm:ss.SSS')}:\n${record.path},${field.path},${field.value},${field.reason}`

6. PutFile Processor:
Configure the processor to store the validated data in a final destination.
- Use the appropriate file format (e.g., CSV, JSON, Parquet, etc.).
	- Directory: /data/validated
	- Filename: person_validated.csv
	- File Type: CSV
	- Compress: false
	- Script Body:
````
# Email alerts

import smtplib
from email.mime.text import MIMEText

# Set up email parameters
sender = 'youremail@gmail.com'
recipient = 'recipientemail@gmail.com'
subject = 'Validation Failure Alert'
body = 'Validation failure occurred in NiFi dataflow.'

# Create email message
msg = MIMEText(body)
msg['Subject'] = subject
msg['From'] = sender
msg['To'] = recipient

# Send email
smtp_server = smtplib.SMTP('smtp.gmail.com', 587)
smtp_server.starttls()
smtp_server.login(sender, 'yourpassword')
smtp_server.sendmail(sender, recipient, msg.as_string())
smtp_server.quit()
````

````
# Slack alerts
from slack_sdk import WebClient
from slack_sdk.errors import SlackApiError

# Set up Slack parameters
token = 'your_slack_bot_token'
channel = '#your_slack_channel'
text = 'Validation failure occurred in NiFi dataflow.'

# Create Slack client
client = WebClient(token=token)

# Send Slack message
try:
    response = client.chat_postMessage(channel=channel, text=text)
    print(f"Slack message sent: {response['ts']}")
except SlackApiError as e:
    print(f"Error sending Slack message: {e}")
````

