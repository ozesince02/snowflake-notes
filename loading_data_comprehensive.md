# **Snowflake Data Loading**

---

## **Table of Contents**

1. [Overview](#overview)
2. [End-to-End Workflow](#end-to-end-workflow)
3. [Loading Data into Snowflake](#loading-data-into-snowflake)
4. [Stages in Snowflake](#stages-in-snowflake)
   - [Internal Stages](#internal-stages)
   - [External Stages](#external-stages)
5. [COPY Command](#copy-command)
   - [Syntax and Options](#syntax-and-options)
   - [Error Handling](#error-handling)
6. [Data Validation and Transformation](#data-validation-and-transformation)
7. [Performance Optimization](#performance-optimization)
8. [Monitoring and Auditing](#monitoring-and-auditing)
9. [Security and Access Control](#security-and-access-control)
10. [Error Handling and Recovery](#error-handling-and-recovery)
11. [Continuous Data Loading with Snowpipe](#continuous-data-loading-with-snowpipe)
12. [Semi-Structured Data Handling](#semi-structured-data-handling)
13. [Data Archiving and Unloading](#data-archiving-and-unloading)
14. [Project Structure and Automation](#project-structure-and-automation)
15. [Common Pitfalls and Best Practices](#common-pitfalls-and-best-practices)

---

## **Overview**

Snowflake provides a robust and scalable data platform tailored for data engineers. It enables efficient data storage, querying, and integration with minimal operational overhead. This guide focuses on loading data, optimizing performance, and managing data workflows in Snowflake.

---

## **End-to-End Workflow**

### **Step 1: Set Up the Environment**
- Create a database and schema for organizing your data.
  ```sql
  CREATE DATABASE my_database;
  CREATE SCHEMA my_schema;
  ```

### **Step 2: Prepare Staging Areas**
- Configure internal or external stages for temporary file storage.

### **Step 3: Upload Files to Stages**
- Use the `PUT` command to upload files to internal stages.
- Configure storage integration for external stages.

### **Step 4: Load Data**
- Use the `COPY INTO` command to load data into Snowflake tables.

### **Step 5: Validate and Transform Data**
- Query the loaded data for validation and apply necessary transformations.

### **Step 6: Monitor Load Operations**
- Use system views like `COPY_HISTORY` for auditing and troubleshooting.

---

## **Loading Data into Snowflake**

Snowflake supports bulk and continuous data loading through structured workflows.

### **Bulk Loading**
- Best suited for large datasets.
- Uses `COPY INTO` command from internal or external stages.

### **Continuous Loading**
- Uses **Snowpipe** for real-time or near-real-time data ingestion.

### **File Formats Supported**
- **Structured Data**: CSV, TSV, PARQUET, AVRO, ORC.
- **Semi-Structured Data**: JSON, XML.

---

## **Stages in Snowflake**

Stages act as intermediaries for files before loading into tables.

### **Internal Stages**
- **Table Stage**: Associated with a specific table.
  ```sql
  PUT file://path/to/data.csv @%my_table;
  COPY INTO my_table FROM @%my_table FILE_FORMAT = (TYPE = 'CSV');
  ```
- **Named Stage**: Created explicitly and reusable.
  ```sql
  CREATE STAGE my_stage;
  PUT file://path/to/data.csv @my_stage;
  ```

### **External Stages**
- Integrate with cloud storage like AWS S3.
  ```sql
  CREATE STAGE ext_stage URL = 's3://my_bucket/' STORAGE_INTEGRATION = my_integration;
  ```

---

## **COPY Command**

### **Syntax and Options**
```sql
COPY INTO target_table
FROM stage_location
FILE_FORMAT = (TYPE = 'CSV')
ON_ERROR = 'CONTINUE';
```

### **Key Options**
- `ON_ERROR`: How to handle errors (`CONTINUE`, `SKIP_FILE`, `ABORT_STATEMENT`).
- `PURGE`: Deletes files after a successful load.
- `FORCE`: Reloads data even if already loaded.

---

## **Data Validation and Transformation**

### **Validation Before Loading**
```sql
COPY INTO target_table
FROM @my_stage
FILE_FORMAT = (TYPE = 'CSV')
VALIDATION_MODE = 'RETURN_ERRORS';
```

### **Transforming Semi-Structured Data**
```sql
SELECT column_name:json_field::STRING AS transformed_field
FROM my_table;
```

---

## **Performance Optimization**

1. **Optimize File Size**:
   - Files between 100 MB and 1 GB are ideal.

2. **Enable Auto Clustering**:
   ```sql
   ALTER TABLE my_table CLUSTER BY (column_name);
   ```

3. **Compress Files**: Use formats like GZIP to reduce storage and transfer times.

---

## **Monitoring and Auditing**

### **Query COPY_HISTORY**
```sql
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.COPY_HISTORY
WHERE TABLE_NAME = 'my_table';
```

---

## **Security and Access Control**

- Set up **role-based access control (RBAC)**.
  ```sql
  GRANT USAGE ON STAGE my_stage TO ROLE my_role;
  ```
- Use secure storage integrations for external stages.

---

## **Error Handling and Recovery**

- Use **Time Travel** for recovering dropped or overwritten data.
  ```sql
  SELECT * FROM my_table AT (OFFSET => -3600);
  ```
- Handle rejected rows programmatically.
  ```sql
  COPY INTO my_table
  VALIDATION_MODE = 'RETURN_ERRORS';
  ```

---

## **Continuous Data Loading with Snowpipe**

### **Configuration**
```sql
CREATE PIPE my_pipe
AUTO_INGEST = TRUE
AS COPY INTO my_table
FROM @my_stage
FILE_FORMAT = (TYPE = 'CSV');
```

---

## **Semi-Structured Data Handling**

### **Load JSON Data**
```sql
COPY INTO my_table
FROM @my_stage
FILE_FORMAT = (TYPE = 'JSON');
```

### **Query Nested JSON**
```sql
SELECT data:field1::STRING, data:field2::INTEGER
FROM my_table;
```

---

## **Data Archiving and Unloading**

### **Unload Data**
```sql
COPY INTO @my_stage
FROM my_table
FILE_FORMAT = (TYPE = 'CSV');
```

---

## **Project Structure and Automation**

### **Suggested Project Structure**
```
project-root/
  ├── sql-scripts/
  │   ├── create_stages.sql
  │   ├── load_data.sql
  ├── config/
  │   ├── storage_integration.yaml
  └── logs/
      └── load_history.log
```

### **Automating Workflows**
- Use `snowsql` for scripting and CI/CD pipelines.
```bash
snowsql -q "PUT file://data.csv @my_stage;"
snowsql -q "COPY INTO my_table FROM @my_stage;"
```

---

## **Common Pitfalls and Best Practices**

1. **Define File Formats**:
   - Use `CREATE FILE FORMAT` for consistency.
2. **Monitor Regularly**:
   - Query `COPY_HISTORY` for insights.
3. **Use Stages Effectively**:
   - Organize stages for efficient data management.
4. **Validate Before Loading**:
   - Use `VALIDATION_MODE` to catch errors early.

---
