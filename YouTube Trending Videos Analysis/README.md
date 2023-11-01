# YouTube Trending Videos Analysis Project

## Background Story

YouTube is the world's largest video streaming platform, with billions of videos consumed daily across the globe. This project aims to optimize trending video duration on YouTube by understanding the impact of likes, comments, and dislikes on video consumption. The Head of Product – trending videos at YouTube has tasked us with this analysis to improve video views.

## Business Problem

YouTube's trending videos vary by location, and the impact of engagement metrics like likes, dislikes, and comments differs from country to country. Demographics play a significant role in video consumption patterns. To address this, our manager wants us to analyze how the average trending periods of videos vary across countries and examine how engagement metrics affect trending video duration.

## Dataset

We have three tables for analysis:
1. YT_trending_videos: Contains video-level information, including trending dates and metrics like comments, likes, and views.

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/f1880ace-5b66-4129-a8fb-99da35736bca)

2. YT_channel_map: Provides channel_id mapping to channel titles.

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/fcc029a7-0f60-4c8c-8df0-0dae70add36c)

3. YT_category_map: Maps videos to categories based on their type.

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/de710e5b-d628-4853-9006-1a8008d33185)

-----------------------------------------------------------------------------------------------------------------------------------------
#### Question 1: Create a report for overall distribution of duration of trending videos by each country. The report will have video_id, video name (title), country, No_of_days_trended. 
Sample Report:

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/a004f1f5-b319-485d-8a19-aba3118663e1)

#### How many videos have trended for more than 5 days in the US?

```sql
SELECT
    country,
    COUNT(video_id) AS no_of_videos
FROM
    (SELECT
        video_id,
        title,
        country,
        COUNT(trending_date) AS no_of_days_trended
    FROM
        yt_trending_videos
    GROUP BY
        video_id,
        title,
        country) a
WHERE
    no_of_days_trended > 5
    AND country = 'US'
GROUP BY
    country;
```

#### Question 2: Create a report for overall distribution of duration of trending videos by each category. The report will have video_id, category title, No_of_days_trended.
Sample Report:

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/0cd62c5c-e9ba-4006-aab0-fd82909719c5)

#### Which category has the highest average trending period?

```sql
SELECT
    snippettitle AS category_title,
    AVG(no_of_days_trended) AS avg_trending_days
FROM
    (SELECT
        video_id,
        title,
        snippettitle,
        COUNT(trending_date) AS no_of_days_trended
    FROM
        yt_trending_videos a
    INNER JOIN
        yt_category_map b
    ON
        a.category_id = b.id
    GROUP BY
        video_id,
        title,
        snippettitle) a
GROUP BY
    snippettitle
ORDER BY
    avg_trending_days DESC;
```

#### Question 3: Create a report for the number of distinct videos trending from each category on day of the week. The report will have weekday, category title, No_videos_trended.
Sample Report:

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/f4ad96be-2143-4066-8749-e15294b934a3)

  Note: The answer cannot be derived from the report. In the report, we have a number of videos trending for each
    weekday by category. Hence, if a video has trended on multiple days, it will be counted on each day. The
    question asks for distinct videos and hence write the query considering that.
    
#### How many distinct videos trended from the category ‘Music’ on weekdays (Monday - Friday)?

```sql
SELECT snippettitle,
       Count(DISTINCT video_id) AS no_of_videos
FROM yt_trending_videos a
INNER JOIN yt_category_map b ON a.category_id = b.id
WHERE Weekday(trending_date) BETWEEN 0 AND 4
  AND snippettitle = 'Music'
GROUP BY snippettitle
```

#### Question 4: Create a summary report which contains country, category title, total_views, total_likes and avg_trending_days.
Sample Report:

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/17886be9-9e54-45cc-b5d4-b948029a40ce)

#### What are the total views for category sports in ‘Canada’?

```sql
SELECT snippettitle AS category_title,
       country,
       Sum(total_views) AS total_views,
       Sum(total_likes) AS total_likes,
       Avg(no_of_days_trended) AS avg_trending_days
FROM
(SELECT video_id,
          title,
          snippettitle,
          country,
          Sum(VIEWS) AS total_views,
          Sum(likes) AS total_likes,
          Count(trending_date) AS no_of_days_trended
   FROM yt_trending_videos a
   INNER JOIN yt_category_map b ON a.category_id = b.id
   GROUP BY video_id,
            title,
            snippettitle,
            country) c
WHERE country = 'Canada'
  AND snippettitle = 'Sports'
GROUP BY snippettitle,
         country
ORDER BY country,
        avg_trending_days DESC
```

#### Question 5: Rank the videos based on views, likes within each country. Which country has the highest number of videos with rank for views and rank of likes both in top 20?

```sql
SELECT country,
       Count(video_id) AS no_of_videos
FROM
  (SELECT video_id,
          title,
          country,
          VIEWS,
          likes,
          Rank() OVER (PARTITION BY country
                       ORDER BY VIEWS DESC) AS rank_views,
                      Rank() OVER (PARTITION BY country
                                   ORDER BY likes DESC) AS rank_likes
   FROM yt_trending_videos) a
WHERE rank_views <= 20
  AND rank_likes <= 20
GROUP BY country
ORDER BY no_of_videos DESC
```

#### Question 6: Generate a report at video level with video viewership rating within the category. (Report at video level means the output should be unique at video_id).
    Formula to assign the rating: ((Views - min(views))*100 ) / max(views) - min(views)
    where max(views) is maximum views in the respective video’s category
    and min(views) is minimum views in the respective video’s category
Sample Report:

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/c7b459b0-c089-41e1-b1ca-8bd6fe389210)

##### What is the average rating of the category Music?

```sql
SELECT category_title,
       Avg(rating) AS avg_rating
FROM
  (SELECT c.*,
          Round(((VIEWS - min_views) * 100) / (max_views - min_views), 0) AS rating
   FROM
     (SELECT DISTINCT video_id,
                      title,
                      snippettitle AS category_title,
                      VIEWS,
                      Max(VIEWS) OVER (PARTITION BY category_id) AS max_views,
                                      Min(VIEWS) OVER (PARTITION BY category_id) AS min_views
      FROM yt_trending_videos a
      INNER JOIN yt_category_map b ON a.category_id = b.id) c) d
GROUP BY category_title
```

#### Question 7: Generate a report at video level with video ratings. (Report at video level means the output should be unique at video_id).
    Rating formula to assign the rating: ((Likes - min(likes))*100 ) / max(likes) - min(likes)
    where max(likes) is maximum likes in the respective video’s category
    and min(likes) is minimum likes in the respective video’s category
Sample Report:

![image](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/99d4be95-7620-44a3-ae65-35dffe1a0420)

#### Which category has the highest average rating based on likes?

```sql
SELECT category_title,
       Avg(rating) AS avg_rating
FROM
  (SELECT c.*,
          Round(((likes - min_likes) * 100) / (max_likes - min_likes), 0) AS rating
   FROM
     (SELECT DISTINCT video_id,
                      title,
                      snippettitle AS category_title,
                      likes,
                      Max(likes) OVER (PARTITION BY category_id) AS max_likes,
                                      Min(likes) OVER (PARTITION BY category_id) AS min_likes
      FROM yt_trending_videos a
      INNER JOIN yt_category_map b ON a.category_id = b.id) c) d
GROUP BY category_title
ORDER BY avg_rating DESC
```


