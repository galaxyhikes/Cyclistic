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

--- 
SELECT 
	rideable_type,
	COUNT (ride_id)
FROM year_data
GROUP BY rideable_type
```
