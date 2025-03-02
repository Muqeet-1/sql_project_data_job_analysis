# 📊 Data Analyst Job Market Insights

# Introduction
This project provides a data-driven analysis of the job market for Data Analysts. We explore the most in-demand skills, the highest-paying job skills, and the optimal skills to learn based on demand and salary trends. The analysis leverages SQL queries and visualizations to present clear insights.

SQL queries? check them out here: [project_sql folder](/project_sql)

# Background

With the increasing demand for Data Analysts, understanding the most valuable skills can help individuals optimize their learning paths. This project explores job market trends by analyzing salaries, required skills, and job availability. 💡

### Questions Addressed in This Project:

- What are the most in-demand skills for Data Analysts? 🔥

- Which skills should I learn to maximize my career opportunities? 🎯

- What are the highest-paying job skills for Data Analysts? 💰

- Which Data Analyst jobs offer the highest salaries? 🏆

- Which skills contribute to the highest salaries in the field? 💵

# Tools & Technologies Used
- **SQL** : Used for querying and extracting insights from large datasets related to job postings and skills demand.
- **PostgreSQL 🛢️**: he relational database management system used to store and manage job market data efficiently.
- **VS Code 💻**: The integrated development environment (IDE) where all SQL queries and scripts were written and tested.
- **Git & GitHub**: Used for version control, tracking project changes, and collaborating effectively.
- **Matplotlib & Seaborn 📊**: Powerful Python libraries used to create insightful visualizations that represent job market trends.
---

# 📈 Key Insights
### 🔥 1. Most In-Demand Skills for Data Analysts
We identified the top 10 skills required for Data Analysts by counting the number of job postings requiring each skill.

```sql
SELECT 
    skills_dim.skills, 
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
left JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
where 
    job_title_short = 'Data Analyst'
GROUP BY 
    skills_dim.skills
ORDER BY 
    demand_count DESC
LIMIT 10;

```


**Top Paying Skills:**

![Top Paying Skills](Assets\top_paying_roles.png)
*Here is a bar chart generated by ChatGPT, visualizing the top 10 most in-demand skills for Data Analysts based on job postings data. The graph clearly shows that SQL is the most sought-after skill, followed by Excel and Python. This analysis provides valuable insights for aspiring Data Analysts to focus on the most relevant skills in the job market. 🚀📊*

### 🎯 2. Most Optimal Skills to Learn
Skills ranked based on demand and salary to determine the most beneficial ones to acquire.


``` sql
WITH top_paying_skill AS (
    SELECT 
        skills_dim.skill_id,
        skills_dim.skills, 
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    inner JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE job_title_short = 'Data Analyst' AND salary_year_avg IS NOT NULL
    GROUP BY skills_dim.skill_id, skills_dim.skills
), 
Average_salary AS (
    SELECT 
        skills_dim.skill_id,
        skills_dim.skills, 
        ROUND(AVG(salary_year_avg), 0) AS high_average_salary
    FROM job_postings_fact
    inner JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE job_title_short = 'Data Analyst' AND salary_year_avg IS NOT NULL
    GROUP BY skills_dim.skill_id, skills_dim.skills
), 
Ranked_skills AS (
    SELECT 
        top_paying_skill.skill_id,
        top_paying_skill.skills,
        demand_count,
        high_average_salary,
        RANK() OVER (ORDER BY high_average_salary DESC) AS salary_rank,
        RANK() OVER (ORDER BY demand_count DESC) AS demand_rank
    FROM top_paying_skill
    INNER JOIN Average_salary ON top_paying_skill.skill_id = Average_salary.skill_id
)

SELECT 
    skill_id,
    skills,
    demand_count,
    high_average_salary,
    (salary_rank + demand_rank) AS optimal_score
FROM Ranked_skills
ORDER BY demand_count DESC;

```

### Optimal skills for Data Analysts:

| #  | Skill       | Demand Count | Avg. Salary (USD) | Salary Rank | Demand Rank | Optimal Score |
|----|------------|--------------|-------------------|-------------|-------------|---------------|
| 1  | Spark      | 187          | $113,002         | 33          | 19          | 52            |
| 2  | Snowflake  | 241          | $111,578         | 39          | 17          | 56            |
| 3  | Databricks | 102          | $112,881         | 35          | 32          | 67            |
| 4  | Airflow    | 71           | $116,387         | 24          | 44          | 68            |
| 5  | Kafka      | 40           | $129,999         | 12          | 57          | 69            |
| 6  | Hadoop     | 140          | $110,888         | 43          | 27          | 70            |
| 7  | GCP        | 78           | $113,065         | 32          | 40          | 72            |
| 8  | Confluence | 62           | $114,153         | 27          | 48          | 75            |
| 9  | Scala      | 59           | $115,480         | 25          | 50          | 75            |
| 10 | AWS        | 291          | $106,440         | 63          | 13          | 76            |
| 11 | Linux      | 58           | $114,883         | 26          | 51          | 77            |
| 12 | Azure      | 319          | $105,400         | 66          | 12          | 78            |
| 13 | Jira       | 145          | $107,931         | 53          | 26          | 79            |
| 14 | Pandas     | 90           | $110,767         | 44          | 36          | 80            |
| 15 | Git        | 74           | $112,250         | 37          | 43          | 80            |

