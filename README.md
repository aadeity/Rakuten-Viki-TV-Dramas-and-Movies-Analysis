🟦 Rakuten Viki TV Dramas & Movies — Analytics & Content Strategy Dashboard
<p align="center"> <img src="(add your Power BI screenshot later)" width="850"/> </p> <p align="center"> <b>Data-Driven Insights for Asian Drama Content Strategy</b><br> <i>Using PostgreSQL, Python, and Power BI</i> </p>
📋 Table of Contents

🌟 Project Overview

💡 Business Problem

🎯 Project Objectives

🛠️ Tech Stack

🚀 Approach

📊 Dashboard

❓ Key Business Questions & SQL Answers

📈 Results & Impact

🔮 Future Enhancements

🌟 Project Overview

Rakuten Viki is a leading streaming platform for Asian dramas and movies.
This project analyzes the entire Viki content catalog to uncover:

Content growth patterns

Which countries dominate production

What genres, runtimes, actors, and directors define success

Ratings & popularity trends

Strategic recommendations for content licensing

This is a full end-to-end analytics project using PostgreSQL + Python + Power BI.

💡 Business Problem

“How should Rakuten Viki prioritize licensing and original content investments across countries, genres, and formats to maximize growth, given limited budget and rapidly shifting audience preferences?”

Viki needs to understand:

Which regions dominate (Korea? China? Japan?)

Which content types & genres are oversaturated or under-supplied

What drives high IMDb ratings and TMDB popularity

Which actors/directors repeatedly contribute to successful titles

This analysis delivers insights that directly support content acquisition, marketing, and audience growth strategies.

🎯 Project Objectives

Build a clean and trusted analytical dataset

Analyze production trends by year, country, genre, certification

Identify high-performing titles and creators

Detect content gaps with high potential

Build a beautiful, interactive Power BI dashboard

🛠️ Tech Stack
Tool	Purpose
PostgreSQL	SQL analytics and data cleaning
Python (Pandas)	Quick EDA and transformations
Power BI	Dashboarding, slicers, KPIs
DAX	Custom measures for ratings & shares
GitHub	Version control & project hosting
🚀 Approach

Minimal explanation (as requested).

1️⃣ Data Preparation

Load CSV → PostgreSQL

Clean lists (genres, production) into exploded rows

Standardize country codes, trim whitespace

Create helper tables/views (titles_rated, exploded genre/country tables)

2️⃣ SQL Analysis Layer

Use optimized CTEs & DISTINCT counts

Generate all KPIs + trend metrics

Compute genre, country, rating, runtime insights

3️⃣ Power BI Dashboard

Viki-themed blue gradient

KPIs, trend lines, bar charts, tables

Slicers (Country, Genre, Year, Type)

Screenshots will be added by you.

📊 Dashboard

💠 Add your dashboard screenshot here
(drop it under this heading)

❓ Key Business Questions & SQL Answers

This section contains all the SQL queries used to answer each business question.
Below every query, you can insert screenshots + your observations.

🟦 1. Is Korean content dominating the catalog?
✅ SQL Query
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
  ROUND(100.0 * COUNT(DISTINCT id) / NULLIF((SELECT COUNT(*) FROM viki.titles),0), 2)
    AS pct_of_catalog
FROM country_exploded
GROUP BY country_code
ORDER BY title_count DESC;


📸 Screenshot Placeholder
Your chart/table here

🟦 2. How has overall content production evolved over time?
✅ SQL Query
SELECT release_year, COUNT(*) AS titles
FROM viki.titles
WHERE release_year IS NOT NULL
GROUP BY release_year
ORDER BY release_year;


📸 Screenshot Placeholder

🟦 3. How do Korean vs Chinese growth trends compare?
✅ SQL Query
WITH country_exploded AS (
  SELECT
    id,
    release_year,
    trim(upper(regexp_replace(
      unnest(string_to_array(
        regexp_replace(production, '[\[\]]','', 'g'),
        ','
      )),
      '''','', 'g'
    ))) AS country_code
  FROM viki.titles
  WHERE release_year IS NOT NULL
),
year_totals AS (
  SELECT release_year, COUNT(DISTINCT id) AS total_titles
  FROM viki.titles
  WHERE release_year IS NOT NULL
  GROUP BY release_year
)
SELECT 
  yt.release_year,
  COUNT(DISTINCT CASE WHEN country_code = 'KR' THEN id END) AS korea_titles,
  COUNT(DISTINCT CASE WHEN country_code = 'CN' THEN id END) AS china_titles,
  yt.total_titles,
  ROUND(100.0 * COUNT(DISTINCT CASE WHEN country_code = 'KR' THEN id END)
        / yt.total_titles, 2) AS korea_share_pct
FROM year_totals yt
LEFT JOIN country_exploded ce ON ce.release_year = yt.release_year
GROUP BY yt.release_year, yt.total_titles
ORDER BY yt.release_year;


📸 Screenshot Placeholder

🟦 4. Which countries besides Korea & China matter?
✅ SQL Query
WITH country_exploded AS (
  SELECT trim(upper(regexp_replace(
    unnest(string_to_array(
      regexp_replace(production, '[\[\]]', '', 'g'),
      ','
    )),
    '''', '', 'g'
  ))) AS country_code
  FROM viki.titles
)
SELECT country_code, COUNT(*) AS title_count
FROM country_exploded
WHERE country_code NOT IN ('KR','CN')
GROUP BY country_code
ORDER BY title_count DESC
LIMIT 10;


