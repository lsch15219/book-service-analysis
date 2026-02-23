### Working with business data in ClickHouse

This project involves analyzing Book Service usage data using ClickHouse. The project consists of writing queries in ClickHouse. Each analysis includes findings and insights derived from the data, focusing on patterns of content consumption across platforms and regions.

The analysis primarily covers mobile platforms - Bookmate iOS and Bookmate Android - with occasional reference to Bookmate Web when relevant. Data comes from two main tables: source_db.audition, source_db.content

### Task 1

Show the top 20 cities and regions by total hours of reading and listening to content on mobile devices. Include separate columns for iOS and Android durations. Exclude federal districts and round all values to whole numbers.

SELECT usage_geo_id_name AS region,
       round(sum(hours)) AS duration,
       round(sumIf(hours, usage_platform_ru='Букмейт iOS')) AS iOS,
       round(sumIf(hours, usage_platform_ru='Букмейт Android')) AS Android
FROM source_db.audition
WHERE usage_country_name = 'Россия' 
      AND usage_geo_id_name NOT LIKE '%федеральный округ%'
      AND (usage_platform_ru ='Букмейт iOS' OR usage_platform_ru ='Букмейт Android')
GROUP BY usage_geo_id_name 
ORDER BY duration  DESC 
LIMIT 20;

region                                 |duration|iOS   |Android|
---------------------------------------+--------+------+-------+
Москва                                 | 26629.0|8056.0|18573.0|
Санкт-Петербург                        | 12538.0|3794.0| 8744.0|
Москва и Московская область            |  8248.0|2412.0| 5837.0|
Екатеринбург                           |  4818.0|1493.0| 3325.0|
Россия                                 |  4469.0|1320.0| 3149.0|
Краснодар                              |  3998.0|1219.0| 2779.0|
Новосибирск                            |  3824.0|1155.0| 2669.0|
Ростов-на-Дону                         |  3115.0| 964.0| 2151.0|
Казань                                 |  3000.0|1022.0| 1978.0|
Пермь                                  |  2914.0| 821.0| 2093.0|
Санкт-Петербург и Ленинградская область|  2884.0| 902.0| 1982.0|
Уфа                                    |  2621.0| 779.0| 1842.0|
Нижний Новгород                        |  2602.0| 702.0| 1899.0|
Челябинск                              |  2586.0| 735.0| 1850.0|
Красноярск                             |  2126.0| 685.0| 1441.0|
Краснодарский край                     |  2084.0| 675.0| 1409.0|
Воронеж                                |  1841.0| 552.0| 1290.0|
Тюмень                                 |  1827.0| 581.0| 1245.0|
Самара                                 |  1808.0| 558.0| 1250.0|
Нижегородская область                  |  1574.0| 429.0| 1145.0|

### Task 2

Analyze the most popular content on mobile platforms. Identify the top 5 books by total hours of reading and listening. Compute the average reading time for text books and average listening time for audiobooks. Include only books available in both formats. Round all numerical values to two decimal places.



```python
SELECT main_content_name,
       main_author_name,
       round(sum(hours), 2) AS total_duration,
       round(avgIf(hours, main_content_type ='Book'), 2) AS avg_read_duration,
       round(avgIf(hours, main_content_type ='Audiobook'), 2) AS avg_listen_duration
FROM source_db.content 
JOIN source_db.audition USING(main_content_id)
WHERE main_content_type IN ('Book', 'Audiobook')
      AND usage_platform_ru IN ('Букмейт iOS', 'Букмейт Android')
GROUP BY main_content_name, main_author_name
HAVING countDistinct(main_content_type)=2
ORDER BY total_duration DESC
LIMIT 5;
```

main_content_name                                                    |main_author_name|total_duration|avg_read_duration|avg_listen_duration|
---------------------------------------------------------------------+----------------+--------------+-----------------+-------------------+
Илон Маск                                                            |Уолтер Айзексон |       1012.93|             0.29|               0.69|
Железное пламя                                                       |Ребекка Яррос   |        781.16|             1.74|               1.89|
Убийства и кексики. Детективное агентство «Благотворительный магазин»|Питер Боланд    |        541.87|             0.68|               1.63|
Четвертое крыло                                                      |Ребекка Яррос   |        501.67|             1.58|               1.34|
Земля лишних. Трилогия                                               |Андрей Круз     |        481.97|             2.47|               2.75|

###  Task 3

Analyze the top 10 authors by total reading time across all platforms, including web. Include the number of unique text books per author and the average listening time of their audiobooks on mobile devices. Exclude authors without audiobooks.


