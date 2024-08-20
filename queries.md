Make sure you download the starter code and run the following:

```sh
  psql < movies.sql
  psql movies_db
```

In markdown, you can place a code block inside of three backticks (```) followed by the syntax highlighting you want, for example

\```sql

SELECT \* FROM users;

\```

Using the `movies_db` database, write the correct SQL queries for each of these tasks:

1.  The title of every movie.

SELECT title FROM movies;

2.  All information on the G-rated movies.

SELECT * FROM movies WHERE rating = 'G';

3.  The title and release year of every movie, ordered with the
    oldest movie first.

SELECT title, release_year FROM movies ORDER BY release_year ASC;
    
4.  All information on the 5 longest movies.

SELECT * FROM movies ORDER BY runtime DESC LIMIT 5;

5.  A query that returns the columns of `rating` and `total`, tabulating the
    total number of G, PG, PG-13, and R-rated movies.

SELECT rating, COUNT(*) AS total FROM movies GROUP BY rating;

6.  A table with columns of `release_year` and `average_runtime`,
    tabulating the average runtime by year for every movie in the database. The data should be in reverse chronological order (i.e. the most recent year should be first).

WITH release_years AS (
  SELECT DISTINCT release_year FROM movies ORDER BY release_year DESC
)
SELECT ry.release_year, AVG(m.runtime) AS average_runtime
FROM release_years ry
LEFT JOIN movies m ON ry.release_year = m.release_year
GROUP BY ry.release_year
ORDER BY ry.release_year DESC;

7.  The movie title and studio name for every movie in the
    database.

SELECT m.title, s.name AS studio_name -- replace 's' with actual column alias
FROM movies m
INNER JOIN studios s ON -- replace 'ON' with actual join condition

8.  The star first name, star last name, and movie title for every
    matching movie and star pair in the database.

SELECT s.first_name, s.last_name, m.title -- replace 's' and 'm' with actual column aliases
FROM movies m
INNER JOIN stars_smovies sm ON m.id = sm.movie_id
INNER JOIN stars s ON sm.star_id = s.id;

9.  The first and last names of every star who has been in a G-rated movie. The first and last name should appear only once for each star, even if they are in several G-rated movies. *IMPORTANT NOTE*: it's possible that there can be two *different* actors with the same name, so make sure your solution accounts for that.

WITH g_stars AS (
  SELECT DISTINCT s.first_name, s.last_name -- replace 's' with actual column alias
  FROM stars s
  INNER JOIN stars_smovies sm ON s.id = sm.star_id
  INNER JOIN movies m ON sm.movie_id = m.id AND m.rating = 'G'
)
SELECT first_name, last_name FROM g_stars;

10. The first and last names of every star along with the number
    of movies they have been in, in descending order by the number of movies. (Similar to #9, make sure
    that two different actors with the same name are considered separately).

WITH stars_count AS (
  SELECT s.first_name, s.last_name, COUNT(*) AS movie_count -- replace 's' with actual column alias
  FROM stars s
  INNER JOIN stars_smovies sm ON s.id = sm.star_id
  GROUP BY s.first_name, s.last_name
)
SELECT first_name, last_name, movie_count FROM stars_count ORDER BY movie_count DESC;

### The rest of these are bonuses

11. The title of every movie along with the number of stars in
    that movie, in descending order by the number of stars.

WITH stars_per_movie AS (
  SELECT m.title, COUNT(*) AS star_count -- replace 'm' with actual column alias
  FROM movies m
  INNER JOIN stars_smovies sm ON m.id = sm.movie_id
  GROUP BY m.title
)
SELECT title, star_count FROM stars_per_movie ORDER BY star_count DESC;

12. The first name, last name, and average runtime of the five
    stars whose movies have the longest average.

WITH avg_runtime_star AS (
  SELECT s.first_name, s.last_name, AVG(m.runtime) AS avg_runtime -- replace 's' and 'm' with actual column aliases
  FROM stars s
  INNER JOIN stars_smovies sm ON s.id = sm.star_id
  INNER JOIN movies m ON sm.movie_id = m.id
  GROUP BY s.first_name, s.last_name
)
SELECT first_name, last_name, avg_runtime FROM (
  SELECT *, ROW_NUMBER() OVER (ORDER BY avg_runtime DESC) AS row_num
  FROM avg_runtime_star
) AS ranked_stars WHERE row_num <= 5;

13. The first name, last name, and average runtime of the five
    stars whose movies have the longest average, among stars who have more than one movie in the database.

WITH starred_movies AS (
  SELECT s.first_name, s.last_name, COUNT(*) AS movie_count -- replace 's' with actual column alias
  FROM stars s
  INNER JOIN stars_smovies sm ON s.id = sm.star_id
  GROUP BY s.first_name, s.last_name
),
avg_runtime_star AS (
  SELECT sm.star_id, AVG(m.runtime) AS avg_runtime -- replace 'sm' and 'm' with actual column aliases
  FROM stars_smovies sm
  INNER JOIN movies m ON sm.movie_id = m.id
  GROUP BY sm.star_id
)
SELECT s.first_name, s.last_name, ar.avg_runtime -- replace 's' and 'ar' with actual column aliases
FROM starred_movies s
INNER JOIN avg_runtime_star ar ON s.id = ar.star_id AND s.movie_count > 1
INNER JOIN stars ss ON ss.id = s.id
ORDER BY ar.avg_runtime DESC
LIMIT 5;

14. The titles of all movies that don't feature any stars in our
    database.

SELECT title FROM movies WHERE id NOT IN (
  SELECT movie_id FROM stars_smovies
);

15. The first and last names of all stars that don't appear in any movies in our database.

SELECT first_name, last_name FROM stars WHERE id NOT IN (
  SELECT star_id FROM stars_smovies
);

16. The first names, last names, and titles corresponding to every
    role in the database, along with every movie title that doesn't have a star, and the first and last names of every star not in a movie.

-- Note: This assumes there is a separate "roles" table with columns id, character_name, ...
SELECT s.first_name, s.last_name, r.character_name AS role_title -- replace 's' and 'r' with actual column aliases
FROM stars s
LEFT JOIN stars_smovies sm ON s.id = sm.star_id
LEFT JOIN roles r ON sm.movie_id = r.movie_id AND s.id = r.star_id
UNION ALL
SELECT NULL, NULL, m.title AS movie_title -- replace 'm' with actual column alias
FROM movies m WHERE id NOT IN (
  SELECT movie_id FROM stars_smovies
)
UNION ALL
SELECT s.first_name, s.last_name, NULL AS role_title -- replace 's' with actual column alias
FROM stars s WHERE id NOT IN (
  SELECT star_id FROM stars_smovies
);
