# HR-Analytics-with-MySQL

### Project Overview
An end-to-end SQL analysis of employee data: cleaning a raw HR table, engineering an age field, and answering workforce questions about demographics, turnover, tenure, and headcount trends.

### Tools
  * MySQL
  * SQL concepts : UPDATE, ALTER TABLE, CASE, date functions, Aggregation, Subqueries

### Dataset Overview
A single table, hr_staging, containing employee records with these fields used in the analysis: birthdate, hire_date, termdate, gender, race, department, jobtitle, location, location_city, location_state.

### Data Cleaning
|Step             |What I did                                                                                                                                                                                                                      |
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|Date formats     |`birthdate` and `hire_date` were stored as text with mixed separators (`/` and `-`). Used `CASE` + `STR_TO_DATE` + `DATE_FORMAT` to standardize to `YYYY-MM-DD`, then `ALTER TABLE ... MODIFY COLUMN` to convert them to `DATE`.|
|Termination dates|`termdate` held timestamps with a UTC suffix. Parsed with `STR_TO_DATE(termdate, '%Y-%m-%d %H:%i:%s UTC')`.                                                                                                                     |
|Missing values   |Blank/NULL `termdate` set to `'0000-00-00'` to mark **currently active** employees; enabled `ALLOW_INVALID_DATES` in `sql_mode` so the column could be converted to `DATE`.                                                     |
|New column       |Added `age` with `ALTER TABLE`, populated using `TIMESTAMPDIFF(YEAR, birthdate, CURDATE())`.                                                                                                                                    |

### Business Questions

All analysis queries filter on age >= 18. Current employees are identified by termdate = '0000-00-00'.

	1.	Gender breakdown of current employees
  
	2.	Race/ethnicity breakdown of current employees

	3.	Age range (youngest and oldest employee)
  
	4.	Age group distribution (18-24, 25-34, 35-44, 45-54, 55-64, 65+) using CASE
  
	5.	Gender distribution within each age group
  
	6.	Headquarters vs. remote: employee count by location
  
	7.	Average length of employment for terminated employees
  
	8.	Gender distribution by department
  
	9.	Job title distribution across the company
  
	10.	Department with the highest turnover rate: termination rate = terminated / total, built with a subquery
  
	11.	Employee distribution by city and state
  
	12.	Employee distribution by state
  
	13.	Headcount change over time: hires, terminations, net change, and net change % per year
  
	14.	Average tenure by department for terminated employees

  Example: Turnover Rate by department 

  SELECT department,
       total_count,
       terminated_count,
       terminated_count / total_count AS termination_rate
FROM (
    SELECT department,
           COUNT(*) AS total_count,
           SUM(CASE WHEN termdate <> '0000-00-00' AND termdate <= CURDATE() THEN 1 ELSE 0 END) AS terminated_count
    FROM hr_staging
    WHERE age >= 18
    GROUP BY department
) AS subquery
ORDER BY termination_rate DESC;

Age group with CASE:

SELECT
    CASE
        WHEN age BETWEEN 18 AND 24 THEN '18-24'
        WHEN age BETWEEN 25 AND 34 THEN '25-34'
        WHEN age BETWEEN 35 AND 44 THEN '35-44'
        WHEN age BETWEEN 45 AND 54 THEN '45-54'
        WHEN age BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END AS age_group,
    COUNT(*) AS count
FROM hr_staging
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY age_group
ORDER BY age_group;

<img width="712" height="672" alt="image" src="https://github.com/user-attachments/assets/1cde62f5-e87c-47ad-944e-2ba784a78962" />

<img width="456" height="664" alt="image" src="https://github.com/user-attachments/assets/df17d5e3-a25f-46f0-b9fe-86fdbcd93760" />

<img width="592" height="676" alt="image" src="https://github.com/user-attachments/assets/af550fdf-38e7-419f-8e73-16ae9aef90ef" />

<img width="620" height="676" alt="image" src="https://github.com/user-attachments/assets/f4d9e50c-437d-4922-98b4-8f7c9a047749" />

<img width="680" height="388" alt="image" src="https://github.com/user-attachments/assets/0b559575-3df9-4af6-b000-c4b35be003a4" />


  ### Key Findings

  •	Gender split

	•	Largest age group
  
	•	Department with highest turnover
  
	•	Average tenure of terminated employees
  
	•	Year with the highest net hiring growth

### Skills Demonstrated

	•	Cleaning inconsistent date data and changing column types safely
  
	•	Feature engineering (age, tenure) with date functions
  
	•	Conditional logic with CASE
  
	•	Aggregations, GROUP BY, and ordering
  
	•	Subqueries for rate calculations
  
	•	Translating business questions into SQL