```python
SELECT main_author_name,
       round(sumIf(hours, main_content_type = 'Book'), 2) AS total_read_duration,   
       uniqExactIf(main_content_id, main_content_type = 'Book') AS total_books,
       round(avgIf(hours, main_content_type = 'Audiobook'
       AND usage_platform_ru IN ('Букмейт iOS', 'Букмейт Android')), 2) AS avg_listen_duration
FROM source_db.content
JOIN source_db.audition USING (main_content_id)
WHERE main_content_type IN ('Book', 'Audiobook')
      AND usage_platform_ru IN ('Букмейт iOS', 'Букмейт Android', 'Букмейт Web')
GROUP BY main_author_name
HAVING uniqExactIf(main_content_id, main_content_type = 'Audiobook') > 0   
ORDER BY total_read_duration DESC
LIMIT 10;

```

main_author_name   |total_read_duration|total_books|avg_listen_duration|
-------------------+-------------------+-----------+-------------------+
Александра Лисина  |            1558.35|71         |               2.27|
Дарья Донцова      |            1498.96|163        |               1.91|
Константин Муравьёв|              834.4|24         |               2.67|
Елена Звёздная     |             814.96|43         |               1.69|
Сергей Лукьяненко  |             813.62|54         |               1.76|
Робин Хобб         |             687.22|18         |               1.29|
Виктор Пелевин     |              633.7|30         |               0.96|
Ребекка Яррос      |              565.9|2          |               1.67|
Татьяна Устинова   |             545.47|63         |               1.53|
Макс Фрай          |             500.54|38         |                1.3|

### Task 4

Segment mobile users based on their content preferences. Identify users as:

- Listener - primarily uses audiobooks (≥70% of total session time).
- Reader - primarily uses text books (≥70% of total session time).
- Both - all other users.

Exclude users without any sessions in books or audiobooks. Determine the number of users in each segment and verify whether Android users have roughly equal numbers of readers and listeners, while iOS users have about twice as many readers as listeners, using the user’s main platform by total time.

SELECT 
    main_platform,
    CASE
        WHEN audiobook_hours / total_hours >= 0.7 THEN 'Listener'
        WHEN book_hours / total_hours >= 0.7 THEN 'Reader'
        ELSE 'Both'
    END AS segment,
    count() AS user_count
FROM (
    SELECT
        puid,
        sumIf(hours, main_content_type = 'Book') AS book_hours,
        sumIf(hours, main_content_type = 'Audiobook') AS audiobook_hours,
        sumIf(hours, main_content_type IN ('Book', 'Audiobook')) AS total_hours,
        if(
            sumIf(hours, usage_platform_ru = 'Букмейт iOS') > sumIf(hours, usage_platform_ru = 'Букмейт Android'),
            'Bookmate iOS',
            'Bookmate Android'
        ) AS main_platform

    FROM source_db.audition
    JOIN source_db.content USING (main_content_id)
    WHERE main_content_type IN ('Book', 'Audiobook')
      AND usage_platform_ru IN ('Букмейт iOS', 'Букмейт Android')
    GROUP BY puid
    HAVING total_hours > 0
)
GROUP BY main_platform, segment
ORDER BY main_platform, segment;

main_platform   |segment |user_count|
----------------+--------+----------+
Bookmate Android|Both    |396       |
Bookmate Android|Listener|2145      |
Bookmate Android|Reader  |2381      |
Bookmate iOS    |Both    |285       |
Bookmate iOS    |Listener|1508      |
Bookmate iOS    |Reader  |1997      |

### Task 5

Investigate whether app usage format (reading or listening) is related to the day of the week. Compute average usage hours for each content type on weekdays vs weekends, across all platforms including web. Round results to whole numbers.

WITH daily_stats AS (
    SELECT toDayOfWeek(msk_business_dt_str) AS day_of_week,
           main_content_type,
           round(sum(hours)) AS total_hours
    FROM source_db.audition
    JOIN source_db.content USING (main_content_id)
    WHERE main_content_type IN ('Book', 'Audiobook')
          AND usage_platform_ru IN ('Букмейт iOS', 'Букмейт Android', 'Букмейт Web')
    GROUP BY day_of_week, main_content_type
),
averages AS (
    SELECT main_content_type,
           round(avgIf(total_hours, day_of_week NOT IN (6,7))) AS avg_weekday_hours,
           round(avgIf(total_hours, day_of_week IN (6,7))) AS avg_weekend_hours
    FROM daily_stats
    GROUP BY main_content_type
)
SELECT main_content_type,
       avg_weekday_hours,
       avg_weekend_hours,
       avg_weekend_hours - avg_weekday_hours AS diff_hours
FROM averages
ORDER BY main_content_type;

main_content_type|avg_weekday_hours|avg_weekend_hours|diff_hours|
-----------------+-----------------+-----------------+----------+
Audiobook        |          20559.0|          16962.0|   -3597.0|
Book             |          11444.0|          10838.0|    -606.0|

### Task 6

