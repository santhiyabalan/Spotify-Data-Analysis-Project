
# Spotify Data Analysis Project

Project Category: Advanced  
[Click Here to get Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using SQL. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

## Table Creation
```sql
DROP TABLE IF EXISTS spotify_;
CREATE TABLE spotify_ (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed VARCHAR(5),
    official_video VARCHAR(5),
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```

## New Table Creation
To enhance the project, a new table named `trackinfo` has been added to perform join functions.

```sql
-- create table trackinfo
CREATE TABLE trackinfo (
    trackid INT,
    ReleaseDate DATE,
    tracklable VARCHAR(100),
    FOREIGN KEY(trackid) REFERENCES spotify_(trackid)
);
```

## Data Input into Trackinfo Table
```sql
LOAD DATA INFILE "C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/trackinfo.csv"
INTO TABLE trackinfo
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES;
```

## SQL Queries for Spotify Data Analysis
### Old Queries
```sql
-- 1. Retrieve the names of all tracks that have more than 1 billion streams.
SELECT * FROM spotify_ WHERE stream > 1000000000;

-- 2. List all albums along with their respective artists.
SELECT DISTINCT album, artist FROM spotify_;

-- 3. List all tracks along with their views and likes where official_video = TRUE.
SELECT track, SUM(views), SUM(likes) FROM spotify_ 
WHERE official_video = "true" 
GROUP BY 1 
ORDER BY 2 DESC;

-- 4. Get the total number of comments for tracks where licensed = TRUE.
SELECT SUM(comments) AS TOTAL_COMMENTS FROM spotify_ WHERE licensed = "TRUE";

-- 5. For each album, calculate the total views of all associated tracks.
SELECT album, SUM(views) AS total_views
FROM spotify_
GROUP BY album
ORDER BY 2 DESC;

-- 6. Find all tracks that belong to the album type single.
SELECT * FROM spotify_ WHERE album_type = "single";

-- 7. Find the top 3 most-viewed tracks for each artist using window functions.
WITH ranking_artist AS (
    SELECT artist, track, SUM(views) AS total_view,
           DENSE_RANK() OVER (PARTITION BY artist ORDER BY SUM(views) DESC) AS ranking  
    FROM spotify_
    GROUP BY artist, track
)
SELECT * FROM ranking_artist WHERE ranking <= 3;

-- 8. Count the total number of tracks by each artist.
SELECT artist, COUNT(*) AS no_of_tracks FROM spotify_ 
GROUP BY artist 
ORDER BY 2;

-- 9. Find artists with more than one album.
SELECT artist, COUNT(album) AS album_count FROM spotify_ 
GROUP BY 1
HAVING album_count > 1 
ORDER BY album_count; 

-- 10. Calculate the average danceability of tracks in each album.
SELECT album, AVG(danceability) FROM spotify_ 
GROUP BY album 
ORDER BY 2 DESC;

-- 11. Find the top 5 most popular tracks (based on likes) by artists with more than 1 million streams, 
-- where the danceability score is above 0.6 and the track is not an official video.
SELECT artist, track, album, danceability, likes, stream
FROM spotify_
WHERE stream > 1000000 AND danceability > 0.6 AND official_video = FALSE
ORDER BY likes DESC
LIMIT 5;

-- 12. Find the top 5 tracks with the highest energy values.
SELECT track, energy FROM spotify_ ORDER BY energy DESC LIMIT 5;

-- 13. Tracks with highest average views per stream.
SELECT track, 
       (SUM(views) / SUM(stream)) AS avg_views_per_stream
FROM spotify_
WHERE stream > 0  -- Avoid division by zero
GROUP BY track
ORDER BY avg_views_per_stream DESC
LIMIT 10;

-- 14. Find tracks where the liveness score is above the average.
SELECT artist, track, liveness 
FROM spotify_ 
WHERE liveness > (SELECT AVG(liveness) FROM spotify_);

-- 15. Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.
WITH cte AS (
    SELECT album,
           MIN(energy) AS lowest_energy,
           MAX(energy) AS highest_energy
    FROM spotify_
    GROUP BY 1
)
SELECT album, 
       highest_energy - lowest_energy AS energy_difference
FROM cte
ORDER BY 2 DESC;

-- 16. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window function.
SELECT track, artist, views, likes,
       SUM(likes) OVER (ORDER BY views) AS cumulative_likes
FROM spotify_
ORDER BY views;

-- 17. Find the Top 3 Artists by Total Likes.
SELECT artist, track, album, SUM(likes) AS total_likes
FROM spotify_
GROUP BY artist
ORDER BY total_likes DESC
LIMIT 3;

-- 18. Find the Top 3 Most Liked Tracks for Each Artist.
WITH ranked_artist AS (
    SELECT artist, track, likes,
           ROW_NUMBER() OVER (PARTITION BY artist ORDER BY likes DESC) AS ranks
    FROM spotify_
)
SELECT * FROM ranked_artist WHERE ranks <= 3;
```

### New Queries for Enhanced Project
```sql
-- Finding Track Details for a Specific Release Date -- eg;2014-06-12
SELECT s.title, s.artist, t.tracklable
FROM SPOTIFY_ s
JOIN TRACKINFO t ON s.trackid = t.trackid
WHERE t.releasedate = '2014-06-12'; 

-- Finding Tracks without a Record Label
SELECT s.title, s.artist 
FROM spotify_ s LEFT JOIN trackinfo t ON s.trackid = t.trackid
WHERE t.tracklable IS NULL;  

-- Artists with More Than One Track Released in a Specific Year
SELECT s.artist, COUNT(s.trackid) AS track_count
FROM SPOTIFY_ s
JOIN TRACKINFO t ON s.trackid = t.trackid
WHERE YEAR(t.releasedate) = 2023  
GROUP BY s.artist
HAVING COUNT(s.trackid) > 1;

-- Finding Artists with Tracks Longer Than the Average
SELECT artist, track, album, duration_min 
FROM spotify_
WHERE duration_min > (SELECT AVG(duration_min) FROM spotify_);

-- Finding Artists Who Have No Tracks Released After a Certain Year -- eg:2020
SELECT s.artist
FROM SPOTIFY_ s
JOIN TRACKINFO t ON s.trackid = t.trackid
GROUP BY s.artist
HAVING MAX(YEAR(t.releasedate)) <= 2020;

-- Finding the Number of Tracks Released Per Year
SELECT YEAR(t.releasedate) AS released_yr,
       COUNT(t.trackid) AS track_count
FROM spotify_ s 
JOIN trackinfo t ON s.trackid = t.trackid
GROUP BY YEAR(t.releasedate)
ORDER BY track_count DESC;
```

### 5. Index Creation and Query Optimization
```sql
-- Index Creation
CREATE INDEX idx_artist ON spotify_(artist);

-- Execute the Query Before Creating the Index
SELECT * FROM spotify_ WHERE artist = 'Gorillaz';

-- Execute the Same Query After Creating the Index
SELECT * FROM spotify_ WHERE artist = 'Gorillaz';
```

## Query Optimization Technique
To improve query performance, we carried out an optimization process focusing on indexing the artist column. This optimization significantly reduces query time, enhancing the overall performance of our database operations in the Spotify project.

### Initial Query Performance Analysis Using EXPLAIN
Before creating the index, we analyzed the performance of a query that retrieved tracks based on the artist column. The performance metrics were as follows:
- **Execution time (E.T.)**: 0.062 ms

### EXPLAIN Results Before Index Creation
![before_indexing](https://github.com/user-attachments/assets/de010985-d99b-4482-a5dd-3e809f747b0a)

### Index Creation on the Artist Column
To optimize the query performance, we created an index on the artist column, which ensures faster retrieval of rows where the artist is queried. The SQL command for creating the index is as follows:
```sql
CREATE INDEX idx_artist ON spotify_(artist);
```

### Performance Analysis After Index Creation
After creating the index, we ran the same query again and observed significant improvements in performance:
- **

Execution time (E.T.)**: 0.000 ms

### EXPLAIN Results After Index Creation
(![after_indexing](https://github.com/user-attachments/assets/7af07f68-1f80-4792-9187-54d95d4a4274))

## Conclusion
The Spotify Data Analysis Project demonstrates advanced SQL skills, including normalization, complex queries, and query optimization techniques. The project provides valuable insights into Spotify's dataset, making it a useful resource for understanding track performance and artist engagement on the platform.
```

