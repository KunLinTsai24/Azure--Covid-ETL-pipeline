# Covid-ETL-pipeline

## Introduction

The COVID-19 pandemic has underscored the critical need for timely and accurate data to inform public health decisions and policy-making. Organizations require efficient data pipelines to ingest, transform, and visualize vast amounts of pandemic-related data from various sources. This project aims to automate the data preparation process for COVID-19 datasets using **Azure Data Factory (ADF)**, facilitating machine learning applications and Power BI reporting to support data-driven decisions.

---
## Business Requirements

**The primary objectives of this project are centered around the ETL process:**

- **Efficient Data Ingestion: Automate the extraction of multiple COVID-19 datasets into a centralized data lake.**
- **Data Validation: Ensure the accuracy and integrity of the ingested data through validation checks.**
- **Data Transformation: Cleanse and restructure data to meet analytical and reporting needs.**
- **Data Loading: Seamlessly load the transformed data into a SQL database for downstream applications.**
- **Automation and Monitoring: Implement robust automation with error handling, notifications, and the ability to rerun failed processes.**
---
## Process Outline

### Solution Architecture

**![](https://github.com/KunLinTsai24/Covid-ETL-pipeline/blob/main/img/covid_architecture_solution.png)**

---

### Data Sources

- **Population Data**: "Population by Age" dataset stored in Azure Blob Storage.
- **European Centre for Disease Prevention and Control (ECDC) Data**:
  - **Cases & Deaths Data**
  - **Hospital Admission Data**
  - **Testing Data**
  - **Country Response Data**
---

### Data Ingestion

**Ingesting Population Data**

**Objective**: Ingest "Population by Age" data from Blob Storage into Azure Data Lake Storage (ADLS) with validation.

**Process**:

1. **Validation Activity**:
    - **Check File Existence**: Used a **Get Metadata** activity to verify the file's presence in Blob Storage.
    - **Validate Schema**: Confirmed the file has the correct number of columns to match the expected schema.
2. **Conditional Logic**:
    - Implemented an **If Condition** activity to evaluate the validation results.
        - **If True**:
            - **Copy Activity**: Transferred the data from Blob Storage to ADLS.
            - **Delete Activity**: Removed the source file from Blob Storage to prevent duplicate processing.
        - **If False**:
            - **Web Activity**: Sent a notification to a specified URL to alert the team of the validation failure.

![](https://github.com/KunLinTsai24/Covid-ETL-pipeline/blob/main/img/Ingest%20population.png)

**Ingesting ECDC Data**

**Objective**: Dynamically ingest multiple ECDC datasets into ADLS.

**Process**:

1. **Lookup Activity**: Created a **Lookup** activity to retrieve a list of URLs and file names for all required ECDC datasets.
2. **Parameterization**: Defined parameters for **RelativeURL** and **FileName** to handle variable data sources.
3. **Variable Assignment**: Assigned the output of the Lookup activity to variables **sourceRelativeURL** and **sinkFileName**.
4. **ForEach Activity**: Used a **ForEach** loop to iterate over each dataset.
    - Within the loop, a **Copy Activity** ingested each dataset from Blob Storage into ADLS.

![](https://github.com/KunLinTsai24/Covid-ETL-pipeline/blob/main/img/Ingest%20ECDC.png)

---

### Data Transformation

Data transformation was performed using **Mapping Data Flows** in Azure Data Factory to prepare the data for analysis and reporting.

**Transforming Cases & Deaths Data**

1. **Source Selection**: Chose the "Cases & Deaths" data as the input source.
2. **Filtering**: Applied filters to include only European Union (EU) countries where the country field is not null.
3. **Column Selection and Renaming**: Selected necessary columns and renamed them for clarity and consistency.
4. **Pivot Operation**: Pivoted the **Indicator** and **Daily Count** columns to create separate columns for "Cases Count" and "Deaths".
5. **Lookup Transformation**: Performed a lookup to the **Country** dimension table to retrieve the two-digit country codes.
6. **Sink Configuration**: Specified the destination within ADLS for the transformed data.

![](https://github.com/KunLinTsai24/Covid-ETL-pipeline/blob/main/img/Case%20%26%20Deaths%20Transform.png)

![](https://github.com/KunLinTsai24/Covid-ETL-pipeline/blob/main/img/Case%20%26%20Death%20Dataflow.png)

**Transforming Hospital Admission Data**

1. **Source Selection**: Selected both daily and weekly hospital admission data files.
2. **Column Selection and Renaming**: Streamlined the datasets by selecting relevant columns and standardizing their names.
3. **Lookup Transformation**: Joined with the **Country** dimension to include country codes and population data.
4. **Conditional Split**: Split the data based on the reporting frequency indicator (daily or weekly).
5. **Aggregation and Join**: Aggregated weekly data and joined it with the date data to align reporting periods.
6. **Pivot Operation**: Pivoted indicators and counts to separate columns for daily/weekly hospital admissions and ICU admissions.
7. **Sorting**: Sorted the data by reported date and country to ensure orderly presentation.
8. **Sink Configuration**: Defined the destination paths in ADLS for the transformed datasets.

**Creating Date Dimension Data**

1. **Source Selection**: Selected the date data source.
2. **Aggregation**:
    - Grouped data by week number.
    - Calculated the minimum (week start date) and maximum (week end date) dates for each week.

![](https://github.com/KunLinTsai24/Covid-ETL-pipeline/blob/main/img/Hospital%20Admission%20Daily%20Transform.png)

![](https://github.com/KunLinTsai24/Covid-ETL-pipeline/blob/main/img/Hospital%20Admission%20Weekly%20Transform.png)

![](https://github.com/KunLinTsai24/Covid-ETL-pipeline/blob/main/img/Hospital%20Admission%20Dataflow.png)

---

### Data Loading to SQL Database

To support machine learning models and Power BI reporting, the transformed data was loaded into an Azure SQL Database.

**Process**:

1. **Table Creation**: Defined and created tables in the SQL Database with appropriate schema matching the transformed data.
2. **Copy Activity**: Used ADF's **Copy Data** activity to transfer data from ADLS to the SQL Database tables.

---

### Automation and Monitoring**

To ensure the pipeline runs smoothly and issues are promptly addressed:

- **Scheduling**: Configured pipelines to run at specified intervals to keep data up-to-date.
- **Error Handling**: Implemented validation steps and conditional logic to handle data issues gracefully.
- **Notifications**: Set up alerts and notifications via web activities to inform stakeholders of any failures or anomalies.

---

## Conclusion

By leveraging Azure Data Factory, the project successfully automated the ingestion, validation, transformation, and loading of critical COVID-19 data. This automation:

- **Improved Efficiency**: Reduced manual effort and accelerated data availability.
- **Enhanced Data Quality**: Ensured data integrity through validation and error handling.
- **Enabled Advanced Analytics**: Provided clean and structured data suitable for machine learning models.
- **Facilitated Informed Decision-Making**: Delivered insightful Power BI reports to support stakeholders in responding to the pandemic.

---
## Learning Outcome

I have mastered various data transformation activities, including filtering irrelevant data to enhance quality and selecting and renaming columns to standardize schemas. Utilizing pivot transformations, I reshaped data to simplify the analysis of key indicators like cases and deaths. I enriched datasets through lookup transformations by adding related information such as country codes and population figures. Additionally, I applied conditional splits to segregate data based on specific conditions for tailored processing, used aggregate transformations to summarize data and identify trends, organized data logically with sort transformations, and combined data from multiple sources using joins for comprehensive analyses.

In enhancing pipeline activities, I learned to retrieve metadata for validation checks using the Get Metadata activity, ensuring data integrity before processing. I implemented conditional logic with If Condition activities and automated repetitive tasks with ForEach activities, significantly improving efficiency. Configuring Copy activities allowed for data transfers between various storage services while handling various data formats. I also enhanced monitoring by sending notifications or triggering external services using Web activities. Finally, I managed storage efficiently by removing processed files with Delete activities, preventing redundancy and conserving resources.
