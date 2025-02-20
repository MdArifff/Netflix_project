# Netflix Movies and TV Shows Data Analysis using SQL
--- Netflix Project


create table netflix4 
(  
	show_id varchar(5),
	type varchar(25),
	title varchar(175),
	director varchar(230),
	casts varchar (1000),
	country	varchar(150),
	date_added varchar (80),
	release_year int,
	rating varchar(10),
	duration varchar(70),
	listed_in varchar(100),
	description varchar (250)
)

	select * from netflix4;


select count(*) 
from netflix4;

select distinct type 
from netflix4;

-- 15 business problems

-- 1. count hte number of movies and tv shows
select type,
     count(*) as total_content
from netflix4
     group by type

-- 2. Find the most common rating for moives and TV shows
select type,
     rating,
	 count(*)
from netflix4
     group by 1,2
     order by 3 desc
     
select type,
     rating,
	 count(*),
	 rank() over(partition by type order by count(*)desc) as ranking  
from netflix4
     group by 1,2
     --order by 3 desc

-- sub query 
select type,
      rating
from 
(
 select type,
     rating,
	 count(*),
	 rank() over(partition by type order by count(*)desc) as ranking  
from netflix4
     group by 1,2
 )   as t1
where 
      ranking = 1



-- 3. List all movies released in a specific year (e.g, 2020) 
-- filter 2020
-- movies

select * from netflix4
where 
    type = 'Movie'
and
    release_year = 2020


--4 Find the top 5 countries with the most conytent on netflix

select 
	country,
	count(show_id) as total_content
from netflix4
group by 1

-- this to separate the country names in one column (UNNEST)
SELECT
	string_to_array(country,',') as new_country
	from netflix4

-- UNNEST very important
SELECT
      unnest(string_to_array(country,',')) as new_country
	from netflix4

-- Final query
SELECT
      unnest(string_to_array(country,',')) as new_country,
	  count(show_id) as total_content
	from netflix4
group by 1
order by 2 desc
limit 5

-- 5.Identify the longest movie
select * from netflix4
where 
     type = 'Movie'
and
     duration = (select max(duration) from netflix4)


-- 6. Find content added in the last 5 years

select * from netflix4
where 
     date_added = current_date - interval '5 years'

select current_date - interval '5 years'

-- final query
select
	*
from netflix4
where 
     to_date(date_added, 'Month DD, YYYY') >= current_date - interval '5 years'

-- 7. find all the movies/tv shows by director 'Rajiv chilaka'

select  * from netflix4
where director = 'Rajiv Chilaka'

-- final query (Ilike/like operator and percentage will select only the director)
select  * from netflix4
where director Ilike '%Rajiv Chilaka%'


-- 8. List all TV shows with more than 5 season
select * from netflix4
where
     type = 'TV Show'
     duration > 5 seasons

--Final quert (split_part take the value of the first ex.apple,banana. output apple) the split line was in text 
	soo we used (::numeric)
select
	*
from netflix4
where 
     type = 'TV Show'
     and
     split_part(duration, ' ', 1)::numeric > 5 

-- 9. Count the number of content items in each genre
	
select
	listed_in,
	show_id,
	unnest (string_to_array(listed_in, ','))
	FROM netflix4

-- final query
select 
      unnest (string_to_array(listed_in, ',')) as genre,
      count(show_id) as total_content
from netflix4
group by 1
-- 10. find each year and the average numbers of content release by India on  netflix.
retuen top 5 year with highest avg content release.

select * from netflix4
where
     country = 'India'

	total content 333/972

select
       extract (year from To_date(date_added, 'month DD, YYYY')) as year,
	 count (*)as yearly_content,
	round(
	count(*)::numeric/(select count(*) from netflix4 where country = 'India')::numeric * 100
	,2) as avg_content_per_year
from netflix4
where country = 'India'
group by 1

-- 11. list all the movie that are documentaries.

select * from netflix4
where 
  listed_in Ilike '%documentaries%'

-- 12. find all the directors
select * from netflix4
where 
    director is null

-- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years

select * from netflix4 
where 
    casts ILike '%Salman Khan%'

-- Find out the 10 years 
select * from netflix4 
where 
    casts ILike '%Salman Khan%'
    And 
release_year > extract(year from current_date) - 10

-- 14. Find out the top 10 actors who have appeared in the highest number of movies produce in India

select 
	show_id,
	casts,
	unnest (string_to_array(casts,',')) as actors
	from netflix4

select 
	--show_id,
	--casts,
	unnest (string_to_array(casts,',')) as actors,
	count(*) as total_content
	from netflix4
	where country Ilike  '%India'
	group by 1
    order by 2 desc
    limit 10

-- 15 Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field.
--Label content containing these keywords as 'Bad' and all other content as 'Good'.
	   --  count how many items fall into each category .

select * from netflix4
where 
      description Ilike '%kill%'
or
   description Ilike '%violence%'



select 
	*,
	  case 
	  when 
     description Ilike '%kill%'
or
   description Ilike '%violence%' then 'Bad_content'
else 'Good_content'
	end category
from netflix4
