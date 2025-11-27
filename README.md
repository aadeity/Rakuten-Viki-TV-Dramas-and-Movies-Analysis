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
  <img src="images/viki_dashboard_banner.png" width="550" alt="Viki Analytics Dashboard"/>
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

## 📊 Dashboard

> 🖼️ **Add your Power BI dashboard screenshots here**

For example:

<p align="center">
  <img src="images/viki_overview_dashboard.png" width="900" alt="Viki Overview Dashboard"/>
</p>

---

### 🟦 1. Is Korean content dominating the catalog?

#### SQL Query

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
```
# Results:
<img width="550"  alt="image" src="https://github.com/user-attachments/assets/62ecbcc4-bb51-4eca-af3e-20f7ec02d559" />

### 🟦 2. How has overall content production evolved over time?

#### SQL Query

```sql
SELECT
  release_year,
  COUNT(*) AS titles
FROM viki.titles
WHERE release_year IS NOT NULL
GROUP BY release_year
ORDER BY release_year;
```
# Results:
<img width="550" alt="image" src="https://github.com/user-attachments/assets/7d60bba9-2f17-414e-8e80-ee763da5c3f3" />

### 🟦 3. How do Korean vs Chinese growth trends compare?

#### SQL Query

```sql
WITH country_exploded AS (
  SELECT
    id,
    release_year,
    trim(upper(regexp_replace(
      unnest(string_to_array(
        regexp_replace(production, '[\[\]]', '', 'g'),
        ','
      )),
      '''', '', 'g'
    ))) AS country_code
  FROM viki.titles
  WHERE release_year IS NOT NULL
),
year_totals AS (
  SELECT
    release_year,
    COUNT(DISTINCT id) AS total_titles
  FROM viki.titles
  WHERE release_year IS NOT NULL
  GROUP BY release_year
)
SELECT 
  yt.release_year,
  COUNT(DISTINCT CASE WHEN country_code = 'KR' THEN id END) AS korea_titles,
  COUNT(DISTINCT CASE WHEN country_code = 'CN' THEN id END) AS china_titles,
  yt.total_titles,
  ROUND(
    100.0 * COUNT(DISTINCT CASE WHEN country_code = 'KR' THEN id END)
      / yt.total_titles,
    2
  ) AS korea_share_pct
FROM year_totals yt
LEFT JOIN country_exploded ce
  ON ce.release_year = yt.release_year
GROUP BY yt.release_year, yt.total_titles
ORDER BY yt.release_year;
```

### 🟦4. Which countries besides Korea & China matter?
#### SQL Query

```sql
WITH country_exploded AS (
  SELECT
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
  COUNT(*) AS title_count
FROM country_exploded
WHERE country_code NOT IN ('KR', 'CN')
GROUP BY country_code
ORDER BY title_count DESC
LIMIT 10;
```

### 🟦5. Which genres dominate the platform?
#### SQL Query

```sql
WITH genre_exploded AS (
  SELECT
    trim(unnest(string_to_array(
      regexp_replace(
        regexp_replace(genres, '\[|\]', '', 'g'),
        '''',
        ''
      ),
      ','
    ))) AS genre
  FROM viki.titles
)
SELECT
  genre,
  COUNT(*) AS title_count
FROM genre_exploded
GROUP BY genre
ORDER BY title_count DESC
LIMIT 20;
```
### 🟦6. How is content positioned by age certification?
#### SQL Query

```sql
SELECT
  age_certification,
  COUNT(*) AS title_count
FROM viki.titles
GROUP BY age_certification
ORDER BY title_count DESC;
```

### 🟦7. What does runtime distribution reveal about format strategy?
#### SQL Query

```sql
SELECT 
  type,
  CASE 
    WHEN runtime < 30 THEN '<30 min'
    WHEN runtime BETWEEN 30 AND 59 THEN '30–59 min'
    WHEN runtime BETWEEN 60 AND 89 THEN '60–89 min'
    WHEN runtime >= 90 THEN '90+ min'
    ELSE 'Unknown'
  END AS runtime_bucket,
  COUNT(*) AS title_count
