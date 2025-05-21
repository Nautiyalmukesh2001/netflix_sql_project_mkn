
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
- [🔍 Key Insights & Findings](#-findings-and-conclusion)

---

## 📌 Overview

This project involves analyzing the Netflix dataset using **PostgreSQL** to answer **business-relevant questions** around content types, genres, release years, popular countries, actors, and more.  

By querying the dataset effectively, we reveal **content trends** and **strategic insights** valuable for stakeholders in the streaming and entertainment industry.

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

---

## 💡 Business Problems & Solutions

Each of the following problems is solved using SQL with optimized queries and real-world objectives in mind:

### 1. 📊 Distribution of Movies vs TV Shows
```sql
SELECT type,COUNT(*) AS total_content
FROM netflix
GROUP BY type;
```
➡️ **Insight**: Helps understand platform content strategy.

<img src="https://github.com/user-attachments/assets/469c27bf-45e8-4fc5-9f81-d49ca2700426" alt="Image description" width="400" height="150"/>

---

### 2. ⭐ Most Common Rating by Type
```sql
WITH RankedRatings AS (
  SELECT type, rating, COUNT(*) AS rating_count,
         RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS rank
  FROM netflix GROUP BY type, rating
)
SELECT type, rating FROM RankedRatings WHERE rank = 1;
```
➡️ **Insight**: Shows what content ratings dominate per type.

---

### 3. 🗓️ Movies Released in 2020
```sql
SELECT * FROM netflix WHERE release_year = 2020;
```

---

### 4. 🌍 Top 5 Countries by Content Count
```sql
SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS country, COUNT(*) AS total
FROM netflix
GROUP BY country
ORDER BY total DESC
LIMIT 5;
```

---

### 5. ⏱️ Longest Movie
```sql
SELECT * FROM netflix WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;
```

---

### 6. 🆕 Content Added in Last 5 Years
```sql
SELECT * FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

---

### 7. 🎬 All Shows Directed by Rajiv Chilaka
```sql
SELECT * FROM (
  SELECT *, UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name FROM netflix
) t WHERE director_name = 'Rajiv Chilaka';
```

---

### 8. 📺 TV Shows with Over 5 Seasons
```sql
SELECT * FROM netflix
WHERE type = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

---

### 9. 🎭 Count Content by Genre
```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, COUNT(*) 
FROM netflix GROUP BY genre;
```

---

### 10. 📈 Top 5 Years for Indian Content (Avg%)
```sql
SELECT release_year, COUNT(*) AS total,
ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM netflix WHERE country = 'India'), 2) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY release_year
ORDER BY avg_release DESC
LIMIT 5;
```

---

### 11. 📚 All Documentaries
```sql
SELECT * FROM netflix WHERE listed_in LIKE '%Documentaries';
```

---

### 12. ❌ Content Without a Director
```sql
SELECT * FROM netflix WHERE director IS NULL;
```

---

### 13. 🎞️ Salman Khan Movies (Last 10 Years)
```sql
SELECT * FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year >= EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

---

### 14. 🎭 Top 10 Indian Actors by Appearances
```sql
SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor, COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

---

### 15. 🚫 Content Categorized by 'Kill' or 'Violence'
```sql
SELECT category, COUNT(*) FROM (
  SELECT CASE
      WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
      ELSE 'Good'
  END AS category
  FROM netflix
) t GROUP BY category;
```

---

## 🔍 Findings and Conclusion

- 🎥 **Content Mix**: Movies dominate the catalog, but TV shows are steadily growing.
- 🌎 **Top Countries**: US, India, and UK lead in content production.
- 🏆 **Popular Ratings**: Most content is family-friendly (TV-MA, TV-14).
- 🧠 **Meta Gaps**: Missing director data and ambiguous cast fields need data cleaning.
- 🎯 **Actionable Insight**: Genre trends and keyword flags help shape content strategy.

---

### 🙌 Let’s Connect

Feel free to explore, clone, or star this repository.  
If you have suggestions, feedback, or questions — I’d love to hear from you!

---
