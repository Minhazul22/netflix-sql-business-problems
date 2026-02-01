# netflix-sql-business-problems
Solving 15 business problems using PostgreSQL on Netflix dataset
![](https://github.com/Minhazul22/netflix-sql-business-problems/blob/main/logo.png)
## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows)

  ## Schema
```sql
  -- Netflix Project
--Create Table
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
	show_id VARCHAR(6),
	type VARCHAR(10),
	title VARCHAR(105),
	director VARCHAR(208),
	casts VARCHAR(1000),
	country	VARCHAR(150),
	date_added VARCHAR(50),
	release_year INT,
	rating VARCHAR(10),
	duration VARCHAR(15),
	listed_in VARCHAR(100),
	description VARCHAR(250)

);
```
## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows
```sql
SELECT 
	type,
	COUNT(*) as total_content
FROM netflix
GROUP BY type;
```

### Find the most common rating for movies and TV shows---
```sql
SELECT 
	type,
	rating
FROM(

SELECT 
	type,
	rating,
	COUNT(*),
	RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC)as ranking
FROM netflix
GROUP BY 1, 2
-- ORDER BY 3 DESC
) as t1
WHERE 
	ranking = 1

```

###task 3: List all movies released in a specific year (e.g., 2020) filter 2020 movies
```sql
SELECT * FROM netflix
WHERE 
	type = 'Movie'
	AND 
	release_year = 2020
```

###Task 4: Find the top 5 countries with the most content on Netflix
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC 
LIMIT 5
```

###Task 5: Identify the Longest movie on netflix----
```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
AND CAST(REGEXP_SUBSTR(duration, '[0-9]+') AS INT) = (
    SELECT MAX(CAST(REGEXP_SUBSTR(duration, '[0-9]+') AS INT))
    FROM netflix
    WHERE type = 'Movie'
);
```



---task 6: find the contents added in the last 5 years---
SELECT * FROM netflix
WHERE date_added IS NOT NULL
AND TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';


---task 7: Find all the movies/TV Shows by director 'Rajiv Chilaka'
SELECT * FROM netflix
WHERE director LIKE '%Rajiv Chilaka%'

--task 8: List all TV Shows with more than 5 Seasons

SELECT title,
	   CAST(SPLIT_PART(duration, ' ', 1) AS INT) AS seasons
from netflix
where type LIKE '%TV Show'
AND CAST(SPLIT_PART(duration, ' ', 1) AS INT) > 5
ORDER BY seasons DESC;


---Task 9 : Count the Number of Content Items in each genre

SELECT
  TRIM(genre) AS genre,
  COUNT(*) AS total_content
FROM netflix
CROSS JOIN UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre
GROUP BY genre
ORDER BY total_content DESC;

--Task 10: Find each year and the Avg numbers of content release by India on Netflix
--- return top 5 year with highest avg content release !
SELECT
  release_year,
  COUNT(*) AS total_content
FROM netflix
WHERE country LIKE '%India%'
GROUP BY release_year
ORDER BY total_content DESC
LIMIT 5;


---List all movies that are documentaries
SELECT * 
FROM netflix 
WHERE type = 'Movie'
AND listed_in ILIKE '%Documentaries%'


-- task 12: Find all content without a director

SELECT *
FROM netflix
WHERE director IS NULL
OR director = '';

-- task 13: Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT title,
		COUNT(*) AS total_appearance
		FROM netflix
WHERE casts ILIKE '%Salman khan%'
AND release_year >= EXTRACT(YEAR FROM CURRENT_DATE)- 10
GROUP BY title;


-- Task 14: Find the top 10 actors who have appeared in the highest number of movies
--produced in India.
SELECT 
	TRIM(actor) as actor,
	COUNT(*) AS movie_count
FROM netflix
CROSS JOIN UNNEST(STRING_TO_ARRAY(casts, ',')) as actor
WHERE type = 'Movie'
AND country ILIKE '%INDIA%'
AND casts IS NOT NULL
AND TRIM(casts) <> ''
GROUP BY TRIM(actor)
ORDER BY movie_count DESC
LIMIT 10;


--Task 15: Categorize the content based on the presence of the keywords 'Kill' and 'violence' in the description 
-- field. Label the content containing these keywords as 'Bad' and all other 
-- content as 'Good'. Count how many ietms fall into each category

SELECT
	CASE 
		WHEN description IS NOT NULL
		AND (description ILIKE '%kill'
			OR description ILIKE '%violence%')
	THEN 'Bad'
	ELSE 'Good'
	END AS content_category,
	COUNT (*) AS total_items
	FROM netflix
	GROUP BY content_category;
	
