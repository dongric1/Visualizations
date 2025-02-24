
#### 1. Exit rate

```sql
WITH
  ranked_events AS (
  SELECT
    user_pseudo_id,
    page_location,
    event_name,
    DATE(TIMESTAMP_MICROS(event_timestamp)) AS event_date,
    ROW_NUMBER() OVER (PARTITION BY user_pseudo_id, DATE(TIMESTAMP_MICROS(event_timestamp))
    ORDER BY
      event_timestamp DESC ) AS rn
  FROM
    `tc-da-1.turing_data_analytics.raw_events` ),
  exits AS (
  SELECT
    page_location,
    COUNT(user_pseudo_id) AS user_count
  FROM
    ranked_events
  WHERE
    rn = 1
  GROUP BY
    page_location
  ORDER BY
    user_count DESC),
  visits_page AS (
  SELECT
    page_location,
    COUNT(user_pseudo_id) AS total_visits
  FROM
    `tc-da-1.turing_data_analytics.raw_events`
  WHERE
    page_title != 'Page Unavailable'
  GROUP BY
    ALL
  HAVING
    COUNT(user_pseudo_id) > 2 )
SELECT
  SUBSTR(t1.page_location, LENGTH('https://shop.googlemerchandisestore.com') ) AS trimmed_page_location,
  t1.page_location,
  (t2.user_count / t1.total_visits)*100 AS exit_rate
FROM
  visits_page AS t1
JOIN
  exits AS t2
ON
  t1.page_location = t2.page_location
```


#### 2. Purchase duration daily

```sql
WITH FirstVisit AS (
    SELECT
        user_pseudo_id,
        DATE(TIMESTAMP_MICROS(event_timestamp)) AS event_date,
        MIN(event_timestamp) AS first_visit_time
    FROM
        `tc-da-1.turing_data_analytics.raw_events`
    GROUP BY
        user_pseudo_id,
        event_date
),
Purchases AS (

    SELECT
        user_pseudo_id,
        DATE(TIMESTAMP_MICROS(event_timestamp)) AS event_date,
        event_timestamp AS purchase_time,
        purchase_revenue_in_usd
    FROM
        `tc-da-1.turing_data_analytics.raw_events`
    WHERE
        event_name = 'purchase'
),
CombinedEvents AS (

    SELECT
        p.user_pseudo_id,
        p.event_date,
        TIMESTAMP_MICROS(f.first_visit_time) AS first_visit_time,
        TIMESTAMP_MICROS(p.purchase_time) AS purchase_time,
        TIMESTAMP_DIFF(TIMESTAMP_MICROS(p.purchase_time), TIMESTAMP_MICROS(f.first_visit_time), SECOND) AS duration_seconds,
        p.purchase_revenue_in_usd
    FROM
        Purchases p
    INNER JOIN
        FirstVisit f
    ON
        p.user_pseudo_id = f.user_pseudo_id
        AND p.event_date = f.event_date
)
SELECT
    c.event_date,
    COUNT(c.user_pseudo_id) AS total_purchases,
    SUM(c.purchase_revenue_in_usd) AS total_revenue,
    AVG(c.duration_seconds) / 60 AS avg_duration_minutes,
    AVG(c.duration_seconds) AS avg_duration_seconds
FROM
    CombinedEvents c
GROUP BY
    c.event_date
ORDER BY
    c.event_date;

```

