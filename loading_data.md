### **Loading Data into Snowflake with COPY Command and Options**

---

# **Overview**

Snowflake's data loading capabilities are designed for flexibility and efficiency. The `COPY INTO` command is a core part of this process, allowing bulk data loading into Snowflake tables from both internal and external stages. This README provides theoretical knowledge, practical examples, and best practices for using the `COPY` command, making it a valuable resource for data engineers.

---

## **1. Data Loading in Snowflake**

### **Theoretical Concepts**

1. **Data Sources**:
   - **Internal Stages**: Files uploaded directly to Snowflake.
   - **External Stages**: Files stored in cloud services like AWS S3, Azure Blob Storage, or Google Cloud Storage.
   - **Local Files**: Files stored on a local machine can be staged and loaded into Snowflake.

2. **File Formats Supported**:
   - **Structured Data**: CSV, TSV, PARQUET, AVRO, ORC.
   - **Semi-Structured Data**: JSON, XML.
   - Compression Formats: GZIP, BZIP2, DEFLATE, and more.

3. **Pipeline for Data Loading**:
   - Stage files → Validate file formats → Copy files into tables → Monitor loading progress.

---

### **Practical Steps to Load Data**

#### **1. Preparing the Stage**
- **Internal Stage**: Upload files to Snowflake's managed storage.
- **External Stage**: Configure cloud storage integration to access external files.

#### **2. Creating File Formats**
Define reusable file format objects for consistency in loading.

```sql
CREATE FILE FORMAT csv_format
TYPE = 'CSV'
FIELD_OPTIONALLY_ENCLOSED_BY = '"'
SKIP_HEADER = 1;
```

#### **3. Copying Data into Tables**
Use the `COPY INTO` command to load data into Snowflake tables.

```sql
COPY INTO target_table
FROM @stage_name
FILE_FORMAT = (FORMAT_NAME = 'csv_format')
ON_ERROR = 'CONTINUE';
```

---

## **2. Internal Stages**

Internal stages are Snowflake-managed locations where files are temporarily stored before loading into tables.

### **Examples**

1. **Table Stage**: Automatically created with each table.  
   Files are stored temporarily for a specific table.
   ```sql
   -- Upload file to a table stage
   PUT file://path/to/sales.csv @%sales;
   
   -- Load data from the table stage
   COPY INTO sales
   FROM @%sales
   FILE_FORMAT = (TYPE = 'CSV');
   ```

2. **Named Stage**: Explicitly created and reusable across multiple tables.
   ```sql
   -- Create a named stage
   CREATE STAGE my_stage;

   -- Upload files to the stage
   PUT file://path/to/sales.csv @my_stage;

   -- Load data into the table
   COPY INTO sales
   FROM @my_stage
   FILE_FORMAT = (TYPE = 'CSV');
   ```

---

## **3. External Stages**

External stages are linked to external cloud storage systems. These stages require pre-configured storage integrations.

### **Example: Load Data from AWS S3**
```sql
-- Create an external stage
CREATE STAGE ext_stage
URL = 's3://my_bucket/data/'
STORAGE_INTEGRATION = my_integration;

-- Load data into the table
COPY INTO sales
FROM @ext_stage
FILE_FORMAT = (TYPE = 'PARQUET');
```

---

## **4. COPY Command Options**

### **Comprehensive COPY Options**

| **Option**           | **Description**                                                                                       | **Example**                                                                                     |
|-----------------------|-------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| `FILE_FORMAT`         | Defines the file type and associated properties (e.g., delimiter, compression).                      | `FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"')`                             |
| `ON_ERROR`            | Specifies error handling behavior (`CONTINUE`, `SKIP_FILE`, `ABORT_STATEMENT`).                      | `ON_ERROR = 'CONTINUE'`                                                                       |
| `FORCE`               | Reloads data even if it has already been loaded.                                                     | `FORCE = TRUE`                                                                                |
| `PURGE`               | Deletes successfully loaded files from the stage after the operation.                                | `PURGE = TRUE`                                                                                |
| `MATCH_BY_COLUMN_NAME`| Matches columns by name or position (`CASE_SENSITIVE`, `CASE_INSENSITIVE`, `NONE`).                  | `MATCH_BY_COLUMN_NAME = 'CASE_SENSITIVE'`                                                    |
| `VALIDATION_MODE`     | Validates data without loading it into the table (`RETURN_ERRORS`, `RETURN_ALL_ERRORS`).             | `VALIDATION_MODE = 'RETURN_ERRORS'`                                                          |