Analyze app update adoption on Android and iOS. Determine the percentage of users on each platform using the latest app version by comparing the user’s last active version with the latest platform version. Round percentages to two decimal places.

WITH last_session_per_user AS (
    SELECT puid,
           usage_platform_ru AS user_last_platform,
           app_version AS user_last_version,
           ROW_NUMBER() OVER (PARTITION BY puid ORDER BY toDateTime(msk_business_dt_str) DESC) AS rn
    FROM source_db.audition
),
max_version_per_platform AS (
    SELECT usage_platform_ru,
           MAX(app_version) AS platform_last_version
    FROM source_db.audition
    WHERE usage_platform_ru IN ('Букмейт iOS','Букмейт Android')
    GROUP BY usage_platform_ru
)
SELECT u.user_last_platform AS platform,
       ROUND(100.0 * SUM(CASE WHEN u.user_last_version = p.platform_last_version THEN 1 ELSE 0 END)
             / COUNT(*),2) AS pct_last_version
FROM last_session_per_user u
JOIN max_version_per_platform p
    ON u.user_last_platform = p.usage_platform_ru
WHERE u.rn = 1 AND u.user_last_platform IN ('Букмейт iOS','Букмейт Android')
GROUP BY u.user_last_platform
ORDER BY platform;

platform       |pct_last_version|
---------------+----------------+
Букмейт Android|           32.32|
Букмейт iOS    |            2.16|

### Task 7

Examine the frequency of app updates on each platform. Count an update as any increase in version per user. Compute the update_rate (average number of updates per user) and round to two decimal places. Compare iOS and Android users to check if iOS users update more frequently.

WITH version_changes AS (
    SELECT puid,
           usage_platform_ru,
           app_version,
           lagInFrame(app_version, 1)
           OVER (PARTITION BY puid, usage_platform_ru ORDER BY msk_business_dt_str) AS prev_version
    FROM source_db.audition
    WHERE usage_platform_ru IN ('Букмейт iOS','Букмейт Android')
)
SELECT usage_platform_ru,
       round(countIf(app_version != prev_version AND prev_version IS NOT NULL)
        /uniqExact(puid), 2) AS update_rate
FROM version_changes
GROUP BY usage_platform_ru;  

usage_platform_ru|update_rate
-----------------+-----------
Букмейт iOS      |       2.43
Букмейт Android  |       3.19

### Task 8

Investigate books on magical themes that may have incorrect category tags. Identify books tagged with “Magic” but not included in fiction. Count all such books in the catalog.

SELECT uniqExact(main_content_id) AS magic_books_count
FROM source_db.content
WHERE has(published_topic_title_list, 'Магия');

magic_books_count|
-----------------+
46               |

### Task 9

Find books with the word “magic” in the title that do not have the “Magic” tag, excluding fiction books. Count these books in the catalog.

SELECT uniqExact(main_content_id) AS num_books_missing_magic_tag
FROM source_db.content
WHERE main_content_name ILIKE '%Магия%'
      AND NOT has(published_topic_title_list, 'Магия')
      AND NOT has(published_topic_title_list, 'Художественная литература');

num_books_missing_magic_tag|
---------------------------+
49                         |

### Task 10

Compute the average number of categories for books tagged “Magic” and for all books in the catalog. Round values to two decimal places. Assess whether the number of categories per book exceeds the recommended 3–4 categories.

SELECT round(avg(length(published_topic_title_list)), 2) AS avg_tags_all_books,
       round(avgIf(length(published_topic_title_list),
       has(published_topic_title_list, 'Магия')), 2) AS avg_tags_magic_books
FROM source_db.content

avg_tags_all_books|avg_tags_magic_books|
------------------+--------------------+
              3.77|                3.22|

### Task 11

Detect anomalies in mobile session lengths (hours_sessions_long) using the coefficient of variation (standard deviation divided by mean). Analyze variation by country and mobile platform, identify the country and platform with the highest coefficient, and limit the sample to that country.

WITH cv_table AS (
    SELECT usage_country_name,
           usage_platform_ru,
           round(stddevPop(hours_sessions_long) / NULLIF(avg(hours_sessions_long), 0), 2) AS cv
    FROM source_db.audition
    WHERE usage_platform_ru IN ('Букмейт iOS', 'Букмейт Android')
    GROUP BY usage_country_name, usage_platform_ru
)
SELECT *
FROM cv_table
WHERE usage_country_name = (SELECT usage_country_name FROM cv_table ORDER BY cv DESC LIMIT 1)
      AND usage_platform_ru IN ('Букмейт iOS', 'Букмейт Android')
ORDER BY cv DESC;

usage_country_name|usage_platform_ru|cv  |
------------------+-----------------+----+
Латвия            |Букмейт Android  |7.77|
Латвия            |Букмейт iOS      |1.64|
