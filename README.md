## HR Analysis Report 

### Project Overview 

This is an HR Analysis Report done using SSMS for data cleaning and querying, and Power BI for visualization. 
The objective was to analyze key performance indicators such as average employee tenure, turnover rates, and employee distribution.

### Data Sources 
The primary data source was a csv file `hr_data`.

### Tools Used 
- SSMS – Data Cleaning and Preparation
- Power BI – Data Visualization 

### Data Cleaning and Preparation
In the initial stages of data cleaning and preparation, the following tasks were performed:
- Data loading and inspection in SSMS.
- Handling missing values.
- Data transformation.

### Exploratory Data Analysis
These questions were explored in detail:
1. What is the average employee tenure?
2. What is the turnover rate across different cadres?
3. What is the distribution of employees in relation to gender, race, state, and remote vs onsite employees?

SQL Codes Used;
```sql
CREATE DATABASE hr_analytics;

USE hr_analytics;

SELECT TOP 100 * FROM dbo.hr_data;

-- DATA CLEANING/TRANSFORMATION
-- convert termdate column from nvarchar to date
UPDATE hr_data
SET termdate = FORMAT(CONVERT(DATETIME, LEFT(termdate,19),120), 'yyyy-MM-dd');

-- create new_termdate and copy data from termdate
ALTER TABLE hr_data
ADD new_termdate DATE;

UPDATE hr_data
SET new_termdate = CASE WHEN termdate IS NOT NULL AND ISDATE(termdate) = 1 
                    THEN CAST(termdate AS DATETIME) ELSE NULL END;

-- create the age column
ALTER TABLE hr_data
ADD age nvarchar(50);

UPDATE hr_data
SET age = DATEDIFF(YEAR,birthdate,GETDATE());

-- QUESTIONS TO ANSWER FROM THE DATA

-- 1) What's the age distribution in the company?
-- oldest vs youngest employee
SELECT MIN(age) AS youngest_employee,
       MAX(age) AS oldest_employee
FROM hr_data;

SELECT TOP 100 * FROM dbo.hr_data;

-- age group distribution
SELECT age_group, COUNT(*) AS total_count
FROM(
   SELECT CASE
      WHEN age >= 20 AND age <= 30 THEN '20-30'
	  WHEN age >= 30 AND age <= 40 THEN '30-40'
	  WHEN age >= 40 AND age <= 50 THEN '40-50'
	  ELSE '50+'
	  END AS age_group
 FROM hr_data
 WHERE new_termdate IS NULL) AS age_groups
 GROUP BY age_group
 ORDER BY age_group;

 -- age group distribution by gender
SELECT age_group, gender, COUNT(*) AS total_count
FROM(
   SELECT gender,
   CASE
      WHEN age >= 20 AND age <= 30 THEN '20-30'
	  WHEN age >= 30 AND age <= 40 THEN '30-40'
	  WHEN age >= 40 AND age <= 50 THEN '40-50'
	  ELSE '50+'
	  END AS age_group
 FROM hr_data
 WHERE new_termdate IS NULL) AS age_groups
 GROUP BY age_group, gender
 ORDER BY age_group;

 -- 2) What's the gender breakdown in the company?
SELECT gender,
      COUNT(*) AS count
FROM hr_data
WHERE new_termdate IS NULL
GROUP BY gender
ORDER BY count DESC;

-- 3) How does gender vary across departments and job titles?
SELECT gender, department, jobtitle, COUNT(*) AS gender_variation
FROM hr_data
WHERE new_termdate IS NULL
GROUP BY gender, department, jobtitle
ORDER BY gender_variation DESC;

-- 4) What's the race distribution in the company?
SELECT race, COUNT(*) AS racial_distribution
FROM hr_data
WHERE new_termdate IS NULL
GROUP BY race 
ORDER BY racial_distribution DESC;

-- 5) What's the average length of employment in the company?
SELECT
 AVG(DATEDIFF(year, hire_date, new_termdate)) AS avg_tenure_yrs
 FROM hr_data
 WHERE new_termdate IS NOT NULL AND new_termdate <= GETDATE();

-- 6) Which department has the highest turnover rate?
-- get total count
-- get terminated count
-- terminated count/total count
USE hr_analytics;

WITH departmental_count AS
(
SELECT department, COUNT(*) AS total_count,
    SUM(CASE WHEN new_termdate IS NOT NULL AND new_termdate <= GETDATE() THEN 1 ELSE 0 END) AS terminated_count
FROM hr_data
WHERE new_termdate IS NOT NULL
GROUP BY department
)
SELECT department,
      total_count,
	  terminated_count,
      CONCAT((ROUND(CAST(terminated_count AS FLOAT)/total_count,2))*100,'%') AS turnover_rate
FROM departmental_count
ORDER BY turnover_rate DESC;


-- 7) What is the average tenure distribution for each department?

SELECT department,
       AVG(DATEDIFF(year, hire_date, new_termdate)) AS tenure
 FROM hr_data
 WHERE new_termdate IS NOT NULL AND new_termdate <= GETDATE()
 GROUP BY department
 ORDER BY tenure DESC;

SELECT 
 department, DATEDIFF(year, MIN(hire_date), MAX(new_termdate)) AS tenure
FROM hr_data
WHERE  new_termdate IS NOT NULL AND new_termdate <= GETDATE()
GROUP BY department
ORDER BY tenure DESC;

-- 8) How many employees work remotely for each department?
SELECT department, COUNT(*) AS employees_count
FROM hr_data
WHERE  new_termdate IS NULL 
   AND location LIKE 'Remote%'
GROUP BY department
ORDER BY employees_count DESC;

-- 9) What's the distribution of employees across different states?
SELECT location_state, COUNT(*) AS employees_count
FROM hr_data
WHERE new_termdate IS NULL 
GROUP BY location_state
ORDER BY employees_count DESC;

-- 10) How are job titles distributed in the company?
SELECT 
 jobtitle,
 COUNT(*) AS title_count
FROM hr_data
WHERE new_termdate IS NULL
GROUP BY jobtitle
ORDER BY title_count DESC;

-- 11) How have employee hire counts varied over time?
SELECT TOP 10 * FROM hr_data;

WITH yearly_records AS
(
SELECT YEAR(hire_date) AS hire_year, 
      COUNT(*) AS yearly_hires,
	  SUM(CASE WHEN new_termdate IS NOT NULL AND new_termdate <= GETDATE() THEN 1 ELSE 0 END)
	  AS yearly_terminations
FROM hr_data
GROUP BY YEAR(hire_date)
)
SELECT hire_year,
       yearly_hires,
	   yearly_terminations,
	   yearly_hires - yearly_terminations AS remaining_employees,
	   CONCAT((ROUND(CAST((yearly_hires - yearly_terminations) AS FLOAT)/NULLIF(yearly_hires,0),2))*100,'%')
	   AS remaining_percent
FROM yearly_records;
```

### Data Analysis Results
- The average tenure is 7 years.
- There has been a persistent increase in the new hire percentage across the years covered in the project (2000-2020).
- The turnover rate was highest (15%) in the Auditing department and lowest (9%) in the Business Development and Marketing departments.
- There are more onsite employees(75%) than remote employees (25%). 
- The gender distribution is as follows: 51% male, 46% female, and 3% non-conforming.

![HR report png1](https://github.com/user-attachments/assets/a061f702-3e2e-439b-9504-f8d367944063)

![HR report png2](https://github.com/user-attachments/assets/ad612163-e124-4e78-801b-fc7cac1a8870)


### Recommendation
Investigate why the auditing and legal departments register higher turnover rates.



