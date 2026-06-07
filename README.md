# 📊 Data Job Market Analysis (2023)

## 📌 Project Overview

This project explores the 2023 Data Analyst job market using SQL and PostgreSQL. The analysis focuses on identifying:

* 💰 Highest-paying Data Analyst roles
* 🔥 Most in-demand technical skills
* 📈 Highest-paying skills
* 🎯 Optimal skills that combine both high demand and high salary

The goal is to help aspiring Data Analysts understand which skills provide the greatest career and salary opportunities.

📂 **SQL Queries:** Check the complete analysis inside the [project_sql folder](/project_sql/).

---

# 🎯 Background

The data analytics industry continues to evolve rapidly, with new technologies and skill requirements emerging every year.

Through this project, I wanted to answer several key questions:

* Which Data Analyst jobs pay the most?
* What skills do top-paying jobs require?
* Which skills are most requested by employers?
* Which technical skills offer the best return on investment?

By analyzing real-world job posting data, I aimed to identify the ideal combination of skills that maximize both employability and earning potential.

---

# 🛠️ Tools & Technologies

| Tool             | Purpose                                          |
| ---------------- | ------------------------------------------------ |
| **SQL**          | Data querying, filtering, and analysis           |
| **PostgreSQL**   | Database management and execution of SQL queries |
| **Excel**        | Initial data inspection and validation           |
| **Git & GitHub** | Version control and project management           |
| **Gemini**       | Generation of data visualization charts          |

---

# 📈 Analysis & Findings

## 1️⃣ Top-Paying Data Analyst Jobs

To identify the highest-paying remote Data Analyst positions, I filtered remote jobs with available salary information and ranked them by annual salary.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim
    ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst'
    AND job_location = 'Anywhere'
    AND salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

### Key Insight

Remote Data Analyst positions can offer exceptionally high compensation packages, with salaries reaching:

* **$650,000** at Mantys
* **$336,500** at Meta

This demonstrates the significant earning potential available in specialized data analytics roles.

---

## 2️⃣ Skills Required for Top-Paying Jobs

After identifying the highest-paying positions, I examined the skills employers requested for these roles.

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim
        ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst'
        AND job_location = 'Anywhere'
        AND salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim
    ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim
    ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC;
```

### Key Insight

Even among the highest-paying opportunities, employers consistently prioritize foundational technical skills such as:

* SQL
* Python
* Data Analysis Tools

Strong fundamentals remain critical regardless of salary level.

---

## 3️⃣ Most In-Demand Skills

To determine which skills appear most frequently in remote Data Analyst job postings, I analyzed overall demand across the dataset.

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim
    ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim
    ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5;
```

![Top Demanded Skills](/assets/Top_demanded_skills.png)

### Key Insight

The market clearly favors the "Big Three" skills:

1. SQL
2. Excel
3. Python

These skills form the foundation of most Data Analyst positions.

---

## 4️⃣ Highest-Paying Skills

Next, I analyzed which individual skills are associated with the highest average salaries.

```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim
    ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim
    ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```

![Top Paying Skills](/assets/Highest_paying_skills.png)

### Key Insight

Specialized technologies command premium salaries.

Top-paying skills include:

* PySpark (~$208K average salary)
* Bitbucket
* Couchbase

Professionals with expertise in niche technologies often earn significantly more than those with only general analytics skills.

---

## 5️⃣ Optimal Skills (High Demand + High Pay)

Finally, I combined salary and demand metrics to identify the most valuable skills for career growth.

```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim
        ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim
        ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND job_work_from_home = TRUE
        AND salary_year_avg IS NOT NULL
    GROUP BY
        skills_dim.skill_id
),

average_salary AS (
    SELECT
        skills_job_dim.skill_id,
        ROUND(AVG(salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim
        ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim
        ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_job_dim.skill_id
)

SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM skills_demand
INNER JOIN average_salary
    ON skills_demand.skill_id = average_salary.skill_id
WHERE
    demand_count > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```

![Optimal Skills](/assets/Optimal_skills.png)

### Key Insight

The strongest combination of salary potential and market demand comes from cloud and big-data technologies, including:

* Go
* Hadoop
* Snowflake
* Azure

These skills represent excellent long-term investments for aspiring analysts.

---

# 📚 What I Learned

This project significantly strengthened my SQL and data analysis skills.

### Key Takeaways

* Developed advanced SQL querying techniques.
* Improved proficiency with Common Table Expressions (CTEs).
* Gained experience working with multi-table joins.
* Learned to aggregate and analyze large datasets efficiently.
* Discovered the distinction between foundational and specialized technical skills in the analytics job market.

---

# 🏁 Conclusions

The analysis reveals a clear roadmap for aspiring Data Analysts:

### Foundation Skills (Must-Have)

* SQL
* Excel
* Python

These skills dominate employer demand and are essential for entering the field.

### High-Value Skills (Career Accelerators)

* Snowflake
* Azure
* Hadoop
* PySpark
* Cloud Technologies

While niche skills often unlock the highest salaries, combining them with strong foundational skills creates the best balance of employability, job opportunities, and earning potential.

> **Bottom Line:** Master the fundamentals first, then strategically specialize in cloud and big-data technologies to maximize your career growth and salary potential.