### 📌 Interpretation:
- **Spark, Snowflake, and Databricks** are highly valuable for Data Analysts due to **strong salary and demand rankings**.
- **Cloud & Big Data skills (AWS, Azure, GCP, Hadoop)** remain in demand but vary in salary.
- **Airflow and Kafka** offer **some of the highest salaries**, making them great skills for those focusing on data engineering.


---

### 💰 3. Top Paying Job Skills
I identified the skills associated with the highest-paying Data Analyst roles.

```sql

with top_paying_jobs AS (
    SELECT 
        job_postings_fact.job_id,
        job_postings_fact.job_title,
        company_dim.name AS company_name,
        job_postings_fact.salary_year_avg
    FROM 
        job_postings_fact
    left join company_dim ON job_postings_fact.company_id = company_dim.company_id
    where 
        job_title_short = 'Data Analyst' AND
        salary_year_avg is not NULL AND
        job_location = 'Anywhere'
    order by 
        salary_year_avg DESC
    limit 10
)




select 
    top_paying_jobs.*,
    skills
from top_paying_jobs
left JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC




```
### Top Paying Skills for Data Analysts:


| Job Title                                | Company                                      | Salary (USD) | Skill       |
|------------------------------------------|----------------------------------------------|-------------|------------|
| Associate Director- Data Insights        | AT&T                                         | 255829.5   | SQL        |
| Associate Director- Data Insights        | AT&T                                         | 255829.5   | Pandas     |
| Associate Director- Data Insights        | AT&T                                         | 255829.5   | Python     |
| Associate Director- Data Insights        | AT&T                                         | 255829.5   | PowerPoint |
| Associate Director- Data Insights        | AT&T                                         | 255829.5   | Excel      |
| Principal Data Analyst                   | SmartAsset                                   | 186000.0   | Tableau    |
| Principal Data Analyst                   | SmartAsset                                   | 186000.0   | GitLab     |
| ERM Data Analyst                         | Get It Recruit - Information Technology      | 184000.0   | SQL        |
| ERM Data Analyst                         | Get It Recruit - Information Technology      | 184000.0   | Python     |
| ERM Data Analyst                         | Get It Recruit - Information Technology      | 184000.0   | R          |


---

### 💼 4. Top Paying Jobs for Data Analysts
I extracted job postings that offer the highest salaries for Data Analysts.

```sql
SELECT 
    job_postings_fact.job_id,
    job_postings_fact.job_title,
    company_dim.name AS company_name,
    job_postings_fact.job_location,
    job_postings_fact.job_schedule_type,
    job_postings_fact.salary_year_avg,
    job_postings_fact.job_posted_date
FROM 
    job_postings_fact
left join company_dim ON job_postings_fact.company_id = company_dim.company_id
where 
    job_title_short = 'Data Analyst' AND
    salary_year_avg is not NULL AND
    job_location = 'Anywhere'
order by 
    salary_year_avg DESC
limit 10;
```

**Top Paying Jobs:**
![Top Paying Jobs](Assets\job.png)

---
### 💼 5. Top Skills Based on salary
I extracted the top 20 highest-paying skills for Data Analysts using SQL by analyzing job postings that include salary information. The data was gathered by joining job postings with skill records, filtering for "Data Analyst" roles, and calculating the average salary for each skill.

```sql
SELECT 
    skills_dim.skills, 
    round(AVG(salary_year_avg), 0) AS Average_salary
FROM job_postings_fact
left JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
where 
    job_title_short = 'Data Analyst' AND salary_year_avg is not NULL
GROUP BY 
    skills_dim.skills
order BY
    Average_salary DESC
LIMIT 20; 
```

**Top Paying Jobs:**
![Top Paying Jobs](Assets\download.png)



## 🚀 Conclusion
- **SQL, Python, and Excel** are the most in-demand skills.
- **Machine Learning, Big Data, and Python** offer the highest salaries.
- Learning **SQL, Python, and Tableau** is optimal based on demand and salary.

This project helps aspiring Data Analysts make informed decisions about skill development.
