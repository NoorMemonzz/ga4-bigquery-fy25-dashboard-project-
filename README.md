# ga4-bigquery-fy25-dashboard-project-

# GA4 BigQuery Dashboard Project

## Overview
This project analyzes Google Analytics 4 (GA4) data using BigQuery and Google Data Studio to evaluate user acquisition, engagement, retention, and conversion performance.

The analysis focuses on understanding how users move through the funnel, how acquisition drives growth, and where drop-offs occur.

---

## Business Objectives
- Analyze user acquisition by channel
- Measure conversion performance and funnel drop-offs
- Track weekly growth and engagement trends
- Analyze new user retention (cohort analysis)
- Explore demographics and device behavior

---

## Data Tables Used

### Core Tables
- Active_users_training_daily
- Weekly_active_users
- Weekly_new_users
- Weekly average engagement
- Key_events
- Event_counts
- Sessions_by_channel
- New Users By Channel
- Page_views
- Total_Revenue
- User_retention
- demographics
- device_category

### Processed Tables (Created in BigQuery)
- Conversion_summary
- demographics_top_countries
- Growth_vs_Engagement
- applicationsubmissions_bychannel
- demo (aggregated demographics)

---

## SQL Analysis & Transformations

---

### 1. Conversion Rate Calculation

```sql
SELECT 
  SUM(CASE 
        WHEN event_name = 'Application_Start' 
        THEN key_events ELSE 0 END) AS application_start,

  SUM(CASE 
        WHEN event_name = 'Application_Submissions' 
        THEN key_events ELSE 0 END) AS application_submissions,

  ROUND(
    SUM(CASE 
          WHEN event_name = 'Application_Submissions' 
          THEN key_events ELSE 0 END)
    /
    SUM(CASE 
          WHEN event_name = 'Application_Start' 
          THEN key_events ELSE 0 END)
  , 4) AS conversion_rate

FROM `steam-key-492615-c7.ga4.Key events`;
-----
Result: ~18.49% conversion rate
````

## 2. Growth vs Engagement Analysis

```
SELECT 
  a.week_number,

  DATE_ADD(DATE '2023-01-01', INTERVAL (a.week_number - 1) * 7 DAY) AS week_start_date,

  FORMAT_DATE('%b %d', DATE_ADD(DATE '2023-01-01', INTERVAL (a.week_number - 1) * 7 DAY)) AS week_label,

  SUM(a.active_users) AS weekly_active_users,
  SUM(n.new_users) AS new_users,

  ROUND(SUM(n.new_users) * 100.0 / SUM(a.active_users), 2) AS new_user_ratio,

  ROUND(AVG(e.avg_engagement_time_seconds)/60, 2) AS avg_engagement_time_minutes

FROM `steam-key-492615-c7.ga4.Weekly_active_users` a  

JOIN `steam-key-492615-c7.ga4.Weekly new users` n             
  ON a.week_number = n.week_number 

JOIN `steam-key-492615-c7.ga4.Weekly average engagement` e
  ON a.week_number = e.week_number

GROUP BY a.week_number
ORDER BY new_users DESC;
``

Trcks: Growth: new vs active users
Engagement: time spent per user
```

## 3. Demographic Aggregation 
```
CREATE OR REPLACE TABLE `steam-key-492615-c7.ga4.demo` AS

SELECT 
  country,
  SUM(active_users) AS active_users,
  SUM(new_users) AS new_users,
  SUM(engaged_sessions) AS engaged_sessions,
  SUM(event_count) AS event_count
FROM `steam-key-492615-c7.ga4.demographics`
GROUP BY country;
````

## 4. New User Retention (Cohort Analysis) --> Calculates retention rates over time

```
-- Retention for New User Cohorts

SELECT 
  cohort_week,

  ROUND(week_1 * 100.0 / week_0, 2) AS week_1_retention,
  ROUND(week_2 * 100.0 / week_0, 2) AS week_2_retention,
  ROUND(week_3 * 100.0 / week_0, 2) AS week_3_retention,
  ROUND(week_4 * 100.0 / week_0, 2) AS week_4_retention,
  ROUND(week_5 * 100.0 / week_0, 2) AS week_5_retention

FROM `steam-key-492615-c7.ga4.User retention`
ORDER BY cohort_week;
``
````
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________
****Key Insights****

Conversion

Only ~18% of users complete applications
Major drop-off between application start and form engagement


- Acquisition

Direct traffic drives majority of conversions
Organic + paid channels contribute less


- Growth Trends

New user ratio declines over time
Shift from acquisition-driven growth to returning users


- Retention

Strong drop in user retention after week 1
Indicates weak onboarding / early engagement


- Demographics & Devices

Traffic concentrated in few countries (primarily US)
Users nearly split between desktop and mobile

