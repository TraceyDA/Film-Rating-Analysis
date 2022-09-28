# Film-Rating-Analysis
````sql
CREATE DATABASE Film_database

/* Delete the tables if they already exist */
drop table if exists Movie;
drop table if exists Reviewer;
drop table if exists Rating;

/* Create the schema for our tables */
create table Movie(mID int, title VARCHAR(MAX), year int, director VARCHAR(MAX));
create table Reviewer(rID int, name VARCHAR(MAX));
create table Rating(rID int, mID int, stars int, ratingDate date);

/* Populate the tables with our data */
insert into Movie values(101, 'Gone with the Wind', 1939, 'Victor Fleming');
insert into Movie values(102, 'Star Wars', 1977, 'George Lucas');
insert into Movie values(103, 'The Sound of Music', 1965, 'Robert Wise');
insert into Movie values(104, 'E.T.', 1982, 'Steven Spielberg');
insert into Movie values(105, 'Titanic', 1997, 'James Cameron');
insert into Movie values(106, 'Snow White', 1937, null);
insert into Movie values(107, 'Avatar', 2009, 'James Cameron');
insert into Movie values(108, 'Raiders of the Lost Ark', 1981, 'Steven Spielberg');

insert into Reviewer values(201, 'Sarah Martinez');
insert into Reviewer values(202, 'Daniel Lewis');
insert into Reviewer values(203, 'Brittany Harris');
insert into Reviewer values(204, 'Mike Anderson');
insert into Reviewer values(205, 'Chris Jackson');
insert into Reviewer values(206, 'Elizabeth Thomas');
insert into Reviewer values(207, 'James Cameron');
insert into Reviewer values(208, 'Ashley White');

insert into Rating values(201, 101, 2, '2011-01-22');
insert into Rating values(201, 101, 4, '2011-01-27');
insert into Rating values(202, 106, 4, null);
insert into Rating values(203, 103, 2, '2011-01-20');
insert into Rating values(203, 108, 4, '2011-01-12');
insert into Rating values(203, 108, 2, '2011-01-30');
insert into Rating values(204, 101, 3, '2011-01-09');
insert into Rating values(205, 103, 3, '2011-01-27');
insert into Rating values(205, 104, 2, '2011-01-22');
insert into Rating values(205, 108, 4, null);
insert into Rating values(206, 107, 3, '2011-01-15');
insert into Rating values(206, 106, 5, '2011-01-19');
insert into Rating values(207, 107, 5, '2011-01-20');
insert into Rating values(208, 104, 3, '2011-01-02');
````
## 1. Find the names of all reviewers who rated Gone with the Wind. 

USE Film_database
````sql
SELECT  DISTINCT name
FROM Movie Mo
INNER JOIN Rating R ON R.mId = Mo.mID
INNER JOIN Reviewer RE ON RE.rId = R.rID
WHERE title = 'Gone with the Wind';
````

### 2. For any rating where the reviewer is the same as the director of the movie, return the reviewer name, movie title, and number of stars. 
````sql
SELECT name, title, stars
FROM Movie Mo
INNER JOIN Rating R ON R.mId = Mo.mID
INNER JOIN Reviewer RE ON RE.rId = R.rID
WHERE director = name;
````

### 3. Return all directors names and reviewer's name together in a single list, alphabetized. 
````sql
SELECT *
FROM 
(SELECT director AS List
FROM Movie

UNION ALL

SELECT name AS List 
FROM Reviewer) as raw_data
ORDER BY List ASC
````

### 4. Find the titles of all movies not reviewed by Chris Jackson. 
````sql
SELECT title
FROM Movie
WHERE mId NOT IN 
(
  SELECT mId
  FROM Rating RA
  INNER JOIN Reviewer RE ON RE.rId = RA.rID
  WHERE name = 'Chris Jackson'
);
````

### 5. For all pairs of reviewers such that both reviewers gave a rating to the same movie, return the names of both reviewers. Eliminate duplicates, don't pair reviewers with themselves, 
### And include each pair only once. For each pair, return the names in the pair in alphabetical order.
````sql
SELECT DISTINCT Re1.name, Re2.name
FROM Rating R1, Rating R2, Reviewer Re1, Reviewer Re2
WHERE	1=1
		AND R1.mID = R2.mID
		AND R1.rID = Re1.rID
		AND R2.rID = Re2.rID
		AND Re1.name < Re2.name
ORDER BY Re1.name, Re2.name;
````

-- USE JOIN 
````sql
SELECT DISTINCT
		Re1.name, Re2.name, R1.mID, R2.mID
FROM Rating R1 
JOIN Reviewer Re1 ON  R1.rID = Re1.rID 
JOIN Rating R2 ON  R1.mID = R2.mID
JOIN Reviewer Re2 ON R2.rID = Re2.rID
WHERE  	1=1
		AND Re1.name < Re2.name 

ORDER BY Re1.name, Re2.name;
````

### 6. For each rating that is the lowest (fewest stars) currently in the database, return the reviewer name, movie title, and number of stars.
````sql
SELECT name, title, stars
FROM Movie MO
JOIN Rating R ON R.mID = Mo.mID
JOIN Reviewer Re ON  R.rID = RE.rID 
WHERE stars = (SELECT MIN(stars) FROM Rating);
````

### 7. List movie titles and average ratings, from highest-rated to lowest-rated. If two or more movies have the same average rating, list them in alphabetical order. 
````sql
SELECT title, AVG(stars) AS average
FROM Movie MO
INNER JOIN Rating R ON R.mID = Mo.mID
GROUP BY Mo.title
ORDER BY average DESC, title;
````

### 8. Find the names of all reviewers who have contributed three or more ratings.
````sql
SELECT name
FROM Reviewer
WHERE (SELECT COUNT(*) FROM Rating WHERE Rating.rId = Reviewer.rId) >= 3;

SELECT name 
		,count(mID)
FROM Reviewer RE
INNER JOIN Rating R ON R.rId = Re.rId
--WHERE (name != 'Brittany Harris' AND mID != 103)
GROUP BY name
HAVING COUNT (mID) >= 3;
````

### 9. Some directors directed more than one movie. For all such directors, return the titles of all movies directed by them, along with the director name. (Sort by director name, then movie title).
````sql
WITH director_tab AS 
(
	SELECT director, COUNT(*) as number_flim
	FROM Movie M2 
	group by director 
	having COUNT(*) >1 
)

SELECT title, M1.director
FROM Movie M1
JOIN director_tab d  ON d.director = M1.director
ORDER BY M1.director, title;
````

### 10. Find the movie(s) with the highest average rating. Return the movie title(s) and average rating.
````sql
SELECT title, AVG(stars) as average
FROM MOVIE
INNER JOIN RATING ON Movie.mID = Rating.mID
INNER JOIN REVIEWER ON Reviewer.rID = Rating.rID
GROUP BY movie.mID, title
HAVING  AVG(stars) = (
SELECT MAX(avg_stars)
FROM (
SELECT title, AVG(stars) AS avg_stars
FROM MOVIE
INNER JOIN RATING ON Rating.mID = Movie.mID
GROUP BY movie.mID, title
) I
);
````

### 11. For each director, return the director's name together with the title(s) of the movie(s) they directed that received the highest rating among all of their movies, and the value of that rating. Ignore movies whose director is NULL. 
````sql
SELECT director, title
		, MAX(stars) Max_rating_star
		, MIN(stars) AS Min_rating_star
FROM Movie Mo
INNER JOIN Rating R ON R.mId = Mo.mID
WHERE director IS NOT NULL --  Ignore movies whose director is NULL.  
GROUP BY director, title;
````
