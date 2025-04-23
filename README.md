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

#### ✅ Java Code:

```sql

       readCSVForAttendanceData("C:\\Users\\01957\\Downloads/attendanceData1.csv");

       public  void readCSVForAttendanceData(String filePath) {
        String line;
              String regex = "\"([^\"]*)\"|([^,]+)"; // Regex to capture quoted and unquoted values
              Pattern pattern = Pattern.compile(regex);
      
              try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
                  br.readLine();
                  while ((line = br.readLine()) != null) {
                      List<String> values = new ArrayList<>();
                      Matcher matcher = pattern.matcher(line);
      
                      while (matcher.find()) {
                          if (matcher.group(1) != null) {
                              values.add(matcher.group(1)); // Quoted value
                          } else {
                              values.add(matcher.group(2)); // Unquoted value
                          }
                      }
      
                     // System.out.println(values.size()+"  "+values); // Print as a list
                      AttendanceData ee=new AttendanceData();
                      ee.setEarlyExitReason(values.get(0));
                      ee.setEmployeeId(values.get(1));
                      ee.setEntryDate(convertDate(values.get(2))  );// date 3
                      // ee.setEntryTime(parseDateTime(values.get(3),values.get(4)));
                      ee.setEntryTime( convertUtcToDhaka(parseDateTime(values.get(3))));
                     // ee.setExitTime(parseDateTime(values.get(3),values.get(5)));
                      ee.setExitTime(convertUtcToDhaka(parseDateTime(values.get(4))));
                      ee.setGlobalDayStatus(values.get(5));
                      ee.setLateEntryReason(values.get(6));
                      ee.setMonth(values.get(7));
                      ee.setName(values.get(8));
                      ee.setOuttime(values.get(9));
                     // ee.setPresentTime(parseDateTime(values.get(3),values.get(11)));
                      ee.setPresentTime( convertUtcToDhaka(parseDateTime(values.get(10))));
                      ee.setStatus(values.get(11));
                      ee.setUpdateStatus(values.get(12));
                      ee.setYear(values.get(13));
                      attendanceDataRepository.save(ee);
                      System.out.println(ee.toString());
                      System.out.println();
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }

          public static LocalDateTime convertUtcToDhaka(LocalDateTime utcDateTime) {
            // Attach UTC zone to the LocalDateTime
            ZonedDateTime utcZoned = utcDateTime.atZone(ZoneId.of("UTC"));
    
            // Convert to Asia/Dhaka zone
            ZonedDateTime dhakaZoned = utcZoned.withZoneSameInstant(ZoneId.of("Asia/Dhaka"));
    
            // Return as LocalDateTime in Asia/Dhaka
            return dhakaZoned.toLocalDateTime();
         }

         public static String convertUtcToDhaka(String inputTime) {
         DateTimeFormatter inputFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
         LocalDateTime utcDateTime = LocalDateTime.parse(inputTime, inputFormatter);

        ZonedDateTime utcZoned = utcDateTime.atZone(ZoneId.of("UTC"));
        ZonedDateTime dhakaZoned = utcZoned.withZoneSameInstant(ZoneId.of("Asia/Dhaka"));

        DateTimeFormatter outputFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        return dhakaZoned.format(outputFormatter);
       }

       public static LocalDateTime parseDateTime(String dateTimeStr) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("M/d/yyyy H:mm", Locale.ENGLISH);
        return LocalDateTime.parse(dateTimeStr, formatter);
      }
      public static String convertDate(String inputDate) {
        // Define the input formatter
        DateTimeFormatter inputFormatter = DateTimeFormatter.ofPattern("M/d/yyyy");
        DateTimeFormatter outputFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");

        // Parse the input date
        LocalDate date = LocalDate.parse(inputDate, inputFormatter);

        // Convert it to the fixed date (February 5 of the same year)
        LocalDate transformedDate = LocalDate.of(date.getYear(), date.getMonthValue(), date.getDayOfMonth());

        // Format and return as String
        return transformedDate.format(outputFormatter);
    }
```
