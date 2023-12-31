### Checking proportion of data containing null values

```sql
--- Calculation of proportion of rows containing null values in datasets with 1 table uploaded, with March 2022 as an example
SELECT 
	(COUNT (*)/284042)
FROM 
	cyclistic.mar_2022
WHERE 
	start_station_name IS NULL OR 
	start_station_id IS NULL OR 
	ride_id IS NULL OR 
	end_station_name IS NULL OR 
	end_station_id IS NULL OR 
	start_lat IS NULL OR 
	start_lng IS NULL OR 
	end_lat IS NULL OR 
	end_lng IS NULL OR 
	ride_id IS NULL OR
	rideable_type IS NULL OR
	started_at IS NULL OR
	ended_at IS NULL OR 
	member_casual IS NULL

--- Calculation of proportion of rows containing null values in datasets with 2 tables uploaded, with June 2022 as an example
SELECT
	(COUNT (*)/769204)
FROM (
	SELECT *
	FROM cyclistic.june_2022_part1 
	UNION DISTINCT 
	SELECT *
	FROM cyclistic.june_2022_part2
	)
WHERE 
	start_station_name IS NULL OR 
	start_station_id IS NULL OR 
	ride_id IS NULL OR 
	end_station_name IS NULL OR 
	end_station_id IS NULL OR 
	start_lat IS NULL OR 
	start_lng IS NULL OR 
	end_lat IS NULL OR 
	end_lng IS NULL OR 
	ride_id IS NULL OR
	rideable_type IS NULL OR
	started_at IS NULL OR
	ended_at IS NULL OR 
	member_casual IS NULL
```

### Creating (first) temporary table for annual cyclistic data

```sql
--- First draft of temporary table without null values 
WITH year_data AS(
	SELECT *
	FROM (
		SELECT *
		FROM cyclistic.mar_2022
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.apr_2022
		UNION DISTINCT
		SELECT *
		FROM cyclistic.may_2022
		UNION DISTINCT
		SELECT *
		FROM cyclistic.june_2022_part1
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.june_2022_part2
		UNION DISTINCT
		SELECT *
		FROM cyclistic.july_2022_part1
		UNION DISTINCT
		SELECT *
		FROM cyclistic.july_2022_part2
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.aug_2022_part1
		UNION DISTINCT
		SELECT *
		FROM cyclistic.aug_2022_part2
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.sept_2022_part1
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.sept_2022_part2
		UNION DISTINCT
		SELECT *
		FROM cyclistic.oct_2022
		UNION DISTINCT
		SELECT *
		FROM cyclistic.nov_2022
		UNION DISTINCT
		SELECT *
		FROM cyclistic.dec_2022
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.jan_2023
		UNION DISTINCT
		SELECT *
		FROM cyclistic.feb_2023
		)
	WHERE 
		start_station_name IS NOT NULL AND
		start_station_id IS NOT NULL AND
		ride_id IS NOT NULL AND
		rideable_type IS NOT NULL AND
		started_at IS NOT NULL AND
		ended_at IS NOT NULL AND
		end_station_name IS NOT NULL AND
		end_station_id IS NOT NULL AND
		start_lat IS NOT NULL AND
		start_lng IS NOT NULL AND
		end_lat IS NOT NULL AND
		end_lng IS NOT NULL AND
		member_casual IS NOT NULL
	)

--- Check 1: Validity of timestamps. Result set is saved into 'year_data_more' and cleaned on Excel
SELECT *
FROM year_data
WHERE started_at > ended_at

--- Check 1: Ensuring timestamp data in 'year_data_more' is valid. Result set yielded 0 result
SELECT *
FROM cyclistic.year_started_more
WHERE started_at > ended_at
```
### Checking and refining of temporary table

