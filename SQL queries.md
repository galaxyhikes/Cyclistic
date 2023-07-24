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

--- First temporary table query for the annual cyclistic data, minus rows with null values 
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

--- Checking validity of timestamps
SELECT *
FROM year_data
WHERE started_at > ended_at
```
