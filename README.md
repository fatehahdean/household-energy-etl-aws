# Household Energy ETL Pipeline (AWS)

A serverless ETL pipeline that cleans raw household electricity data and makes it queryable with SQL.

## Architecture

S3 (`raw/`) → Lambda (Python, streaming ETL) → S3 (`processed/`) → Athena (SQL)

## Dataset

UCI Individual Household Electric Power Consumption: 2,075,259 minute-level readings from one household.

## Key Results

- Cleaned 2,049,280 rows and skipped 25,979 rows with missing values (`?`)
- Peak average demand at 20:00 (1.899 kW)
- Lowest average demand between 00:00 and 02:00 (around 0.44 to 0.66 kW)

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
