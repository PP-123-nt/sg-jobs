**Steps:**

1. Create new Jupyter Notebook in VSC
2. Copy zip file into the same folder as Jupyter Notebook
3. Import and extract the zip file -> reates SGJobData.csv   
4. Create sgjob.db database
5. Create job_raw SQL table
6. Create job_rawdf panda df (for data checking using python code)
7. Data check
   - a) max/min/average salary: with invalid values (max salary $25M)
   - b) occupationId: all blank with no record
   - c) 3988 rows has no records in all columns
   - d) min. experience year: max is 88
   - e) no. of vacancy: max is 999
   - f) 2022-10 to 2023-04 are probably a ramp-up as posting starts from 172 - 33K/mth, 2023-05 onwards posting are stabilize at the range of 60K - 84K
8. Create job_clean SQL table from job_raw table, exclude 3988 rows without record and drop occupationId column


**Analysis:**
1. Data Range: 2023-05 to 2024-05
2. Group Employment Types? 
    - Permanent/Full Time
    - Part Time/Temporary
    - Freelance/Flexi-work
    - Internship/Attachment
    - Contract
3. Salary Analysis: for Permanet/Full Time only, set criteria to replace outlier with min/max salary median value
4. Closed position: with no. of application by employment type, position level, no. of views
5. High Views and Application Analysis: reason (salary? position level? employment type?)
6. Category: if want to do analsis on this need to parse to split with multiple category per jobid







4. check which data need to clean
    -> average/max/min salary: for outlier/invalid/missing excl in salary related analysis, keep the records for other analysis
    -> missing record
    -> if missing record in other column is minimal, drop? or keep for other analysis?
    
5. after SQL table data clean then create 2 panda df 
    -> df_base: take only 1st category as primary category after clean json format
    -> df_exploded: take all categories after clean json format, 1 jobid has multiple rows with different category
