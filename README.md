# Household Energy ETL Pipeline (AWS)

A serverless ETL pipeline that cleans raw household electricity data and makes it queryable with SQL.

## Architecture

S3 (`raw/`) → Lambda (Python, streaming ETL) → S3 (`processed/`) → Athena (SQL)

## Dataset

UCI Individual Household Electric Power Consumption: 2,075,259 minute-level readings from one household.

## Key Results

- Cleaned 2,049,280 rows and skipped 25,979 rows with missing values (`?`)
- Peak average demand at 20:00 (1.899 kW)
- Lowest average demand between 00:00 and 05:00 (around 0.44 to 0.66 kW, lowest at 04:00)

## Setup Screenshots

### 1. S3 bucket layout
Bucket with `raw/`, `processed/` and `athena-results/` folders.

<img width="1441" height="695" alt="1-s3-bucket" src="https://github.com/user-attachments/assets/df7805d9-59fc-436b-987a-403ffd544b69" />

### 2. Lambda configuration
Memory set to 1024 MB and timeout to 5 minutes.

<img width="1439" height="698" alt="2-lambda-config" src="https://github.com/user-attachments/assets/7e4adc3d-1d52-4c16-a4b4-3e57af0344e5" />

### 3. Lambda run after streaming refactor
2,049,280 rows written, 25,979 skipped, peak memory 271 MB, runtime about 16 seconds.

<img width="1128" height="540" alt="4-Lambda-execution-result" src="https://github.com/user-attachments/assets/643e0d73-a690-4492-a897-415e7e9702bf" />

### 4. Processed output in S3
Cleaned CSV written to `processed/` (126 MB).

<img width="1447" height="738" alt="5-s3-processed" src="https://github.com/user-attachments/assets/cca9d225-ef6a-418a-afb3-ca2c666765f2" />

### 5. Athena table preview
Sample rows read directly from the processed CSV.

![Athena table preview] <img width="1418" height="707" alt="6-Athena-table" src="https://github.com/user-attachments/assets/e4146fc9-e39b-45be-9063-5f7763dece71" />


### 6. Peak daily demand
Top 10 days by peak active power. Each day has 1,440 readings, one per minute.

![Peak daily demand](screenshots/7-peak-daily-demand.png)

### 7. Average demand by hour (00:00 to 09:00)
Lowest demand overnight, rising after sunrise.

![Hourly demand part 1](screenshots/8-Hourly-demand-query1.png)

### 8. Average demand by hour (10:00 to 23:00)
Demand peaks in the evening, highest at 20:00.

![Hourly demand part 2](screenshots/8-Hourly-demand-query2.png)

## Engineering Notes

- The first version loaded the full 133 MB file into memory and timed out at 300 seconds.
- The current version streams the file line by line and writes to `/tmp`. Peak memory dropped from 1024 MB to 271 MB, and runtime is about 16 seconds.
- The execution role currently uses `PowerUserAccess`. This is a known limitation. It should be replaced with a least-privilege role scoped to this bucket.
- Budget alert email is not yet verified because of an IAM permission issue.

## Example Query

```sql
SELECT CAST(SUBSTR(time_str, 1, 2) AS integer) AS hour_of_day,
       ROUND(AVG(global_active_power), 3) AS avg_kw
FROM energy_db.household_power
GROUP BY 1
ORDER BY 1;
```

## Next Steps

- Replace `PowerUserAccess` with a scoped role
- Add an S3 trigger for automatic processing
- Convert output to Parquet
