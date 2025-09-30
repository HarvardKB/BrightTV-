# BrightTV-
Analysis of BrightTV data

SELECT
  *
FROM
  "BRIGHTTV"."PUBLIC"."BRIGHTTV"
LIMIT
  10;


  -------------------------- I wanted to combine both tables on the common column which is the user id column. 
  SELECT *
FROM user_profiles p
JOIN brighttv v
  ON p.userID = v.userID;

-------------------------------------- 

SELECT COUNT(*) AS missing_in_brighttv
FROM user_profiles p
LEFT JOIN brighttv v
  ON p.userID = v.userID
WHERE v.userID IS NULL;

-----------------------------------------------

SELECT COUNT(*) AS missing_in_profiles
FROM brighttv v
LEFT JOIN user_profiles p
  ON v.userID = p.userID
WHERE p.userID IS NULL;
  
-------------------------------------------------- This is the code where we will extract the revelant columns from each tabel while only keeping 1 user id column for which we have used the Join. 

SELECT 
    p.userID, 
    p.gender, 
    p.race, 
    p.age, 
    p.province,
    v.channel2, 
    v.recorddate2, 
    v.duration2
FROM user_profiles p
LEFT JOIN brighttv v
  ON p.userID = v.userID;

  ---------------------------------------------------------------------- I want to calculate min and max duration. 

  SELECT 
    MIN(v.duration2) AS min_duration,
    MAX(v.duration2) AS max_duration
FROM user_profiles p
LEFT JOIN brighttv v
  ON p.userID = v.userID;
--------------------------------------------------------- We wanted to make sure that the 11hr duration was real, so this code looks was to find the corresponding entry. 

SELECT *
FROM brighttv
WHERE duration2 = '11:29:28';

------------------------------------------------------------ We want to find how many unique viewers we have. These would have logged a viewiing. 

SELECT COUNT(DISTINCT userID) AS unique_viewers
FROM brighttv;
  
----------------------------------------------------------- Now we want a list of all resgistered users. Whether or not they watched something. 

SELECT COUNT(DISTINCT userID) AS total_users
FROM user_profiles;

-------------------------------------------------------------- Now we want the number of unique users whether or not they watched something 

SELECT COUNT(DISTINCT p.userID) AS unique_users
FROM user_profiles p
LEFT JOIN brighttv v
  ON p.userID = v.userID;

  ----------------------------------------------------- We want to calculate how many viewership records we have. 

  SELECT COUNT(*) AS total_transactions
FROM user_profiles p
LEFT JOIN brighttv v
  ON p.userID = v.userID;

  ---------------------------------------------------- We want to see how many viewers had more than 1 engagement

 SELECT COUNT(*) AS users_with_multiple_engagements
FROM (
    SELECT p.userID, COUNT(v.userID) AS num_transactions
    FROM user_profiles p
    LEFT JOIN brighttv v
      ON p.userID = v.userID
    GROUP BY p.userID
    HAVING COUNT(v.userID) > 1
) t;

------------------------------------------------------ I now want the date and time separately in a column 

SELECT 
    p.userID,
    p.gender,
    p.race,
    p.age,
    p.province,
    v.channel2,
    v.duration2,
    -- convert to SA time and extract date
    TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS view_date,
    -- convert to SA time and extract time
    CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME) AS view_time
FROM user_profiles p
LEFT JOIN brighttv v
  ON p.userID = v.userID;

  --------------------------------------------------------------- We now want to find earliest date and latest date

  SELECT 
    MIN(TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(recorddate2, 'YYYY/MM/DD HH24:MI')))) AS earliest_date,
    MAX(TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(recorddate2, 'YYYY/MM/DD HH24:MI')))) AS latest_date
FROM brighttv;

-------------------------------------------------------------------- We now want earliest time and latest time 

SELECT 
    MIN(CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME)) AS earliest_time,
    MAX(CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME)) AS latest_time
FROM brighttv;


-------------------------------------------------------------------------------------

   SELECT 
    MIN(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(recorddate2, 'YYYY/MM/DD HH24:MI'))) AS earliest_timestamp,
    MAX(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(recorddate2, 'YYYY/MM/DD HH24:MI'))) AS latest_timestamp
FROM brighttv;

----------------------------------------------------------------- Now we want to extract, month name, day name and day of month 