```sql
--- Temporary table query draft 2 with VALID timestamp data
WITH year_data AS(
	SELECT *
	FROM (
		SELECT *
		FROM cyclistic.mar_2022
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.apr_2022
		UNION DISTINCT
		SELECT *
		FROM cyclistic.may_2022
		UNION DISTINCT
		SELECT *
		FROM cyclistic.june_2022_part1
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.june_2022_part2
		UNION DISTINCT
		SELECT *
		FROM cyclistic.july_2022_part1
		UNION DISTINCT
		SELECT *
		FROM cyclistic.july_2022_part2
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.aug_2022_part1
		UNION DISTINCT
		SELECT *
		FROM cyclistic.aug_2022_part2
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.sept_2022_part1
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.sept_2022_part2
		UNION DISTINCT
		SELECT *
		FROM cyclistic.oct_2022
		UNION DISTINCT
		SELECT *
		FROM cyclistic.nov_2022
		UNION DISTINCT
		SELECT *
		FROM cyclistic.dec_2022
		UNION DISTINCT 
		SELECT *
		FROM cyclistic.jan_2023
		UNION DISTINCT
		SELECT *
		FROM cyclistic.feb_2023
		UNION DISTINCT
		SELECT *
		FROM cyclistic.year_started_more 	 --- addition to original temporary table query
		)
	WHERE 
		start_station_name IS NOT NULL AND
		start_station_id IS NOT NULL AND
		ride_id IS NOT NULL AND
		rideable_type IS NOT NULL AND
		started_at IS NOT NULL AND
		ended_at IS NOT NULL AND
		end_station_name IS NOT NULL AND
		end_station_id IS NOT NULL AND
		start_lat IS NOT NULL AND
		start_lng IS NOT NULL AND
		end_lat IS NOT NULL AND
		end_lng IS NOT NULL AND
		member_casual IS NOT NULL AND
		started_at <= ended_at			 --- addition to original temporary table query
	)

--- Check 2: Determine if there are non-unique ride ids
SELECT ride_id, COUNT(ride_id)
FROM year_data
GROUP BY ride_id
HAVING COUNT(ride_id) > 1

--- Check 3: Determine the types of membership
SELECT DISTINCT member_casual
FROM year_data

--- Check 4: Determine the earliest and latest start time and date of bike usage
SELECT MIN(started_at), MAX(started_at)
FROM year_data

--- Check 5: Check for invalid values of start/end date and time. Query is repeated for column 'ended_at'.
SELECT *
FROM year_data
WHERE started_at NOT BETWEEN (
	SELECT MIN(started_at)
	FROM year_data)
	AND (
	SELECT MAX(started_at)
	FROM year_data)

--- Check 6: Check for invalid geographical coordinates
SELECT
	MIN(start_lat) AS min_start_lat,
	MAX(start_lat) AS max_start_lat,
	MIN(end_lat) AS min_end_lat,
	MAX(end_lat) AS max_end_lat,
	MIN(start_lng) AS min_start_lng,
	MAX(start_lng) AS max_start_lng,
	MIN(end_lng) AS min_end_lng,
	MAX(end_lng) AS max_end_lng
FROM year_data

--- Check 7: Determine if station names and IDs are unique. 
SELECT 
	COUNT (DISTINCT start_station_name),
	COUNT (DISTINCT start_station_id),
	COUNT (DISTINCT end_station_name),
	COUNT (DISTINCT end_station_id)
FROM year_data

--- Check 8: Determine the types of bikes
SELECT DISTINCT rideable_type
FROM year_data

--- Shortlisting the unique station names from both starting and end locations with 'docked bikes'. Results table was saved as 'docked_unique'
SELECT *
FROM (
	SELECT
		DISTINCT start_station_name AS station_name, 
		start_lat AS lat, 
		start_lng AS lng
	FROM year_data
	WHERE rideable_type = 'docked_bike'
	)
UNION DISTINCT
SELECT *
FROM (
	SELECT
		DISTINCT end_station_name, 
		end_lat, 
		end_lng
	FROM year_data
	WHERE rideable_type = 'docked_bike'
	)

--- Replacing 'docked_bikes' with 'classic_bikes'. Table was saved as 'nil_docked' for further checks
SELECT
	ride_id,
	started_at,
	ended_at,
	start_station_name,
	end_station_name,
	start_lat,
	start_lng,
	end_lat,
	end_lng,
	CASE WHEN rideable_type = 'docked_bike' THEN 'classic_bike'
	WHEN rideable_type = 'electric_bike' THEN 'electric_bike'
	ELSE 'classic_bike'
	END AS rideable_type,
	member_casual
FROM year_data

--- Check if there only exists 2 bike types in table 'nil_docked'. Result set yielded 1658093 trips with electric and 2839778 with classic bikes
SELECT 
	rideable_type,
	COUNT (ride_id)
FROM cyclistic.nil_docked
GROUP BY rideable_type

--- Check if the records containing 'docked_bike'  was accounted towards 'classic_bike'. Result set yielded 176402 trips with docked bikes and 2663376 with classic bikes in the original temporart table year_data, corresponding to the total count with classic bikes in updated table above
SELECT 
	rideable_type,
	COUNT (ride_id)
FROM year_data
GROUP BY rideable_type

--- Further refinement of temporary table query with new column on trip duration and updated bike types. Table generated was then saved as a permanent table
SELECT
	ride_id,
	started_at,
	ended_at,
	ROUND(TIMESTAMP_DIFF(ended_at, started_at, SECOND)/60,2) AS trip_min,
	start_station_name,
	end_station_name,
	start_lat,
	start_lng,
	end_lat,
	end_lng,
	CASE WHEN rideable_type = 'docked_bike' THEN 'classic_bike'
	WHEN rideable_type = 'electric_bike' THEN 'electric_bike'
	ELSE 'classic_bike'
	END AS rideable_type,
	member_casual
FROM year_data
```

