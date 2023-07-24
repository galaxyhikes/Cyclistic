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
### Refining temporary table query

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

--- Check 2: Determine if there are non-unique ride ids. This yielded 1 result with 27 rows of data with the ride id 0E+00. Thus this ride id will be excluded in the annual data table with the following clause: WHERE ride_id != "0E+00"
SELECT ride_id, COUNT(ride_id)
FROM year_data
GROUP BY ride_id
HAVING COUNT(ride_id) > 1

--- Check 3: Determine the types of bikes. This yielded 3 distinct results: electric_bike, classic_bike and docked_bike
SELECT DISTINCT rideable_type
FROM year_data

--- Check 4: Determine the types of membership. This yielded 2 distinct results: casual, member
SELECT DISTINCT member_casual
FROM year_data

--- Check 5: Determine the earliest and latest start time and date of bike usage, to ensure the data in the table is within the timeframe Mar 2022 - Feb 2023. Earliest date was 1/3/22 and latest was 28/2/23
SELECT MIN(started_at), MAX(started_at)
FROM year_data

--- Check 6: Check for invalid values of start/end date and time. Query is repeated for column 'ended_at'
SELECT *
FROM year_data
WHERE started_at NOT BETWEEN (
	SELECT MIN(started_at)
	FROM year_data)
	AND (
	SELECT MAX(started_at)
	FROM year_data)

--- Check 7: Determine if the minimum and maximum longitude and latitude geographical coordinates are within the allowable ranges. Latitude results were within -90 to +90 degrees, and longitude results were within -180 to +180 degrees
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

--- Check 8: Determine if station names and IDs are unique. Results revealed distinct station names > station IDs, thus station IDs are not unique, so station names will be used for subsequent analyses
SELECT 
	COUNT (DISTINCT start_station_name),
	COUNT (DISTINCT start_station_id),
	COUNT (DISTINCT end_station_name),
	COUNT (DISTINCT end_station_id)
FROM cyclistic.year_data

```
