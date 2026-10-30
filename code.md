## 🚨 What updates and what doesn't after csv file is updated with latest records

Your Database Tables will update perfectly, but your Python objects (like DataFrames) will NOT update automatically.

Here is the breakdown:

The SQL Tables (job_raw, job_clean): Yes! Because you run CREATE OR REPLACE, these will be wiped and rebuilt with the fresh data inside the sgjob.db file.

The Pandas DataFrames (job_rawdf): NO. If you created a DataFrame yesterday using job_rawdf = jobcon.execute(...).df(), it is a "photograph" of the old data. It will not update. You must re-run the cell that creates job_rawdf to pull the fresh data into Pandas.

SQL Views: If you created a view (like the employment_types_summary from earlier), YES, it will automatically show the new data. Because a View is just a saved query, it looks at job_raw fresh every time you query it.


**Tomorrow, when you download the new SGJobData.csv, run this exact block of code:**

```python
import duckdb

# 1. Connect to the database
jobcon = duckdb.connect('sgjob.db')

# 2. OVERWRITE job_raw with the new CSV (Use OR REPLACE!)
jobcon.execute("""
    CREATE OR REPLACE TABLE job_raw AS 
    SELECT * FROM read_csv_auto('SGJobData.csv')
""")

# 3. OVERWRITE job_clean using the newly updated job_raw
jobcon.execute("""
    CREATE OR REPLACE TABLE job_clean AS 
    SELECT * EXCLUDE (occupationId)
    FROM job_raw
    WHERE metadata_jobPostId IS NOT NULL
""")

# 4. (Optional) Re-pull into Pandas if you need a DataFrame
# This ensures your Pandas data is fresh!
job_clean_df = jobcon.execute("SELECT * FROM job_clean").df()

print("✅ Database and DataFrames updated with the latest data!")
```








<br>

# 1. keep 1st category (primary_category)
# 使用 try_cast 防止因存在格式错误的 JSON 而中断
query_primary = """
    WITH parsed AS (
        SELECT *,
               TRY_CAST(categories AS JSON) AS cat_json
        FROM jobdf
    )
    SELECT * EXCLUDE (cat_json, categories),
           cat_json->>'$[0].category' AS primary_category
    FROM parsed
"""

print("正在处理 primary_category...")
jobdf_base = jobcon.execute(query_primary).df() 
print(f"✅ jobdf_base shape: {jobdf_base.shape}")

# 2. expand category (df_exploded): multiple rows for same job id with > 1 category
query_exploded = """
    WITH parsed AS (
        SELECT *,
               TRY_CAST(categories AS JSON) AS cat_json
        FROM jobdf
    )
    SELECT * EXCLUDE (cat_json, categories),
           unnest(json_extract_string(cat_json, '$[*].category')) AS category
    FROM parsed
"""

print("\n正在展开所有 categories...")
jobdf_exploded = jobcon.execute(query_exploded).df()
print(f"✅ jobdf_exploded shape: {jobdf_exploded.shape}")


# Delete the old DataFrames to free up RAM
del jobdf_base
del jobdf_exploded

print("Old DataFrames deleted. Memory cleared.")


# Delete the old table
jobcon.execute("DROP TABLE IF EXISTS jobdf")
print("Table 'jobdf' deleted successfully.")


