# 🗄️ Layoffs Data Cleaning Project (SQL Server)

## 📌 Overview
This project demonstrates how to **clean, transform, and analyze raw layoffs data** using **SQL Server**.  
The dataset contains information about company layoffs, industries, countries, funding, and dates.  
The goal of the project is to:
- Standardize inconsistent values.
- Handle missing/null data.
- Convert data types to proper formats.
- Remove duplicate/unnecessary rows.
- Perform exploratory analysis using SQL queries.

---

## ⚙️ Steps in the Project

### 1. Create Database & Temporary Table
```sql
CREATE DATABASE project;

SELECT *
INTO temp_layoffs
FROM layoffs
WHERE 1 = 0;

INSERT INTO temp_layoffs
SELECT *
FROM layoffs;
```
We copy the data into a new working table `temp_layoffs` to avoid touching the raw data.

---

### 2. Standardizing Text Data
- Trim extra spaces from company names:
```sql
UPDATE temp_layoffs
SET company = LTRIM(RTRIM(company));
```
- Standardize industries (e.g., `crypto` → `Crypto`):
```sql
UPDATE temp_layoffs
SET industry = 'Crypto'
WHERE industry LIKE 'crypto%';
```
- Fix country names (remove trailing dots in `United States.`):
```sql
UPDATE temp_layoffs
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```

---

### 3. Cleaning Date Column
Convert the `date` column to SQL **DATE** type:
```sql
UPDATE temp_layoffs
SET date = TRY_CONVERT(DATE, date, 101);

ALTER TABLE temp_layoffs
ALTER COLUMN date DATE;
```

---

### 4. Handling NULL & Invalid Data
- Convert string `"NULL"` to actual `NULL`:
```sql
ALTER TABLE temp_layoffs
ALTER COLUMN percentage_laid_off FLOAT NULL;

UPDATE temp_layoffs
SET percentage_laid_off = NULL
WHERE percentage_laid_off = 'NULL';

ALTER TABLE temp_layoffs
ALTER COLUMN total_laid_off INT NULL;

UPDATE temp_layoffs
SET total_laid_off = NULL
WHERE total_laid_off = 'NULL';
```
- Replace empty strings in industry:
```sql
UPDATE temp_layoffs
SET industry = NULL
WHERE industry = '' OR industry = 'null';
```

---

### 5. Removing Useless Rows
```sql
DELETE
FROM temp_layoffs
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

---

### 6. Fixing Missing Industry Values
Fill missing `industry` values based on other rows of the same company:
```sql
UPDATE t1
SET t1.industry = t2.industry
FROM temp_layoffs t1
JOIN temp_layoffs t2
ON t1.company = t2.company
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```

---

### 7. Exploratory Data Analysis (EDA)

- Max & Min layoffs per company:
```sql
SELECT company, MAX(total_laid_off) AS max_laid_off, MIN(total_laid_off) AS min_laid_off
FROM temp_layoffs
GROUP BY company;
```

- Total layoffs per company:
```sql
SELECT company, SUM(total_laid_off) AS total_laid_off, SUM(percentage_laid_off) AS total_percentage
FROM temp_layoffs
GROUP BY company;
```

- Layoffs by industry:
```sql
SELECT industry, SUM(total_laid_off) AS total_laid_off
FROM temp_layoffs
GROUP BY industry
ORDER BY 2 DESC;
```

- Layoffs by country:
```sql
SELECT country, SUM(total_laid_off) AS total_laid_off
FROM temp_layoffs
GROUP BY country
ORDER BY 2 DESC;
```

- Layoffs by year:
```sql
SELECT YEAR(date) AS year, SUM(total_laid_off) AS total_laid_off
FROM temp_layoffs
GROUP BY YEAR(date)
ORDER BY 1 DESC;
```

- Monthly layoffs trend:
```sql
SELECT FORMAT(date, 'yyyy-MM') AS month, SUM(total_laid_off) AS total_laid_off
FROM temp_layoffs
GROUP BY FORMAT(date, 'yyyy-MM')
ORDER BY month;
```

---

## 📊 Results
- Data was successfully cleaned and standardized.
- Missing/null values were handled properly.
- Final dataset is ready for visualization (e.g., in Power BI or Tableau).
- Analysis revealed trends in layoffs across **companies, industries, countries, and years**.

---

## 🛠️ Tech Stack
- SQL Server  
- T-SQL (DDL, DML, CTEs, Aggregations)

---

## 🚀 Next Steps
- Export cleaned dataset to **Power BI** for visualization.
- Add more advanced analytics (rolling averages, YOY trends).
- Automate cleaning process with stored procedures.

---