### Data analysis
```sql
--- Analysis 1: False starts
SELECT
	member_casual,
	rideable_type,
	COUNT(ride_id) AS num_trips
FROM cyclistic.year_data_new
WHERE
	trip_min < 1
GROUP BY member_casual, rideable_type
ORDER BY member_casual

--- Analysis 2: Rider proportion
SELECT 
	member_casual,
	COUNT (ride_id) AS num_trips
FROM cyclistic.year_data_new
WHERE trip_min >= 1
GROUP BY member_casual

--- Analysis 3: Bike preference
SELECT
	member_casual,
	rideable_type,
	COUNT(ride_id) AS num_trips
FROM cyclistic.year_data_new
WHERE trip_min >= 1
GROUP BY member_casual, rideable_type

--- Analysis 4a: Trips taken on monthly basis
SELECT 
	member_casual,
	COUNT(ride_id) AS num_trips,
	CONCAT(FORMAT_TIMESTAMP('%b', started_at), ' ', EXTRACT(YEAR FROM started_at)) AS month_year
FROM cyclistic.year_data_new
WHERE trip_min >= 1 
GROUP BY member_casual, month_year
ORDER BY month_year

--- Analysis 4b: Trips taken on weekly basis
SELECT 
	member_casual,
	COUNT(ride_id) AS num_trips,
	FORMAT_TIMESTAMP('%A', started_at) AS day_week
FROM cyclistic.year_data_new
WHERE trip_min >= 1 
GROUP BY member_casual, day_week
ORDER BY day_week

--- Analysis 4c: Trips taken on hourly basis
SELECT 
	member_casual,
	COUNT(ride_id) AS num_trips,
	EXTRACT(HOUR FROM started_at) AS hour,
FROM cyclistic.year_data_new
WHERE trip_min >= 1
GROUP BY member_casual, hour
ORDER BY hour

--- Analysis 5a: Initial duration analysis of trips taken (overall)
SELECT
	member_casual,
	MIN(trip_min) AS min_trip,
	MAX(trip_min) AS max_trip,
	AVG(trip_min) AS avg_trip,
	APPROX_QUANTILES(trip_min, 2)[OFFSET(1)] AS median_trip,
	rideable_type
FROM cyclistic.year_data_new
WHERE trip_min >= 1
GROUP BY member_casual, rideable_type

--- Analysis 5b: Refined duration analysis of trips taken (overall)
SELECT
	member_casual,
	MIN(trip_min) AS min_trip,
	MAX(trip_min) AS max_trip,
	AVG(trip_min) AS avg_trip,
	APPROX_QUANTILES(trip_min, 2)[OFFSET(1)] AS median_trip,
	rideable_type
FROM cyclistic.year_data_new
WHERE trip_min BETWEEN 1 AND 1440
GROUP BY member_casual, rideable_type

--- Analysis 5c: Refined duration analysis of trips taken on a weekly basis
SELECT
	member_casual,
	AVG(trip_min) AS avg_trip,
	APPROX_QUANTILES(trip_min, 2)[OFFSET(1)] AS median_trip,
	FORMAT_TIMESTAMP('%A', started_at) AS day_week,
	rideable_type
FROM cyclistic.year_data_new
WHERE trip_min BETWEEN 1 AND 1440
GROUP BY member_casual, rideable_type, day_week

--- Analysis 5d: Refined duration analysis of trips taken on an hourly basis
SELECT
	member_casual,
	AVG(trip_min) AS avg_trip,
	APPROX_QUANTILES(trip_min, 2)[OFFSET(1)] AS median_trip,
	EXTRACT(HOUR FROM started_at) AS hour,
	rideable_type
FROM cyclistic.year_data_new
WHERE trip_min BETWEEN 1 AND 1440
GROUP BY member_casual, rideable_type, hour

--- Analysis 5 extension: Calculating the proportion of casual riders riding >20 minutes (34.4%)
SELECT
	COUNTIF(trip_min BETWEEN 20 AND 1440) / COUNTIF(trip_min BETWEEN 1 AND 1440) AS proportion
	FROM cyclistic.year_data_new
WHERE member_casual = 'casual'

--- Analysis 5 extension part 2: To justify majority of casual riders ride <=30 minutes on classic bike (74.6%)
SELECT
	COUNTIF(trip_min BETWEEN 1 AND 30) / COUNTIF(trip_min BETWEEN 1 AND 1440) AS proportion
	FROM cyclistic.year_data_new
WHERE member_casual = 'casual' AND rideable_type = 'classic_bike'

--- Analysis 6: Station popularity; creating a temporary with all starting and ending stations combined
WITH stations AS (
	SELECT *
	FROM (
		SELECT
			start_station_name AS station_name,
			ride_id
		FROM cyclistic.year_data_new
		)
	UNION ALL
	SELECT *
	FROM (
		SELECT
			end_station_name,
			ride_id
		FROM cyclistic.year_data_new
		)
)

--- Finding the most popular stations for each user type by joining with table docked_unique. Same query is then repeated by replacing the WHERE clause with 'member'
SELECT
	d.lat,
	d.lng,
	s.station_pop,
	s.station_name
FROM (
	SELECT
		COUNT(ride_id) AS total_visit,
		station_name
	FROM stations
	WHERE member_casual = 'casual'
	GROUP BY station_name, station_name
	ORDER BY station_pop DESC
	LIMIT 50
	) AS s
JOIN cyclistic.docked_unique AS d ON s.station_name = d.station_name
```