SELECT
    p.userID,
    p.gender,
    p.race,
    p.age,
    p.province,
    v.channel2,
    v.duration2,

    -- Define the base SA timestamp calculation once for clarity (TS_SA)
    -- This expression will be repeated below.
    -- (CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS TS_SA
    
    -- 1. SA Date
    TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS view_date,

    -- 2. SA Time
    CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME) AS view_time,

    -- 3. SA Month Name (Using the dedicated function to avoid TO_CHAR issues)
    MONTHNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS month_name,
    
    -- 4. SA Day of Month (Using the dedicated function)
    DAYOFMONTH(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_of_month,
    
    -- 5. SA Day Name (Using the dedicated function to avoid TO_CHAR issues)
    DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_name
    
FROM user_profiles p
LEFT JOIN brighttv v
    ON p.userID = v.userID;

    ---------------------------------------------------------


    SELECT
    p.userID,
    p.gender,
    p.race,
    p.age,
    p.province,
    v.channel2,
    v.duration2,

    -- 1. SA Date
    TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS view_date,

    -- 2. SA Time
    CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME) AS view_time,

    -- 3. SA Month Name
    MONTHNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS month_name,

    -- 4. SA Day of Month
    DAYOFMONTH(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_of_month,

    -- 5. SA Day Name (Required for the CASE statement)
    DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_name,
    
    -- 6. NEW: Day Type (Using a CASE statement)
    CASE 
        -- Check if the day name is Saturday or Sunday (case-insensitive check is safest)
        WHEN UPPER(DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')))) IN ('Sat','Sun') 
        THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type

FROM user_profiles p
LEFT JOIN brighttv v
    ON p.userID = v.userID;
    
---------------------------------------------------------------------------------- We now want to include day type so we can have a column that distinguishes weekday and weekend.

SELECT
    p.userID,
    p.gender,
    p.race,
    p.age,
    p.province,
    v.channel2,
    v.duration2,

    -- Define the working SA timestamp calculation once for reference (without SS)
    -- CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')) AS TS_SA
    
    -- 1. SA Date
    TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS view_date,

    -- 2. SA Time
    CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME) AS view_time,

    -- 3. SA Month Name
    MONTHNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS month_name,

    -- 4. SA Day of Month
    DAYOFMONTH(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_of_month,

    -- 5. SA Day Name (Will return 'Sat', 'Sun', 'Mon', etc.)
    DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_name,
    
    -- 6. NEW: Day Type (Checking for 'SAT' or 'SUN')
    CASE 
        -- Use SUBSTRING or a known abbreviation function if DAYNAME returns a full name, 
        -- but since you say it's 'Sat'/'Sun', we check for those directly.
        WHEN UPPER(DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')))) IN ('SAT', 'SUN')
        THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type

FROM user_profiles p
LEFT JOIN brighttv v
    ON p.userID = v.userID;

-------------------------------------------------------------------- Now we need to create time buckets

SELECT
    p.userID,
    p.gender,
    p.race,
    p.age,
    p.province,
    v.channel2,
    v.duration2,

    -- 1. SA Date
    TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS view_date,

    -- 2. SA Time
    CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME) AS view_time,

    -- 3. SA Month Name
    MONTHNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS month_name,

    -- 4. SA Day of Month
    DAYOFMONTH(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_of_month,

    -- 5. SA Day Name
    DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_name,
    
    -- 6. Day Type (Checking for 'SAT' or 'SUN')
    CASE 
        WHEN UPPER(DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')))) IN ('SAT', 'SUN')
        THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type,
    
    -- 7. Time Bucket (All 24 hours covered, no 'Unknown')
    CASE 
        -- Extract the hour (HH24) from the converted SA timestamp
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 6 AND 8 THEN 'Early Morning (06:00 – 08:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 9 AND 15 THEN 'Daytime (09:00 – 15:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 16 AND 19 THEN 'Early Evening (16:00 – 19:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 20 AND 22 THEN 'Prime Time (20:00 – 22:59)'
        -- Late Night (23:00 to 00:59) covers hours 23 and 0
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) IN (23, 0) THEN 'Late Night (23:00 – 00:59)'
        -- Overnight (01:00 to 05:59) covers the remaining hours (1 to 5)
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 1 AND 5 THEN 'Overnight (01:00 – 05:59)'
    END AS time_bucket

FROM user_profiles p
LEFT JOIN brighttv v
    ON p.userID = v.userID;


--------------------------------------------------------------------- We now add duration bucketing: 

SELECT
    p.userID,
    p.gender,
    p.race,
    p.age,
    p.province,
    v.channel2,
    v.duration2,

    -- 1. SA Date
    TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS view_date,

    -- 2. SA Time
    CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME) AS view_time,

    -- 3. SA Month Name
    MONTHNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS month_name,

    -- 4. SA Day of Month
    DAYOFMONTH(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_of_month,

    -- 5. SA Day Name
    DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_name,
    
    -- 6. Day Type (Checking for 'SAT' or 'SUN')
    CASE 
        WHEN UPPER(DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')))) IN ('SAT', 'SUN')
        THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type,
    
    -- 7. Time Bucket
    CASE 
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 6 AND 8 THEN 'Early Morning (06:00 – 08:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 9 AND 15 THEN 'Daytime (09:00 – 15:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 16 AND 19 THEN 'Early Evening (16:00 – 19:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 20 AND 22 THEN 'Prime Time (20:00 – 22:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) IN (23, 0) THEN 'Late Night (23:00 – 00:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 1 AND 5 THEN 'Overnight (01:00 – 05:59)'
    END AS time_bucket,
    
    -- 8. Duration Bucket with Time Frame Included
    CASE
        WHEN v.duration2 IS NULL THEN 'Rolling / Continuous Format (Variable / Undefined)'
        WHEN v.duration2 < CAST('00:01:00' AS TIME) THEN 'Instant Exit / Drop-Off (00:00:01 – 00:00:59)'
        WHEN v.duration2 < CAST('00:03:00' AS TIME) THEN 'Very Short Engagement (00:01:00 – 00:02:59)'
        WHEN v.duration2 < CAST('00:15:00' AS TIME) THEN 'Short Form Programs (00:03:00 – 00:14:59)'
        WHEN v.duration2 < CAST('00:30:00' AS TIME) THEN 'Short-Half Hour Programs (00:15:00 – 00:29:59)'
        WHEN v.duration2 < CAST('01:00:00' AS TIME) THEN 'Standard Half-Hour Programs (00:30:00 – 00:59:59)'
        WHEN v.duration2 < CAST('02:00:00' AS TIME) THEN 'One-Hour Programs (01:00:00 – 01:59:59)'
        WHEN v.duration2 < CAST('04:00:00' AS TIME) THEN 'Feature-Length / Extended Episode (02:00:00 – 03:59:59)'
        WHEN v.duration2 < CAST('08:00:00' AS TIME) THEN 'Extended Event (04:00:00 – 07:59:59)'
        ELSE 'Marathon / All-Day (08:00:00 – 23:59:59)'
    END AS duration_bucket

FROM user_profiles p
LEFT JOIN brighttv v
    ON p.userID = v.userID;

---------------------------------------------------------------------------- We want to now add buckets age

SELECT
    p.userID,
    p.gender,
    p.race,
    p.age,
    p.province,
    v.channel2,
    v.duration2,

    -- 1. SA Date
    TO_DATE(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS view_date,

    -- 2. SA Time
    CAST(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')) AS TIME) AS view_time,

    -- 3. SA Month Name
    MONTHNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS month_name,

    -- 4. SA Day of Month
    DAYOFMONTH(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_of_month,

    -- 5. SA Day Name
    DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) AS day_name,
    
    -- 6. Day Type (Checking for 'SAT' or 'SUN')
    CASE 
        WHEN UPPER(DAYNAME(CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI')))) IN ('SAT', 'SUN')
        THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type,
    
    -- 7. Time Bucket
    CASE 
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 6 AND 8 THEN 'Early Morning (06:00 – 08:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 9 AND 15 THEN 'Daytime (09:00 – 15:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 16 AND 19 THEN 'Early Evening (16:00 – 19:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 20 AND 22 THEN 'Prime Time (20:00 – 22:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) IN (23, 0) THEN 'Late Night (23:00 – 00:59)'
        WHEN EXTRACT(HOUR FROM CONVERT_TIMEZONE('UTC', 'Africa/Johannesburg', TO_TIMESTAMP_NTZ(v.recorddate2, 'YYYY/MM/DD HH24:MI'))) BETWEEN 1 AND 5 THEN 'Overnight (01:00 – 05:59)'
    END AS time_bucket,
    
    -- 8. Duration Bucket with Time Frame Included
    CASE
        WHEN v.duration2 IS NULL THEN 'Rolling / Continuous Format (Variable / Undefined)'
        WHEN v.duration2 < CAST('00:01:00' AS TIME) THEN 'Instant Exit / Drop-Off (00:00:01 – 00:00:59)'
        WHEN v.duration2 < CAST('00:03:00' AS TIME) THEN 'Very Short Engagement (00:01:00 – 00:02:59)'
        WHEN v.duration2 < CAST('00:15:00' AS TIME) THEN 'Short Form Programs (00:03:00 – 00:14:59)'
        WHEN v.duration2 < CAST('00:30:00' AS TIME) THEN 'Short-Half Hour Programs (00:15:00 – 00:29:59)'
        WHEN v.duration2 < CAST('01:00:00' AS TIME) THEN 'Standard Half-Hour Programs (00:30:00 – 00:59:59)'
        WHEN v.duration2 < CAST('02:00:00' AS TIME) THEN 'One-Hour Programs (01:00:00 – 01:59:59)'
        WHEN v.duration2 < CAST('04:00:00' AS TIME) THEN 'Feature-Length / Extended Episode (02:00:00 – 03:59:59)'
        WHEN v.duration2 < CAST('08:00:00' AS TIME) THEN 'Extended Event (04:00:00 – 07:59:59)'
        ELSE 'Marathon / All-Day (08:00:00 – 23:59:59)'
    END AS duration_bucket,

    -- 9. NEW: Age Bucket
    CASE
        WHEN p.age BETWEEN 0 AND 12 THEN 'Kids / Tweens (0–12)'
        WHEN p.age BETWEEN 13 AND 17 THEN 'Teens (13–17)'
        WHEN p.age BETWEEN 18 AND 24 THEN 'Young Adults (18–24)'
        WHEN p.age BETWEEN 25 AND 34 THEN 'Adults (25–34)'
        WHEN p.age BETWEEN 35 AND 44 THEN 'Mid Adults (35–44)'
        WHEN p.age BETWEEN 45 AND 54 THEN 'Older Adults (45–54)'
        WHEN p.age >= 55 THEN 'Seniors (55+)'
        ELSE 'Age Undefined' -- For NULL or invalid age values
    END AS age_bucket

FROM user_profiles p
LEFT JOIN brighttv v
    ON p.userID = v.userID;