📸 Screenshot Placeholder

🟦 5. Which genres dominate the platform?
✅ SQL Query
WITH genre_exploded AS (
  SELECT
    trim(unnest(string_to_array(
      regexp_replace(regexp_replace(genres, '\[|\]', '', 'g'),
      '''',''), ','
    ))) AS genre
  FROM viki.titles
)
SELECT genre, COUNT(*) AS title_count
FROM genre_exploded
GROUP BY genre
ORDER BY title_count DESC
LIMIT 20;


📸 Screenshot Placeholder

🟦 6. How is content positioned by age certification?
✅ SQL Query
SELECT age_certification, COUNT(*) AS title_count
FROM viki.titles
GROUP BY age_certification
ORDER BY title_count DESC;


📸 Screenshot Placeholder

🟦 7. What does runtime distribution reveal about format strategy?
✅ SQL Query
SELECT runtime, COUNT(*) AS title_count
FROM viki.titles
WHERE runtime IS NOT NULL
GROUP BY runtime
ORDER BY runtime;

(Optional Bucketed Runtime)
SELECT 
  type,
  CASE 
    WHEN runtime < 30 THEN '<30 min'
    WHEN runtime BETWEEN 30 AND 59 THEN '30–59 min'
    WHEN runtime BETWEEN 60 AND 89 THEN '60–89 min'
    WHEN runtime >= 90 THEN '90+ min'
  END AS runtime_bucket,
  COUNT(*) AS title_count
FROM viki.titles
GROUP BY type, runtime_bucket
ORDER BY type, runtime_bucket;


📸 Screenshot Placeholder

🟦 8. Which titles perform best by IMDb and TMDB?
IMDb
SELECT title, release_year, production, imdb_score
FROM viki.titles
WHERE imdb_score IS NOT NULL
ORDER BY imdb_score DESC NULLS LAST, imdb_votes DESC NULLS LAST
LIMIT 20;

TMDB
SELECT title, release_year, production, tmdb_popu, tmdb_score
FROM viki.titles
WHERE tmdb_popu IS NOT NULL
ORDER BY tmdb_popu DESC NULLS LAST, tmdb_score DESC NULLS LAST
LIMIT 20;


📸 Screenshot Placeholder

🟦 9. Do certain actors/directors repeatedly appear in successful titles?
Actors
WITH top_titles AS (
  SELECT id
  FROM viki.titles
  WHERE imdb_score IS NOT NULL
  ORDER BY imdb_score DESC
  LIMIT 200
)
SELECT c.name, COUNT(*) AS appearances_in_top_titles
FROM viki.credits c
JOIN top_titles t ON c.id = t.id
WHERE c.role = 'ACTOR'
GROUP BY c.name
ORDER BY appearances_in_top_titles DESC
LIMIT 20;

Directors
WITH top_titles AS (
  SELECT id
  FROM viki.titles
  WHERE imdb_score IS NOT NULL
  ORDER BY imdb_score DESC
  LIMIT 200
)
SELECT c.name, COUNT(*) AS directed_top_titles
FROM viki.credits c
JOIN top_titles t ON c.id = t.id
WHERE c.role = 'DIRECTOR'
GROUP BY c.name
ORDER BY directed_top_titles DESC
LIMIT 20;


📸 Screenshot Placeholder

🟦 10. Are there content gaps where demand exceeds supply?
Genre Gap Analysis
WITH genre_exploded AS (
  SELECT
    t.id,
    trim(unnest(string_to_array(
      regexp_replace(regexp_replace(t.genres, '\[|\]', '', 'g'), '''', ''),
      ','
    ))) AS genre,
    t.imdb_score
  FROM viki.titles t
)
SELECT
  genre,
  COUNT(DISTINCT id) AS title_count,
  ROUND(AVG(imdb_score) FILTER (WHERE imdb_score IS NOT NULL), 2) AS avg_imdb_score
FROM genre_exploded
GROUP BY genre
HAVING COUNT(DISTINCT id) >= 5
ORDER BY avg_imdb_score DESC;


📸 Screenshot Placeholder

🟦 11. How do ratings vary by country and genre?
Country Ratings
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
  ROUND(AVG(imdb_score) FILTER (WHERE imdb_score IS NOT NULL), 2) AS avg_imdb_score
FROM country_exploded
GROUP BY country_code
ORDER BY avg_imdb_score DESC NULLS LAST;

Genre Ratings
WITH genre_exploded AS (
  SELECT
    t.id,
    trim(unnest(string_to_array(
      regexp_replace(regexp_replace(t.genres, '\[|\]', '', 'g'), '''',''),
      ','
    ))) AS genre,
    t.imdb_score
  FROM viki.titles t
)
SELECT
  genre,
  COUNT(DISTINCT id) AS rated_titles,
  ROUND(AVG(imdb_score) FILTER (WHERE imdb_score IS NOT NULL), 2) AS avg_imdb_score
FROM genre_exploded
GROUP BY genre
ORDER BY avg_imdb_score DESC NULLS LAST;


📸 Screenshot Placeholder

📈 Results & Impact

This analysis supports:

Smarter content licensing (which countries/genres to prioritize)

Better marketing focus (promoting high-rated/popular titles)

Improved user engagement (focusing on genres users love)

Discovery of content gaps (opportunities to expand library)

🔮 Future Enhancements

Watch-time & engagement analytics

Content recommendation engine

Cost vs. popularity ROI modeling

Episode-level analytics

Forecasting content trends
