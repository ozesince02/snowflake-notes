# Data Sharing in Snowflake

## Table of Contents
1. [Introduction](#introduction)
2. [What is Data Sharing in Snowflake?](#what-is-data-sharing-in-snowflake)
3. [Benefits of Data Sharing](#benefits-of-data-sharing)
4. [Components of Data Sharing](#components-of-data-sharing)
5. [Types of Data Sharing](#types-of-data-sharing)
6. [Setting Up Data Sharing](#setting-up-data-sharing)
   - [Creating a Share](#creating-a-share)
   - [Granting Access](#granting-access)
   - [Consuming Shared Data](#consuming-shared-data)
   - [Sharing with Non-Snowflake Users](#sharing-with-non-snowflake-users)
7. [Advanced Topics](#advanced-topics)
   - [Reader Accounts](#reader-accounts)
   - [Dynamic Data Masking](#dynamic-data-masking)
   - [Data Clean Rooms](#data-clean-rooms)
   - [Integration with Other Platforms](#integration-with-other-platforms)
8. [Use Cases](#use-cases)
9. [Best Practices](#best-practices)
10. [Limitations and Considerations](#limitations-and-considerations)
11. [Examples](#examples)
    - [Basic Example](#basic-example)
    - [Advanced Example](#advanced-example)
12. [References](#references)

---

## Introduction

Snowflake provides a seamless and secure way to share data across different organizations and platforms without the need for data replication or complex ETL processes. This document aims to provide a comprehensive understanding of Snowflake's data sharing capabilities, starting from basic concepts to advanced use cases.

---

## What is Data Sharing in Snowflake?

Data Sharing in Snowflake allows providers to share live, real-time data with consumers in a secure and efficient manner. Unlike traditional methods, Snowflake eliminates the need for physical data movement by enabling direct access to the data.

---

## Benefits of Data Sharing

- **Real-Time Data Access:** Consumers access live data without replication delays.
- **Cost Efficiency:** Eliminates the need for complex ETL pipelines and storage costs for duplicated data.
- **Simplified Collaboration:** Facilitates secure data sharing between multiple parties.
- **Governance and Security:** Provides granular control over data access.

---

## Components of Data Sharing

1. **Provider:** The account that owns the data and creates the share.
2. **Consumer:** The account or user who consumes the shared data.
3. **Shares:** A named object created by the provider to share specific data objects (e.g., tables, views).
4. **Reader Accounts:** Consumer accounts created and managed by the provider for organizations that do not have their own Snowflake accounts.

---

## Types of Data Sharing

1. **Direct Sharing:** Sharing data between two Snowflake accounts within the same region.
2. **Reader Accounts:** Sharing with organizations that do not have Snowflake accounts.
3. **Cross-Cloud Sharing:** Sharing data across different cloud platforms (AWS, Azure, GCP).
4. **Cross-Region Sharing:** Sharing data across different geographic regions.

---

## Setting Up Data Sharing

### Creating a Share

1. Identify the data objects (tables, views, schemas) to share.
2. Use the `CREATE SHARE` command:
   ```sql
   CREATE SHARE my_share;
   ```
3. Add data to the share:
   ```sql
   GRANT USAGE ON DATABASE my_database TO SHARE my_share;
   GRANT SELECT ON SCHEMA my_schema TO SHARE my_share;
   GRANT SELECT ON TABLE my_table TO SHARE my_share;
   ```

### Granting Access

Provide access to the consumer’s Snowflake account:
```sql
ALTER SHARE my_share ADD ACCOUNT = '<consumer_account_identifier>';
```

### Consuming Shared Data

Consumers can add the shared data to their environment:
```sql
CREATE DATABASE shared_db FROM SHARE <provider_account>.<share_name>;
```

### Sharing with Non-Snowflake Users

For organizations or individuals who do not have their own Snowflake accounts, you can create user accounts within your Snowflake instance and assign them appropriate roles and permissions:

1. **Create a New User Account:**
   ```sql
   CREATE USER non_snowflake_user
   PASSWORD = 'SecurePassword123!'
   DEFAULT_ROLE = 'PUBLIC'
   DEFAULT_WAREHOUSE = 'my_warehouse';
   ```

2. **Grant Roles and Permissions:**
   Assign roles to control the level of access:
   ```sql
   CREATE ROLE shared_data_role;
   GRANT USAGE ON DATABASE my_database TO ROLE shared_data_role;
   GRANT SELECT ON SCHEMA my_schema TO ROLE shared_data_role;
   GRANT SELECT ON TABLE my_table TO ROLE shared_data_role;
   GRANT ROLE shared_data_role TO USER non_snowflake_user;
   ```

3. **Provide Access Information:**
   Share the login details and necessary Snowflake client tools (e.g., SnowSQL, web interface) with the user.

4. **Monitor and Manage Access:**
   Regularly audit and revoke permissions as needed:
   ```sql
   REVOKE ROLE shared_data_role FROM USER non_snowflake_user;
   DROP USER non_snowflake_user;
   ```

---

## Advanced Topics

### Reader Accounts

- Reader accounts are managed by the provider for consumers without Snowflake accounts.
- Use the following to create a reader account:
  ```sql
  CREATE MANAGED ACCOUNT reader_account_name ADMIN_NAME = 'admin_user';
  ```

### Dynamic Data Masking

Apply masking policies to sensitive data:
```sql
CREATE MASKING POLICY mask_ssn AS (val STRING) RETURNS STRING ->
CASE
  WHEN CURRENT_ROLE() IN ('AUTHORIZED_ROLE') THEN val
  ELSE 'XXX-XX-XXXX'
END;
```

### Data Clean Rooms

Enable secure collaboration while maintaining privacy:
- Use anonymized data sharing.
- Employ restricted views to ensure compliance.

### Integration with Other Platforms

Leverage Snowflake’s integration capabilities:
- APIs for automated sharing.
- Connectors for BI tools like Tableau, Power BI.

---

## Use Cases

1. **Healthcare:** Sharing de-identified patient data for research.
2. **Finance:** Providing real-time market data to consumers.
3. **Retail:** Sharing inventory and sales data with suppliers.
4. **Education:** Collaborating on academic research data.

---

## Best Practices

- Use **Row Access Policies** for granular control.
- Regularly audit shared data access.
- Avoid over-granting permissions.
- Document all shares and their purposes.

---

## Limitations and Considerations

- Sharing across regions incurs data transfer costs.
- Dynamic masking policies may impact performance.
- Cross-cloud sharing requires proper configuration.

---

## Examples

### Basic Example

Provider:
```sql
CREATE SHARE sales_share;
GRANT SELECT ON TABLE sales_data TO SHARE sales_share;
ALTER SHARE sales_share ADD ACCOUNT = 'consumer_account';
```

Consumer:
```sql
CREATE DATABASE sales_data_consumer FROM SHARE provider_account.sales_share;
SELECT * FROM sales_data_consumer.sales_data;
```

### Advanced Example

Provider:
```sql
CREATE SHARE secure_share;
GRANT USAGE ON SCHEMA sensitive_schema TO SHARE secure_share;
GRANT SELECT ON VIEW secure_view TO SHARE secure_share;
ALTER SHARE secure_share ADD ACCOUNT = 'consumer_account';
```

Consumer:
```sql
CREATE DATABASE secure_data FROM SHARE provider_account.secure_share;
SELECT * FROM secure_data.secure_view;
```

---

## References

- [Snowflake Documentation](https://docs.snowflake.com/)
- [Snowflake Data Sharing Overview](https://docs.snowflake.com/en/user-guide/data-sharing-intro.html)
- [Snowflake Best Practices for Data Sharing](https://docs.snowflake.com/en/user-guide/data-sharing-best-practices.html)
- [Dynamic Data Masking](https://docs.snowflake.com/en/sql-reference/sql/create-masking-policy.html)