---

### **Error Handling**

Snowflake offers flexible error handling options during data loading:

| **Option**          | **Behavior**                                                       |
|----------------------|-------------------------------------------------------------------|
| `CONTINUE`          | Load valid rows and log errors for invalid rows.                  |
| `SKIP_FILE`         | Skip files with errors and load others.                           |
| `ABORT_STATEMENT`   | Abort the entire operation if any error occurs (default).         |

---

### **Advanced Example: Load Data with Error Handling**
```sql
COPY INTO sales
FROM @my_stage
FILE_FORMAT = (TYPE = 'CSV')
ON_ERROR = 'SKIP_FILE'; -- Skip problematic files
```

---

## **5. Real-Life Scenarios**

### **Scenario 1: Validating Data Before Loading**
To check for errors without actually loading data:
```sql
COPY INTO sales
FROM @my_stage
FILE_FORMAT = (TYPE = 'CSV')
VALIDATION_MODE = 'RETURN_ERRORS';
```

---

### **Scenario 2: Loading Only New Data**
Use the `FORCE` option to prevent reloading files:
```sql
COPY INTO sales
FROM @my_stage
FILE_FORMAT = (TYPE = 'CSV')
FORCE = FALSE;
```

---

### **Scenario 3: Automating Data Loading**
Combine the `PUT` and `COPY` commands in a script:
```bash
#!/bin/bash

# Upload file to Snowflake stage
snowsql -q "PUT file://data/sales_data.csv @my_stage;"

# Load data into Snowflake table
snowsql -q "COPY INTO sales FROM @my_stage FILE_FORMAT = (TYPE = 'CSV') ON_ERROR = 'CONTINUE';"
```

---

## **6. Best Practices**

1. **Optimize File Sizes**:
   - Files should be between 100 MB and 1 GB for optimal performance.

2. **Use Compression**:
   - Compress files before uploading to reduce transfer time (e.g., `.gzip`).

3. **Monitor Load Performance**:
   - Analyze the `COPY_HISTORY` view for insights.
   ```sql
   SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.COPY_HISTORY WHERE TABLE_NAME = 'sales';
   ```

4. **Predefine File Formats**:
   - Use `CREATE FILE FORMAT` for consistency and reusability.

5. **Automate with Snowpipe**:
   - For real-time loading, use Snowpipe to detect and load new files automatically.

---

## **7. Common Errors and Troubleshooting**

| **Error**                         | **Cause**                                         | **Solution**                                       |
|-----------------------------------|--------------------------------------------------|--------------------------------------------------|
| File not found in the stage       | File was not uploaded to the correct stage.       | Verify the stage and file path.                 |
| Column mismatch                   | Source and target columns do not align.           | Use `MATCH_BY_COLUMN_NAME`.                     |
| Data validation errors            | Invalid data format or type mismatch.             | Validate data using `VALIDATION_MODE`.          |

---

## **8. Conclusion**

Snowflake's `COPY INTO` command, paired with its versatile options, provides a robust framework for loading data into Snowflake tables. With support for internal and external stages, as well as structured and semi-structured data, data engineers can handle diverse ingestion scenarios efficiently.

This guide offers a mix of theory, practical examples, and best practices to ensure successful data loading operations. For advanced workflows, consider integrating with Snowpipe for real-time data ingestion or leveraging automated scripts.

--- 
