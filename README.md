#  Rakuten Viki TV Dramas & Movies — Analytics & Content Strategy Dashboard

   <div align="center">

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Python](https://img.shields.io/badge/Python-0A97F5?style=for-the-badge&logo=python&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-1F6FEB?style=for-the-badge&logo=powerbi&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-0088CC?style=for-the-badge)
![Data%20Analytics](https://img.shields.io/badge/Data%20Analytics-00AEEF?style=for-the-badge)

</div>


<p align="center">
  <!-- Replace this with your real banner or dashboard image -->
  <img src="images/viki_dashboard_banner.png" width="850" alt="Viki Analytics Dashboard"/>
</p>

<p align="center">
  <b>Data-Driven Insights for Asian Drama Content Strategy</b><br>
  <i>Built with PostgreSQL, Python, and Power BI</i>
</p>

---

## 📋 Table of Contents

- [🌟 Project Overview](#-project-overview)
- [💡 Business Problem](#-business-problem)
- [🎯 Project Objectives](#-project-objectives)
- [🛠️ Tech Stack](#-tech-stack)
- [🚀 Approach](#-approach)
- [📊 Dashboard](#-dashboard)
- [❓ Key Business Questions & SQL Answers](#-key-business-questions--sql-answers)
- [📈 Results & Impact](#-results--impact)
- [🔮 Future Enhancements](#-future-enhancements)

---

## 🌟 Project Overview

Rakuten Viki is a global streaming platform for Asian dramas and movies.  
This project analyzes the **Rakuten Viki TV Dramas & Movies** catalog to uncover:

- Content growth patterns over time  
- Which countries dominate production  
- Which genres, certifications, runtimes, actors, and directors define success  
- Ratings and popularity trends (IMDb & TMDB)  
- Strategic recommendations for content licensing and expansion  

This is a **full end-to-end analytics project** using **PostgreSQL, Python, and Power BI**.

---

## 💡 Business Problem

> **How should Rakuten Viki prioritize licensing and original content investments across countries, genres, and formats to maximize growth, given limited budget and rapidly shifting audience preferences?**

Viki needs to understand:

- Which regions dominate (Korea, China, Japan, Taiwan, etc.)  
- Which content types & genres are oversaturated vs. under-supplied  
- What drives high IMDb ratings and TMDB popularity  
- Which actors/directors repeatedly appear in successful titles  

This analysis delivers insights that directly support **content acquisition, marketing, and audience growth strategies**.

---

## 🎯 Project Objectives

- Build a clean, trusted analytical dataset from Viki metadata  
- Analyze production trends by **year, country, genre, certification, and runtime**  
- Identify **high-performing titles, actors, and directors**  
- Detect **content gaps** where demand likely exceeds supply  
- Build a **Viki-themed Power BI dashboard** for interactive exploration  

---

## 🛠️ Tech Stack

| Tool            | Purpose                               |
|-----------------|----------------------------------------|
| **PostgreSQL**  | Data storage & SQL analytics           |
| **Python**      | EDA & data transformations             |
| **Power BI**    | Interactive dashboard & visuals        |
| **DAX**         | Custom measures for KPIs & shares      |
| **GitHub**      | Version control & project hosting      |

---

## 🚀 Approach

> Minimal description (focus is on SQL & insights).

1️⃣ **Data Preparation**  
- Loaded `titles` and `credits` CSV files into PostgreSQL  
- Cleaned list-like fields (`genres`, `production`) into normalized rows  
- Standardized country codes, trimmed whitespace  
- Created helper views/tables: `titles_rated`, exploded genre/country queries  

2️⃣ **SQL Analysis Layer**  
- Wrote modular SQL queries with CTEs & DISTINCT counts  
- Generated all KPIs: total titles, shares, trends, ratings, popularity  
- Built country, genre, runtime, and people-based aggregations  

3️⃣ **Power BI Dashboard**  
- Connected to PostgreSQL (Import mode)  
- Designed a **Viki-blue gradient** dashboard  
- Added KPI cards, trend lines, bar charts, and drill-down tables  
- Included slicers for **Country, Genre, Type, Release Year**  

---

## 📊 Dashboard

> 🖼️ **Add your Power BI dashboard screenshots here**

For example:

```markdown
<p align="center">
  <img src="images/viki_overview_dashboard.png" width="900" alt="Viki Overview Dashboard"/>
</p>
---

### 🟦 1. Is Korean content dominating the catalog?

#### ✅ SQL Query

```sql
WITH country_exploded AS (
  SELECT
    id,
    trim(upper(regexp_replace(
      unnest(string_to_array(
        regexp_replace(production, '[\[\]]', '', 'g'),
        ','
      )),
      '''', '', 'g'
    ))) AS country_code
  FROM viki.titles
)
SELECT
  country_code,
  COUNT(DISTINCT id) AS title_count,
  ROUND(
    100.0 * COUNT(DISTINCT id) / NULLIF((SELECT COUNT(*) FROM viki.titles), 0),
    2
  ) AS pct_of_catalog
FROM country_exploded
GROUP BY country_code
ORDER BY title_count DESC;

### 🟦 2. How has overall content production evolved over time?
