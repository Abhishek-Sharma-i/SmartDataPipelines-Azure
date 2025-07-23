Azure Data Factory + ADLS Gen2 Project
End-to-End Data Pipelines on Azure (SQL → ADLS | ForEach | Data Flow | Conditional Copy | Join)

I would like to express my gratitude to CSI (Celebal Summer Internship- https://celebaltech.com/) for the opportunity to work on this end-to-end Data Engineering project. This hands-on experience helped me understand the building blocks of Azure-based data pipelines, real-world ETL workflows, parameterization, conditional logic, and data transformation using ADF (Azure Data Factory) and ADLS Gen2.

🧭 Project Overview
The project was split into multiple independent tasks. Each task reflects a key concept in modern data engineering, including parameterized pipelines, ForEach control flows, conditional copying, folder partitioning, joining heterogeneous data sources, and saving optimized file formats like JSON & Parquet.

✅ Task 1: Threshold-Based Conditional Copy (SQL → ADLS)
Objective:
Create a pipeline that:

Reads a threshold value from a file in ADLS.

Checks if the number of rows in the Customer table exceeds that threshold.

If true, copies customer data from SQL to ADLS in JSON format.

Implementation Highlights:

Dataset: Customer SQL dataset, Threshold CSV dataset

Activities Used:

Lookup activity (for reading threshold)

Stored procedure (or script) to get row count

If Condition activity

Copy activity

Sink: JSON output in ADLS under:
Customer/<Year>/<Month>/<Day>/
<img width="1920" height="1080" alt="ThresholdPipelineRunStatus" src="https://github.com/user-attachments/assets/42c4a9d9-467f-4fd6-9220-33dc8647aff9" />
<img width="1920" height="1080" alt="ThresholdPipelineExecutionSummary" src="https://github.com/user-attachments/assets/f83a9bd1-7ed9-4e2c-a809-e7a87ccb6bd2" />



✅ Task 2: Overwrite Data by Partitioned Path
Objective:
Ensure every run of the pipeline writes the customer JSON data to a dynamically partitioned folder:

sql
Copy
Edit
Customer/Year/Month/Day/
Key Concepts:

@utcNow('yyyy'), @utcNow('MM'), @utcNow('dd') used in sink dataset path.

This prevents overwriting while enabling versioned ingestion.
<img width="1920" height="1080" alt="PipelineSummary" src="https://github.com/user-attachments/assets/5f73eeed-caa2-41bb-a39f-10f76e1afd0e" />



✅ Task 3: Parameterized ForEach Pipeline for Copying Multiple Tables
Pipeline Name: Foreach_Example2

Objective:
Use a single Copy Activity inside ForEach to:

Copy all Product records where ProductID > 100

Copy all Customer records where CustomerID > 100 AND CustomerID < 1000

Key Highlights:

Pipeline Parameter:

json
Copy
Edit
[
  {"tableName": "Product", "filter": "ProductID > 100"},
  {"tableName": "Customer", "filter": "CustomerID > 100 AND CustomerID < 1000"}
]
Dynamic SQL Query in dataset:

csharp
Copy
Edit
@concat('SELECT * FROM ', item().tableName, ' WHERE ', item().filter)
Sink Path:

swift
Copy
Edit
foreach_output/@{item().tableName}/@{utcNow('yyyy/MM/dd')}
<img width="1476" height="887" alt="ExecutionDetailsForeach_Example2 Pipeline" src="https://github.com/user-attachments/assets/ffd3183d-80f6-4542-9f95-fec15cf98c61" />
<img width="1920" height="1080" alt="PipelineStatusForeach_Example2" src="https://github.com/user-attachments/assets/0c90cf47-357c-4d2c-b839-1fd0095b0e45" />



✅ Task 4: Data Flow - Join Customer (SQL) with Address (CSV)
Objective:
Join:

Customer data from SQL

CustomerAddress data from a CSV in ADLS

Apply filter:
CustomerID > 1000 AND CustomerID < 2000

Sort the output in ascending order of CustomerID and store it as a Parquet file.

Key Components:

Data Flow: df_join_customer_with_address

Source 1: SQL Customer table

Source 2: ADLS CSV file from:
https://adfstoragedata2.blob.core.windows.net/input/customer_address.csv

Join: On CustomerID (after type-casting to avoid mismatches)

Filter:

csharp
Copy
Edit
CustID > 1000 && CustID < 2000
Sink Output:
joined_output/customer_joined.parquet
<img width="1920" height="1080" alt="DataFlowSummary" src="https://github.com/user-attachments/assets/cb27a84f-4589-4a87-be06-43fbae7b5665" />


🗂 Folder Structure in ADLS Gen2
pgsql
Copy
Edit
adlsroot/
├── Customer/
│   └── 2025/
│       └── 07/
│           └── 17/
│               └── customer_data.json
├── foreach_output/
│   ├── Product/
│   └── Customer/
├── joined_output/
│   └── customer_joined.parquet
└── input/
    └── threshold.csv



    🧠 Key Learnings & Concepts
✅ Parameterization using expressions in ADF
✅ ForEach control for looping SQL table copies
✅ JSON-based dynamic queries
✅ Data Flow expression editing and type conversion
✅ Structured output via JSON & Parquet
✅ Logical condition-based pipeline branching
✅ Partitioning via dynamic folder paths
✅ Sink optimization using format-specific configurations


🔁 Reusability
All SQL queries and copy logic are dynamic and accept parameters.

Easily add more tables to copy by just extending the parameter list.

Modular Data Flows for future extensions (joining other datasets).



## ✅ Final Outcome Summary

| Task    | Status   | Output Type             | Tools Used                   |
|---------|----------|-------------------------|------------------------------|
| Task 1  | ✅ Done  | JSON                    | Lookup, If Condition, Copy  |
| Task 2  | ✅ Done  | JSON with Dynamic Path  | Sink Dynamic Path           |
| Task 3  | ✅ Done  | JSON & CSV              | ForEach + Dynamic SQL       |
| Task 4  | ✅ Done  | Parquet                 | Data Flow Join              |


---

## 🔚 Conclusion

This project demonstrates multiple core Azure Data Factory concepts including conditional logic, dynamic folder paths, parameterized ForEach loops, and Data Flow joins. All four tasks were successfully executed with different data formats (JSON, Parquet) and dynamic transformations.

## 📌 Notes
- All pipelines were tested using Debug and then published.
- Dynamic paths were used to avoid overwriting during multiple executions.
- Joins were performed with data type compatibility in mind (string conversion applied where needed).

