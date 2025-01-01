### **Time Travel**

# **Time Travel in Snowflake**

## **Overview**
Time Travel in Snowflake allows you to access historical data for querying, cloning, and recovery. It provides the ability to "travel back in time" to a specific point to analyze data changes or recover accidentally deleted or modified data.

---

## **Key Features**
- **Point-in-Time Querying**: Access data as it existed at a specific time.
- **Cloning from a Historical State**: Create a clone from a past version of the object.
- **Data Recovery**: Restore deleted objects or undo accidental changes.

---

## **Time Travel Retention Period**
- **Default**: 1 day for standard accounts.
- **Extended Time Travel** (Optional): Up to 90 days (requires Enterprise or higher edition).

---

## **Syntax**

### Querying Historical Data
```sql
SELECT * FROM table_name AT (TIMESTAMP => 'YYYY-MM-DD HH24:MI:SS');
```

### Restoring Dropped Tables
```sql
UNDROP TABLE table_name;
```

### Cloning a Historical State
```sql
CREATE TABLE new_table_name CLONE table_name AT (TIMESTAMP => 'YYYY-MM-DD HH24:MI:SS');
```

### Query by Offset
```sql
SELECT * FROM table_name BEFORE (STATEMENT => 'QUERY_ID');
```

---

## **Examples**

### 1. Query Historical Data by Timestamp
```sql
SELECT * FROM sales AT (TIMESTAMP => '2024-12-01 10:00:00');
```

### 2. Recover a Dropped Table
```sql
DROP TABLE sales; -- Table is accidentally dropped

-- Restore the table
UNDROP TABLE sales;
```

### 3. Clone from a Historical State
```sql
-- Clone the sales table as it existed 1 hour ago
CREATE TABLE sales_snapshot CLONE sales AT (OFFSET => -1 * INTERVAL '1 HOUR');
```

---

## **Use Cases**
1. **Data Auditing**: Verify data changes over time or recover historical records for compliance.
2. **Disaster Recovery**: Recover accidentally dropped or modified tables.
3. **Testing and Development**: Clone past states of data for testing purposes.
4. **Debugging**: Analyze data before and after a specific operation to debug issues.

---

## **Limitations**
- Time Travel is limited by the retention period configured for your account.
- Cloning or querying from Time Travel does not work beyond the retention window.
- Extended Time Travel requires additional costs.

---

## **Best Practices**
- Use Time Travel for short-term recovery and auditing.
- Leverage **cloning from Time Travel** to create historical backups without affecting the current data.
- Set appropriate retention policies for critical tables and databases.

---
