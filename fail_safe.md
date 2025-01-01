### **Fail-Safe**

# **Fail-Safe in Snowflake**

## **Overview**
Fail-Safe is a Snowflake feature designed for **disaster recovery**. It provides a 7-day recovery window for Snowflake to restore data after it has been permanently removed from Time Travel.

---

## **How It Works**
- After the Time Travel retention period ends, deleted data enters the Fail-Safe period.
- Fail-Safe is intended for emergencies and is managed by Snowflake.
- Direct access by users to Fail-Safe data is **not allowed**.

---

## **Key Features**
- **Automatic**: Fail-Safe is built-in and does not require manual configuration.
- **7-Day Recovery**: Data remains recoverable by Snowflake support for seven days after Time Travel ends.
- **Last Resort**: Only used for catastrophic events, not for regular data recovery.

---

## **Scenarios**

### 1. Accidental Data Deletion
If data is accidentally deleted and Time Travel retention has expired:
- Contact Snowflake Support to initiate data recovery through Fail-Safe.

### 2. Disaster Recovery
Fail-Safe ensures critical data can be recovered in case of severe system failures or corruption.

---

## **Example Timeline**

| **Event**              | **Action**                  | **Time Travel (1 Day)** | **Fail-Safe (7 Days)** |
|-------------------------|-----------------------------|--------------------------|-------------------------|
| Day 0                  | Table dropped               | Recoverable via `UNDROP`| N/A                     |
| Day 1                  | Time Travel expires         | N/A                      | Recoverable via Fail-Safe|
| Day 8                  | Fail-Safe expires           | N/A                      | Data permanently deleted|

---

## **Limitations**
- Fail-Safe recovery requires assistance from Snowflake support.
- Data recovery through Fail-Safe is **not instant** and may take time.
- Fail-Safe is not a substitute for backups or versioning.

---

## **Best Practices**
- Use Time Travel for short-term data recovery.
- Leverage regular backups and external storage for long-term data retention.
- Do not rely solely on Fail-Safe for disaster recovery; it is a last-resort mechanism.

---
