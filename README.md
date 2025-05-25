
# 🎬 Netflix Movies & TV Shows Data Analysis Using SQL

![Netflix Logo](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

> 📊 A comprehensive SQL-driven analysis of Netflix's global content library — uncovering trends, viewer preferences, and regional insights.

---

## 📚 Table of Contents

- [📌 Overview](#-overview)
- [🎯 Objectives](#-objectives)
- [📁 Dataset](#-dataset)
- [🗂️ Schema](#-schema)
- [💡 Business Problems & Solutions](#-business-problems--solutions)
- [🔍 Findings and Conclusion](#-findings-and-conclusion)

---

## 📌 Overview

This project presents a comprehensive analysis of Netflix’s movies and TV shows dataset using **PostgreSQL**. The primary objective is to extract meaningful insights and answer business-relevant questions related to:

- 📺 Content types  
- 🎭 Genres  
- 📅 Release years  
- 🌍 Popular countries  
- 🎬 Top actors  
- ➕ And more...

---

## 🎯 Objectives

- 📽️ Analyze distribution of content types (Movies vs TV Shows)
- ⭐ Identify most frequent ratings
- 🌍 Explore release trends by year and country
- 🎬 Discover top genres, durations, and cast members
- 🔎 Detect sensitive content and missing metadata

---

## 📁 Dataset

- Source: [Netflix Shows on Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)
  
---

## 🗂️ Schema

```sql
CREATE TABLE netflix (
    show_id      VARCHAR(6),
    type         VARCHAR(10),
    title        VARCHAR(150),
    director     VARCHAR(208),
    casts        VARCHAR(1000),
    country      VARCHAR(150),
    date_added   VARCHAR(50),
    release_year INT,
    rating       VARCHAR(10),
    duration     VARCHAR(15),
    listed_in    VARCHAR(100),
    description  VARCHAR(250)
);
```

| Column        | Description                                 |
|---------------|---------------------------------------------|
| show_id       | Unique identifier                           |
| type          | 'Movie' or 'TV Show'                        |
| title         | Title of the content                        |
| director      | Director(s) of the content                  |
| casts         | Cast members                                |
| country       | Country of production                       |
| date_added    | Date content was added to Netflix           |
| release_year  | Year of content release                     |
| rating        | Age rating (e.g., PG, TV-MA)                |
| duration      | Runtime or number of seasons                |
| listed_in     | Genre/category                              |
| description   | Brief summary                               |


---

## 💡 Business Problems & Solutions

Each of the following problems is solved using SQL with optimized queries and real-world objectives in mind:

### 1. 📊 Distribution of Movies vs TV Shows
```sql
-- Count the number of Movies vs TV Shows

SELECT type,COUNT(*) AS total_content
FROM netflix
GROUP BY type;
```
➡️ **Objective**: Determine the distribution of content types on Netflix. Helps understand platform content strategy.

<img src="https://github.com/user-attachments/assets/469c27bf-45e8-4fc5-9f81-d49ca2700426" alt="Image description" width="400" height="150"/>

---

### 2. ⭐ Most Common Rating by Type
```sql
-- Find the most common rating for movies and TV shows

SELECT type,rating,rating_count
FROM(
SELECT type,rating , COUNT(*) AS rating_count,
RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) AS ranking
FROM netflix
GROUP BY type,rating
)
WHERE ranking = 1;
```
➡️ **Objective**: Identify the most frequently occurring rating for each type of content. Shows what content ratings dominate per type.

<img src="https://github.com/user-attachments/assets/6eff94bd-99d4-4c5f-8583-2de244f76d6d" alt="Image description" width="400" height="150"/>

---

### 3. 🗓️ Movies Released in 2020
```sql
-- List all movies released in a specific year (e.g., 2020)

SELECT *
FROM netflix
WHERE type = 'Movie' and release_year = 2020
```
➡️ **Objective**: Retrieve all movies released in a specific year.

<img src="https://github.com/user-attachments/assets/9571eda9-8ffb-423c-a591-abfea89a800c" alt="Image description" width="800" height="150"/>

---

### 4. 🌍 Top 5 Countries by Content Count
```sql
-- Find the top 5 countries with the most content on Netflix

SELECT UNNEST(STRING_TO_ARRAY(country,',')) as new_country, COUNT(show_id) AS total_content
FROM netflix
GROUP BY UNNEST(STRING_TO_ARRAY(country,','))
ORDER BY total_content DESC
LIMIT 5;

-- NOTE: column country has concat value so to separate the countries i use this -> UNNEST(STRING_TO_ARRAY(country,','))
```
➡️ **Objective**: Identify the top 5 countries with the highest number of content items.

<img src="https://github.com/user-attachments/assets/80d99e6a-1d3c-4faa-8d4a-4dba5eda7a5d" alt="Image description" width="350" height="150"/>

---

### 5. ⏱️ Longest Movie
```sql
-- Identify the longest movie

SELECT *
FROM netflix
WHERE type = 'Movie' AND duration IS NOT NULL
ORDER BY SPLIT_PART(duration,' ',1)::INT DESC
LIMIT 1;

-- NOTE: In PostgreSQL (and some other SQL engines) By default, ORDER BY ... DESC puts NULL values first. Because NULL is considered unknown and not comparable — it doesn't have a defined position unless you tell SQL explicitly.
```
➡️ **Objective**: Find the movie with the longest duration.


<img src="https://github.com/user-attachments/assets/ecbb1422-313d-48cf-ab4b-a2c967c12381" alt="Image description" width="800" height="150"/>

---

### 6. 🆕 Content Added in Last 5 Years
```sql
-- Find content added in the last 5 years

SELECT * 
from netflix
WHERE TO_DATE(date_added,'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 YEARS';
```
➡️ **Objective**: Retrieve content added to Netflix in the last 5 years.


<img src="https://github.com/user-attachments/assets/772048ab-250d-4a51-af8b-6907a1dbf3a7" alt="Image description" width="600" height="150"/>

---

### 7. 🎬 All Shows Directed by Rajiv Chilaka
```sql
-- Find all the movies/TV shows by director 'Rajiv Chilaka'!

SELECT * FROM (
  SELECT *, UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name FROM netflix
) t WHERE director_name = 'Rajiv Chilaka';

-- Another method
SELECT *
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';
```
➡️ **Objective**: List all content directed by 'Rajiv Chilaka'.


<img src="https://github.com/user-attachments/assets/91c88aa6-2994-47d5-821e-976e326c3d7a" alt="Image description" width="400" height="150"/>

---

### 8. 📺 TV Shows with Over 5 Seasons
```sql
-- List all TV shows with more than 5 seasons

SELECT * FROM netflix
WHERE type = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::INT > 5;

-- Another method:
SELECT * FROM netflix
WHERE type = 'TV Show' AND (SPLIT_PART(duration,' ',2) ILIKE '%Seasons%' AND SPLIT_PART(duration,' ',1)::INT > 5);
```
➡️ **Objective**: Identify TV shows with more than 5 seasons.

<img src="https://github.com/user-attachments/assets/958c8218-9113-4891-98a0-11aef3a6a67d" alt="Image description" width="400" height="150"/>

---

### 9. 🎭 Count Content by Genre
```sql
-- Count the number of content items in each genre

SELECT UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, COUNT(*) 
FROM netflix GROUP BY genre;

-- Another method
SELECT UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, COUNT(show_id) 
FROM netflix 
GROUP BY 1;
```
➡️ **Objective**: Count the number of content items in each genre.


<img src="https://github.com/user-attachments/assets/79cace2f-3f15-444c-81cd-a3bcbd0d1a78" alt="Image description" width="400" height="250"/>

---

### 10. 📈 Top 5 Years for Indian Content (Avg%)
```sql
-- Find each year and the average numbers of content release in India on netflix. Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

SELECT EXTRACT(YEAR FROM TO_DATE(date_added,'Month DD, YYYY')) AS YEAR , COUNT(show_id) AS total_content , 
ROUND(COUNT(show_id)*1.0/12,2) AS avg_content_per_month
FROM netflix
WHERE country ILIKE '%India%'
GROUP BY 1
ORDER BY avg_content_per_month DESC
LIMIT 5;

```
➡️ **Objective**: Calculate and rank years by the average number of content releases by India.


<img src="https://github.com/user-attachments/assets/10d3a9cb-bc80-4b2f-85d5-2df06b354216" alt="Image description" width="400" height="250"/>

---

### 11. 📚 All Documentaries
```sql
-- List all movies that are documentaries

SELECT * FROM netflix WHERE listed_in LIKE '%Documentaries';
```
➡️ **Objective**: Retrieve all movies classified as documentaries.


<img src="https://github.com/user-attachments/assets/5d4b2d99-9541-455c-b08b-3d82e61460c0" alt="Image description" width="800" height="250"/>

---

### 12. ❌ Content Without a Director
```sql
-- Find all content without a director

SELECT * FROM netflix WHERE director IS NULL;
```
➡️ **Objective**: List content that does not have a director.


<img src="https://github.com/user-attachments/assets/1a185c45-ad35-494b-909e-029a5a430135" alt="Image description" width="600" height="300"/>

---

### 13. 🎞️ Salman Khan Movies (Last 10 Years)
```sql
-- Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT COUNT(show_id) AS count_of_movie_by_salman
FROM netflix
WHERE (type = 'Movie' AND casts ILIKE '%Salman Khan%') 
AND release_year >= EXTRACT(YEAR FROM CURRENT_DATE - INTERVAL '10 YEARS')
```
➡️ **Objective**: Count the number of movies featuring 'Salman Khan' in the last 10 years.


<img src="https://github.com/user-attachments/assets/e7040747-44ba-46d6-b707-f12c416ef400" alt="Image description" width="400" height="150"/>

---

### 14. 🎭 Top 10 Indian Actors by Appearances
```sql
-- Find the top 10 actors who have appeared in the highest number of movies produced in India.

SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor, COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY COUNT(*) DESC
LIMIT 10;
```
➡️ **Objective**: Identify the top 10 actors with the most appearances in Indian-produced movies.


<img src="https://github.com/user-attachments/assets/8c893067-103a-45b8-ab8e-283edb554fd7" alt="Image description" width="500" height="300"/>

---

### 15. 🚫 Content Categorized by 'Kill' or 'Violence'
```sql
-- Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
-- the description field. Label content containing these keywords as 'Bad' and all other 
-- content as 'Good'. Count how many items fall into each category.

SELECT
CASE
WHEN description ILIKE '%kill%' OR 
	description ILIKE '%violence%' THEN 'bad'
ELSE 'good'
END AS content_categorize,
COUNT(*)
FROM netflix
GROUP BY 1

--Another method
SELECT content_categorize,count(*)
FROM(SELECT
CASE
WHEN description ILIKE '%kill%' OR 
	description ILIKE '%violence%' THEN 'bad'
ELSE 'good'
END AS content_categorize
FROM netflix)
GROUP BY content_categorize

-- we can solve this using CTE also
```
➡️ **Objective**: Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.


<img src="https://github.com/user-attachments/assets/eafd1218-4bce-4c80-81ea-b781cb290317" alt="Image description" width="400" height="150"/>

---

### 16. 🎬 Top Actor-Director Collaborations
```sql
-- What are the top 10 most frequent pairings of actor/actress and director (cast-director duo) in the dataset
SELECT 
    director, 
    unnest(string_to_array(casts, ',')) AS actor,
    COUNT(*) AS collaboration_count
FROM netflix
WHERE director IS NOT NULL AND casts IS NOT NULL
GROUP BY director, actor
ORDER BY collaboration_count DESC
LIMIT 10;
```
➡️ **Objective**: Helps uncover strong creative partnerships on Netflix.


<img src="https://github.com/user-attachments/assets/8890ce7b-48ed-494d-82d0-d3958105fc90" alt="Image description" width="400" height="150"/>

---

### 17. 2. ⏱️ Average Duration: Movies vs TV Shows
```sql
-- Find the average duration of movies versus TV shows (e.g., movie length in minutes vs. number of seasons).

SELECT 
    type,
    AVG(CAST(split_part(duration, ' ', 1) AS INTEGER)) AS avg_duration
FROM netflix
WHERE duration IS NOT NULL
GROUP BY type;

```
➡️ **Objective**: Highlights content length differences by type.


<img src="https://github.com/user-attachments/assets/6e2c18fe-9ca6-44fe-ad24-cb626b0eb613" alt="Image description" width="400" height="150"/>


---

### 18. 📆 Peak Month for Content Addition
```sql
-- Which month(s) saw the highest number of new additions to Netflix?

SELECT 
    TO_CHAR(TO_DATE(date_added, 'Month DD, YYYY'), 'Month') AS month,
    COUNT(*) AS total_added
FROM netflix
WHERE date_added IS NOT NULL
GROUP BY month
ORDER BY total_added DESC
LIMIT 1;

```
➡️ **Objective**: Temporal trend analysis based on date_added, useful for seasonal strategies.


<img src="https://github.com/user-attachments/assets/3e1582a4-652b-4efe-9698-b5e972a57acb" alt="Image description" width="400" height="150"/>

---


### 19.🎭 Top Genres by Content Type
```sql
-- Identify the most common genres for each content type (Movie or TV Show).

SELECT 
    type,
    TRIM(unnest(string_to_array(listed_in, ','))) AS genre,
    COUNT(*) AS genre_count
FROM netflix
GROUP BY type, genre
ORDER BY type, genre_count DESC;

```
➡️ **Objective**: Expands on genre distribution but filters by type.


<img src="https://github.com/user-attachments/assets/39211110-fb9e-46b9-b4b8-3de6eb938ab0" alt="Image description" width="800" height="150"/>

---


### 20. 🎬 Prolific Directors by Volume
```sql
-- Which directors have the highest average IMDb-style "volume"—based on number of shows/movies and average release year?

SELECT 
    director,
    COUNT(*) AS total_titles,
    AVG(release_year) AS avg_release_year
FROM netflix
WHERE director IS NOT NULL
GROUP BY director
HAVING COUNT(*) > 3
ORDER BY total_titles DESC
LIMIT 10;

```
➡️ **Objective**: Find consistently active and prolific directors.


<img src="https://github.com/user-attachments/assets/d434ec06-efef-4f67-9764-580ad7b4e05e" alt="Image description" width="400" height="150"/>


---

### 21. 🌍 Countries with Longest Average Content
```sql
-- Which countries produce the longest average content duration?

SELECT 
    country,
    AVG(CAST(split_part(duration, ' ', 1) AS INTEGER)) AS avg_duration
FROM netflix
WHERE country IS NOT NULL AND duration IS NOT NULL
GROUP BY country
ORDER BY avg_duration DESC
LIMIT 10;

```
➡️ **Objective**: Cross-analyze country and duration.


<img src="https://github.com/user-attachments/assets/0676e46c-9b02-48ae-9c88-7d061ff44432" alt="Image description" width="400" height="150"/>


---

### 22. 📈 Year-over-Year Content Addition
```sql
-- How has the number of new content additions changed year-over-year from 2010 to present?

SELECT 
    EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) AS year_added,
    COUNT(*) AS content_added
FROM netflix
WHERE date_added IS NOT NULL
GROUP BY year_added
ORDER BY year_added;

```
➡️ **Objective**: Time-series view of Netflix’s content growth.


<img src="https://github.com/user-attachments/assets/b43be6bb-7893-4ea0-9659-5b3aec602cdd" alt="Image description" width="400" height="150"/>


---

### 23. 🎥 Frequent Actor Pairings
```sql
-- Which actors appear together most frequently in the same content?

WITH expanded_cast AS (
    SELECT show_id, TRIM(unnest(string_to_array(casts, ','))) AS actor
    FROM netflix
    WHERE casts IS NOT NULL
),
actor_pairs AS (
    SELECT 
        a1.show_id,
        LEAST(a1.actor, a2.actor) AS actor1,
        GREATEST(a1.actor, a2.actor) AS actor2
    FROM expanded_cast a1
    JOIN expanded_cast a2 ON a1.show_id = a2.show_id AND a1.actor < a2.actor
)
SELECT actor1, actor2, COUNT(*) AS appearances
FROM actor_pairs
GROUP BY actor1, actor2
ORDER BY appearances DESC
LIMIT 10;

```
➡️ **Objective**: Performs a co-appearance (network-style) analysis from the casts column.


<img src="https://github.com/user-attachments/assets/e361da21-06d6-4509-af52-f49998d0a374" alt="Image description" width="400" height="150"/>


---

### 24. 🌐 Single vs Multi-Country Productions
```sql
-- What percentage of Netflix content is localized to a single country vs. co-produced by multiple countries?

SELECT 
    CASE 
        WHEN country LIKE '%,%' THEN 'Multi-Country'
        ELSE 'Single Country'
    END AS production_type,
    COUNT(*) AS content_count
FROM netflix
GROUP BY production_type;

```
➡️ **Objective**: Analyzes global collaborations in country.


<img src="https://github.com/user-attachments/assets/026f149f-b44e-4dc2-b392-3d32b103bee5" alt="Image description" width="400" height="150"/>

---

### 25. 📚 Popular Genres by Decade
```sql
-- Group content based on decade of release (e.g., 1980s, 1990s, etc.) and find the most popular genres for each decade.

WITH decade_data AS (
    SELECT *,
           (release_year / 10) * 10 AS decade
    FROM netflix
    WHERE release_year IS NOT NULL
)
SELECT 
    decade,
    TRIM(unnest(string_to_array(listed_in, ','))) AS genre,
    COUNT(*) AS genre_count
FROM decade_data
GROUP BY decade, genre
ORDER BY decade, genre_count DESC;

```
➡️ **Objective**: Shows how genre preferences evolved over time.


<img src="https://github.com/user-attachments/assets/d3fd8629-d7cb-45ef-a09a-18d7f3419b28" alt="Image description" width="400" height="150"/>

---


## 🔍 Findings and Conclusion

This analysis offers a well-rounded understanding of Netflix's content library, highlighting key patterns and actionable insights:

- 🎥 **Content Type Distribution**: Movies dominate the platform but TV shows are on the rise.
- 🌎 **Geographic Trends**: The U.S., India, and the UK lead in content volume, with India showing strong growth.
- 📆 **Temporal Patterns**: New additions peak in certain months—useful for release strategy planning.
- 🏷️ **Genre Preferences**: Different genres trend in different decades and formats.
- 🎭 **Creative Collaborations**: Frequent actor-director duos highlight strong production patterns.
- ⏱️ **Content Duration**: TV shows vary in season length while movies differ by country runtime.
- 🧑‍🤝‍🧑 **Actor Networks**: Popular actor pairings offer casting insights.
- 🌐 **Globalization**: Local vs. multi-country productions help understand content localization.
- 🔍 **Data Quality Gaps**: Missing fields in director/casts highlight metadata improvement areas.


Overall, this analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

---

### 🙌 Let’s Connect

Feel free to explore, clone, or star this repository.  
If you have suggestions, feedback, or questions — I’d love to hear from you!

---
