## netflix_data_analysis_sql
![netflix_data_analysis_sql](https://github.com/SunilKhandale/netflix_data_analysis_sql/blob/main/netflix_image.jpeg)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives
1. Analyze the distribution of content types (movies vs TV shows).
2. Identify the most common ratings for movies and TV shows.
3. List and analyze content based on release years, countries, and durations.
4. Explore and categorize content based on specific criteria and keywords.

## schema
## creating database and table

```sql
create database netflix_db;
use netflix_db;

drop table if exists netflix;
create table netflix(
show_id varchar(10), type varchar(15), title varchar(150), director varchar(250), casts varchar(1000), country varchar(150),	
date_added varchar(50), release_year int, rating varchar(10), duration varchar(15),	listed_in varchar(150), description varchar(300));
select * from netflix;
select distinct type from netflix;
```
## -- Task 1 : count number of movies vs tv shows
```sql
select type, count(*) as total_content from netflix group by type;
```

## -- Task 2 : Find the Most Common Rating for Movies and TV Shows
```sql
select type, rating from (
select type, rating, count(*),
rank() over(partition by type order by count(*) desc) as ranking
from netflix
group by 1, 2) as t1
where ranking = 1;
select distinct type from netflix;
```

## -- Task 3: List All Movies Released in a Specific Year (e.g., 2020) 
```sql
select type, release_year from netflix
where release_year = '2020' and type = 'Movie';
```

## -- 4. Find the Top 5 Countries with the Most Content on Netflix
```sql
SELECT jt.country, COUNT(*) AS total_content
FROM netflix n
CROSS JOIN JSON_TABLE(
    CONCAT('["', REPLACE(n.country, ',', '","'), '"]'),
    '$[*]' COLUMNS (country VARCHAR(150) PATH '$')
) AS jt
WHERE jt.country IS NOT NULL AND jt.country <> ''
GROUP BY jt.country
ORDER BY total_content DESC
LIMIT 5;

-- here we have mutiple name of countries in 1 row so we need to seprate it
```

## -- 5. Identify the Longest Movie
```sql
select * from netflix
where type = 'movie'
and
duration = (select max(duration) as max_duration from netflix);
```

## -- 6. Find Content Added in the Last 5 Years
```sql
SELECT *
FROM netflix
where STR_TO_DATE(date_added, '%M %d, %Y') >= curdate() - interval 5 year; -- %d needs to be small d --
-- select curdate() - interval 5 year;
```

## -- 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka' 
```sql
select * from netflix
where director like '%JJC Skillz%'; -- in % name is not case senstive it has to be in propoer format 
 -- we used another name because we have only 110 rows in our table and there is no such name rajiv chilaka
```

## -- 8. List All TV Shows with More Than 5 Seasons
```sql
select * from netflix
 where type = 'TV Show'
 and duration >= '5 seasons';
 
 SELECT *, duration = CAST(SUBSTRING_INDEX(duration, ' ', 1) as unsigned)
 from netflix
where type = 'TV Show' 
and
duration >= 5;
```

## -- 9. Count the Number of Content Items in Each Genre
-- here some content belongs to multiple genres so we need to seprate them first
```sql
SELECT count(n.show_id), n.listed_in, jt.genre, count(*) as total_genere
FROM netflix n,
JSON_TABLE(
    CONCAT('["', REPLACE(n.listed_in, ',', '","'), '"]'),
    '$[*]' COLUMNS (genre VARCHAR(255) PATH '$')
) AS jt
GROUP BY n.show_id, n.listed_in, jt.genre;
```

## -- 10.Find each year and the average numbers of content release in India on netflix.
```sql
select date_added, country from netflix where country = 'India';

select extract(year from str_to_date(date_added, '%M %d, %Y')) as only_year, count(*),
round(count(*)/(select count(*) from netflix where country = 'India') * 100, 2) as avg_contet_per_year from netflix
where country = 'India'
group by 1;
```

## -- task 11 List All Movies that are Documentaries 
```sql
select * from netflix
where listed_in like '%Documentaries%';
```

## -- 12. Find All Content Without a Director 
```sql
select director, count(*) from netflix
where director is null or director = ''
group by 1;
```
## --  13 Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
```sql
select * from netflix
where casts = 'Salman khan' or casts like'%Salman khan%'
and release_year >= date_sub(curdate(), interval 10 year);
-- or release_year >= date_sub(release_year, interval 10 year)
-- or extract(year from curdate()) - 10
```

## -- 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
```sql
select n.show_id, trim(jt.casts), n.country -- trim is used to treat the spaces after the vlue in single row is separeted 
FROM netflix n,
JSON_TABLE(
    CONCAT('["', REPLACE(n.casts, ',', '","'), '"]'),
    '$[*]' COLUMNS (casts VARCHAR(100) PATH '$')
) as jt
where n.type = 'Movie' and n.country = 'India';
```

## -- 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
```sql
with new_table as(
select *, 
case
when description like '%kill%' 
or
description like '%violence%'
then 'bad_content'
else 'good_content'
end category 
from netflix
)
select category,
count(*) as total_content from new_table
group by 1;
commit;
```

## Findings and Conclusion
1. Content Distribution: The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
2. Common Ratings: Insights into the most common ratings provide an understanding of the content's target audience.
3. Geographical Insights: The top countries and the average content releases by India highlight regional content distribution.
4. Content Categorization: Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.
This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
rollback;

```