FROM viki.titles
GROUP BY type, runtime_bucket
ORDER BY type, runtime_bucket;

```


### 🟦8. Which titles perform best by IMDb and TMDB metrics?
#### SQL Query-– Top-Rated (IMDb)

```sql
SELECT
  title,
  release_year,
  production,
  imdb_score,
  imdb_votes
FROM viki.titles
WHERE imdb_score IS NOT NULL
ORDER BY imdb_score DESC NULLS LAST, imdb_votes DESC NULLS LAST
LIMIT 20;
```

#### SQL Query--Most Popular (TMDB)
```sql
SELECT
  title,
  release_year,
  production,
  tmdb_popu,
  tmdb_score
FROM viki.titles
WHERE tmdb_popu IS NOT NULL
ORDER BY tmdb_popu DESC NULLS LAST, tmdb_score DESC NULLS LAST
LIMIT 20;
```


### 🟦9. Do certain actors repeatedly appear in successful titles?
#### SQL Query


```sql
WITH top_titles AS (
  SELECT id
  FROM viki.titles
  WHERE imdb_score IS NOT NULL
  ORDER BY imdb_score DESC
  LIMIT 200
)
SELECT
  c.name,
  COUNT(*) AS appearances_in_top_titles
FROM viki.credits c
JOIN top_titles t
  ON c.id = t.id
WHERE c.role = 'ACTOR'
GROUP BY c.name
ORDER BY appearances_in_top_titles DESC
LIMIT 20;
```


### 🟦10. Do certain directors repeatedly appear in successful titles?
#### SQL Query

```sql
WITH top_titles AS (
  SELECT id
  FROM viki.titles
  WHERE imdb_score IS NOT NULL
  ORDER BY imdb_score DESC
  LIMIT 200
)
SELECT
  c.name,
  COUNT(*) AS directed_top_titles
FROM viki.credits c
JOIN top_titles t
  ON c.id = t.id
WHERE c.role = 'DIRECTOR'
GROUP BY c.name
ORDER BY directed_top_titles DESC
LIMIT 20;
```


### 🟦11. Are there content gaps where demand exceeds supply?
#### SQL Query

```sql
WITH genre_exploded AS (
  SELECT
    t.id,
    trim(unnest(string_to_array(
      regexp_replace(
        regexp_replace(t.genres, '\[|\]', '', 'g'),
        '''',
        ''
      ),
      ','
    ))) AS genre,
    t.imdb_score
  FROM viki.titles t
)
SELECT
  genre,
  COUNT(DISTINCT id) AS title_count,
  ROUND(
    AVG(imdb_score) FILTER (WHERE imdb_score IS NOT NULL),
    2
  ) AS avg_imdb_score
FROM genre_exploded
GROUP BY genre
HAVING COUNT(DISTINCT id) >= 5
ORDER BY avg_imdb_score DESC;
```


### 🟦12. How do ratings vary by country and genre?
#### SQL Query– Ratings by Country

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
    ))) AS country_code,
    imdb_score
  FROM viki.titles
)
SELECT
  country_code,
  COUNT(DISTINCT id) AS rated_titles,
  ROUND(
    AVG(imdb_score) FILTER (WHERE imdb_score IS NOT NULL),
    2
  ) AS avg_imdb_score
FROM country_exploded
GROUP BY country_code
ORDER BY avg_imdb_score DESC NULLS LAST;
```


#### SQL Query – Ratings by Genre
```sql
WITH genre_exploded AS (
  SELECT
    t.id,
    trim(unnest(string_to_array(
      regexp_replace(
        regexp_replace(t.genres, '\[|\]', '', 'g'),
        '''',
        ''
      ),
      ','
    ))) AS genre,
    t.imdb_score
  FROM viki.titles t
)
SELECT
  genre,
  COUNT(DISTINCT id) AS rated_titles,
  ROUND(
    AVG(imdb_score) FILTER (WHERE imdb_score IS NOT NULL),
    2
  ) AS avg_imdb_score
FROM genre_exploded
GROUP BY genre
ORDER BY avg_imdb_score DESC NULLS LAST;
```
