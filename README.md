# Migrate-MySQL-Database-to-MongoDB

When you export data from **MySQL Workbench to CSV**, and **datetime values are showing incorrectly** (like `53:00.0` instead of `2024-04-01 09:00:00.000000`), it's usually due to **Excel or the CSV viewer misinterpreting the datetime format**.

### ✅ Here's what's happening:
- The value `2024-04-01 09:00:00.000000` is stored correctly in MySQL.
- On export to CSV:
  - Workbench **includes microseconds**.
  - **Excel** or **other viewers** might parse it as a number (e.g., time duration or float), especially if it sees `:00.000000`.
  - So `09:00:00.000000` might show as `53:00.0` — which is Excel interpreting it as 53 minutes?

---

### 🛠 How to Fix It

#### ✅ Option 1: Format the datetime on export
Update your query to **format the datetime** as a string before export:

```sql
SELECT DATE_FORMAT(your_datetime_column, '%Y-%m-%d %H:%i:%s') AS formatted_time
FROM your_table;
```

Then export the result. This forces MySQL to **output it as a string**, which won't confuse Excel or CSV viewers.

---

#### ✅ Option 2: Open CSV with a text editor instead of Excel
- Try opening the `.csv` file with **Notepad++** or **VS Code**.
- You'll see the original datetime values correctly (e.g., `2024-04-01 09:00:00.000000`).
- Excel sometimes parses these into time duration — not actual datetime — which causes confusion.

---

#### ✅ Option 3: Use `.xlsx` or another tool for export
If you're using Excel and want better control, consider:
- Exporting as `.xlsx` using a Python script or GUI tool
- Using **DBeaver** or **HeidiSQL**, which export with more control over datetime types

---


#### ✅ Example of a database: mysql to CSV for MongoDB

```sql
    SELECT 
    early_exit_reason,
    employee_id,
    DATE_FORMAT(entry_date, '%Y-%m-%d') AS entry_date,
     DATE_FORMAT(entry_time, '%Y-%m-%d %H:%i:%s') AS entry_time,
     DATE_FORMAT(exit_time, '%Y-%m-%d %H:%i:%s') AS exit_time,
      global_day_status,
      late_entry_reason,
       month,
       name,
      outtime,
       DATE_FORMAT(present_time, '%Y-%m-%d %H:%i:%s') AS present_time,
    status,
    update_status,
    year
  FROM sonbidanvote23.employee_inserted_data;
```
