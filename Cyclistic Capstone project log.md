# Context
I am a new junior data analyst in the marketing analyst team of Cyclistic, a bike-share company in Chicago. Customers are given 3 different pricing plans to utilise the programme: single-ride passes, full-day passes and annual memberships. Customers who purchase single-ride and full-day passes are casual riders, while those that purchase annual memberships are Cyclistic members. 

Finance analysts at Cyclistic concluded that annual members are more profitable than casual riders, and the director of marketing, Lily Moreno believes that maximising annual members is the key to future growth. Thus, she has set a clear goal: design marketing strategies aimed at converting casual riders into annual members. 

Primary objective of this project is to **investigate how annual members use Cyclistic bikes differently from casual riders.**

Datasets are made available by Motivate International Inc. and accessible by the public from [here]( https://divvy-tripdata.s3.amazonaws.com/index.html). According to the [license](https://ride.divvybikes.com/data-license-agreement) agreement, the datasets are collected and provided by Lyft Bike and Scooters, and Divvy is the name of the bicycle sharing service operated by them. As Cyclistic is a fictional company, there is no real data collected about its users. However, as mentioned in the project description, Divvy datasets are appropriate for the analysis. Thus, though for the purpose of this project, the analysis is done for Cyclistic, it is actually an analysis of Divvy bicycle sharing service. 

To ensure that the conclusions derived from the analysis is most reflective of the current users, the most recent datasets from the past year (March 2022 - February 2023) will be analysed. However, I quickly realised that files from May-Oct 2022 were too huge to be uploaded onto BigQuery (size limiting of 100MB). Thus I will conduct part of the cleaning process on spreadsheet, mainly to remove rows containing null values, and thereafter be imported onto BigQuery if possible.

# Step 1a: Preparing data on Excel
First, I will filter out rows with missing data, with hopes of filling them in if possible. On checking through the columns, only columns containing station information contained null values. Since each station detail comprises of its name, ID and geographical coordinates, I had hoped to fill in the missing values by cross-referencing other cells with the corresponding names/ID/coordinates. After filtering, I realised that the rows with missing start_station_name/end_station_name also had missing start_station_id/end_station_name. So I thought perhaps it would be possible to fill the 2 columns with the relevant geographical coordinates of the end and start stations. However, as I observed other complete rows, I noticed that for a unique station, there can be slight differences in the corresponding latitude and longitude coordinates. In addition, slight differences in latitude and longitude can also result in an entirely different station name and ID. Thus, I decided to omit data with missing station information be it name, ID or geographical coordinates as I will not be able to discern the precise station detail on my own.

For May 2022, there were a total of 634858 rows of data. On filtering, I found a total of 132313 rows with missing station information ie start_station_name and/or start_station_id and/or end_station_name and/or end_station_id. Thus, they were deleted (20.8% of data), which allowed me to upload the file onto BigQuery successfully. 

And that formed my basic cleaning step on Excel to remove sufficient data for the datasets to be uploaded onto BigQuery. I attempted to do the same for the remaining files in ascending order of file size - October 2022, September 2022 and stopped right there, when I realised I could have split each csv file into 2 smaller ones to upload onto BigQuery instead! I had this epiphany only when I was unable to upload the file for September 2022 even after removing empty rows. Thus, I stopped right there and started splitting the remaining csv files (for June-September) into smaller files for uploading (june_2022_part1, june_2022,part2 etc).

The following shows the proportion of rows removed relative to the total number of sets of data obtained for September and October 2022.
>*September 2022:*
- Total number of rows: 701339
- Total number of empty rows attributed to missing station information: 166194 (23.7%)

>*October 2022:*
- Total number of rows: 558685
- Total number of empty rows attributed to missing station information: 144416 (25.8%)

And with that, all data sets have been successfully uploaded onto SQL!

# Step 1b: Preparing data on BigQuery
Now that all datasets have been successfully uploaded onto BigQuery, I will clean the combined yearly data as a whole. The cleaning steps that I had previously done on excel were not very thorough and were purely for the sole purpose of allowing the upload, thus I would consider all datasets to be generally uncleaned. 

First, I want to check the proportion of data that contain null values, to understand if the data analysed is potentially underrepresented after omission. Following query was used for each month, to get an idea if the proportion of data affected is significantly different across the months.

```sql
--- Calculate proportion of rows with null values in Mar 2022
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

--- Query used for months with 2 datasets uploaded (June - Sept), with June as the example below
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

Following is the set of results representing the proportion of data containing missing values.
March 2022: 24.0%
April 2022: 26.6%
May 2022: 20.8%
June 2022: 19.4%
July 2022: 22.0%
Aug 2022: 23.0%
Sept 2022: 23.7%
Oct 2022: 25.8%
Nov 2022: 24.3%
Dec 2022: 25.5%
Jan 2023: 22.1%
Feb 2023: 21.5%

On average, around 23.2% of data will be deleted due to missing values, which is pretty consistent across the months as reflected above. Thus, each dataset should be affected to a similar extent.

Next, I want to combine all the datasets into a single table to start cleaning up the data as a whole. To do so, I made sure that the table schema (field name, data type and mode) across all tables are the same. Then I will use the following query to combine them into a single table and begin my cleaning process.

```sql
--- Creating a temporary table for the annual cyclistic data, minus rows with null values to allow for cleaning to occur
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
```

# Step 2: Data cleaning & validation
Now that I have the annual data, I first check for errors in the timestamp data using the below query with the above temporary table query.

```sql
--- Checking validity of data (timestamps)
SELECT *
FROM year_data
WHERE started_at > ended_at
```

This yielded a total of 69 results. For this result set, the timestamps are likely to placed into the wrong columns. Thus, I will be amending the fields of the 2 columns: started_at and ended_at. Using SQL to update them is rather time-consuming as a query needs to be written for each row of data obtained, so the data set obtained will be downloaded into a csv file, and edited on excel before importing into the BigQuery again (name of table: year_started_more)

Before combining this set of data into the annual data table above, I will run a query to check again if the end time is indeed after the start time. 

```sql
SELECT *
FROM cyclistic.year_started_more
WHERE started_at > ended_at
```

Above query yielded 0 result, which means that the timestamps are now in the right columns and the timestamp data is now valid! Thus, I will now combine table year_data_more into the annual cyclistic data with the following query. ^a

```sql
--- Edited temporary table query with valid timestamps
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
		FROM cyclistic.year_started_more
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
		started_at <= ended_at
	)
```

With that, I will move on to validating the data in the rest of the columns with the below queries, referencing to the temporary table query above. 

``` sql
--- Determine if there are non-unique ride ids
SELECT ride_id, COUNT(ride_id)
FROM year_data
GROUP BY ride_id
HAVING COUNT(ride_id) > 1
--- This yielded 1 result with 27 rows of data with the ride id 0E+00. Thus this ride id will be excluded in the annual data table with the following clause: WHERE ride_id != "0E+00"

--- Determine the types of bikes, which yielded 3 distinct results: electric_bike, classic_bike and docked_bike
SELECT DISTINCT rideable_type
FROM year_data

--- Determine the types of membership, which yielded 2 distinct results: casual, member
SELECT DISTINCT member_casual
FROM year_data

--- Determine the earliest and latest start time and date of bike usage, to ensure the data in the table is within the timeframe Mar 2022 - Feb 2023. Earliest date was 1/3/22 and latest was 28/2/23
SELECT MIN(started_at), MAX(started_at)
FROM year_data

--- Check for invalid values for start date and time for bike use
SELECT *
FROM year_data
WHERE started_at NOT BETWEEN (
	SELECT MIN(started_at)
	FROM year_data)
	AND (
	SELECT MAX(started_at)
	FROM year_data)

--- Check for invalid values for end date and time for bike use 
SELECT *
FROM year_data
WHERE ended_at NOT BETWEEN (
	SELECT MIN(ended_at)
	FROM year_data)
	AND (
	SELECT MAX(ended_at)
	FROM year_data)

--- Determine if the minimum and maximum longitude and latitude geographical coordinates are within the allowable ranges. Latitude results were within -90 to +90 degrees, and longitude results were within -180 to +180 degrees
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

--- Determine if station names and IDs are unique. Results revealed distinct station names > station IDs, thus station IDs are not unique, so station names will be used for subsequent analyses
SELECT 
	COUNT (DISTINCT start_station_name),
	COUNT (DISTINCT start_station_id),
	COUNT (DISTINCT end_station_name),
	COUNT (DISTINCT end_station_id)
FROM year_data
```

And with that, most of the validation steps have been completed! However, upon further reading on Divvy's bike sharing programme, I found out that only 2 bike types are available for usage - classic and electric bike (ebike) - rather than 3 types as suggested by the data. The difference between the 2 types lies in the parking options: classic bikes can only be docked at a Divvy station while ebikes can be parked at 3 possible locations - docked at a Divvy station or locked at e-stations or Divvy approved public bike racks. Thus, the rideable type 'docked bikes' is likely erroneous, and I will run the following query referencing temporary table above in an attempt to decipher which category these bikes belong to. 

```sql
--- Shortlisting the unique station names from both starting and end locations 
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
```

Table was subsequently saved as 'docked_unique'. ^d

Using the above data (table name: docked_unique), a geographical map of the locations containing docked bikes was [created](https://public.tableau.com/views/Cyclisticcapstone/Distinctstationswithdockedbikes?:language=en-GB&:display_count=n&:origin=viz_share_link). This was overlaid with a [separate map](https://d21xlh2maitm24.cloudfront.net/chi/CHI-E-Bike-Zone-Map.png?mtime=20210708100037) found on Divvy's website demarcating the areas with traditional and e-stations. End result as follows:

![[Overlaid docked stations.png]]
Figure 1: Locations of stations with docked bikes overlaid on Divvy's system map

Figure 1 shows that most stations associated with a docked bike are parking options for both classic and ebikes, so the discrepancy still remains unresolved and I had to look for information elsewhere. Back at the course forum page, I found a fellow learner who had emailed Divvy about the same issue, and he received the following reply from a representative:

> "_there are two kinds of bikes: electric and non-electric. The non-electric bikes are nowadays labelled as classic_bike and docked_bike. Hence, if you see docked_bike and classic_bike in the same dataset, know that you can categorise them together, as they are physically and functionally the same_".

Knowing this, the fields containing 'docked_bike' will be replaced with 'classic_bike'. However, as the UPDATE clause is not supported by BigQuery, the following query is used instead (referencing to above [[Cyclistic Capstone project log#^a|temporary table]]):

```sql
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
```

After updating the fields, it is crucial to ensure all fields containing 'docked_bikes' are no longer present. Thus, the clause "WHERE rideable_type = 'docked_bike' " was added to the above query. However, result set yielded 176402 rows with 'classic_bike' as the rideable type, which were likely the affected rows. So a new table was created (name: nil_docked) to conduct my checks using following queries:

``` sql
--- Check if the table still contains 'docked_bike'. Result set did not yield any data
SELECT *
FROM cyclistic.nil_docked
WHERE rideable_type = 'docked_bike'

--- Check if there only exists 2 bike types in table. Result set yielded 1658093 trips with electric and 2839778 with classic bikes
SELECT 
	rideable_type,
	COUNT (ride_id)
FROM cyclistic.nil_docked
GROUP BY rideable_type
```

In addition to the above checks, I want to know if the docked bike count was truly added into the classic bike count. Thus, I did a final check using the following query, referencing to the [[Cyclistic Capstone project log#^a|temporary table]].

```sql
SELECT 
	rideable_type,
	COUNT (ride_id)
FROM year_data
GROUP BY rideable_type

--- Docked bike accounted for 176402 trips and classic 2663376, making up to total 2839778 trips with classic bikes in the updated annual dataset
```

And with that, data cleaning is officially completed! 

_P.S I know there are other methods in updating the fields, I have tried creating a temporary table containing just ride_id and rideable_type, then using inner join to combine the tables but I figured the above query which I settled with is more succinct.

# Step 3: Data transformation

With the data cleaned up, I want to transform some of the columns into more tangible data that we can use to analyse the differences between the 2 membership types before saving the table for further analyses. Given start and end time, I want to find out the duration of each trip, which can be obtained from the difference between start and end time. 

Aside from duration, distance is another useful information that is key to characterise each trip. However, with only starting and ending station geographical coordinates, only displacement can be calculated, which is not representative of the trips taken by users, especially where detours or round trips are taken. Thus, I will use the following query to create a permanent table titled ==year_data_new== for the subsequent analyses:

```sql
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
FROM year_data --- Referencing temporary table query above
```

Aside from that, I have also retained a copy of the data without any amendments made to the rideable types and saved it as year_data (similar query to above minus the CASE clause).

# Step 4: Data analysis
Before delving into the data analysis, we need to first define the objective. With the data available, I aim to compare the following between users: 
1. Bike preference 
2. Number of trips taken & bike usage
4. Number of round trips
5. Number of accidental trips/ false starts
6. Duration travelled

#### Analysis 1: False starts 
Before delving into the data, I want to understand the frequency of false starts initiated (defined by trips < 1 min). Though Divvy's website had specifically mentioned that trips below 1 min (potentially false starts or users trying to re-dock a bike to ensure it was secure) had been removed, there is still a high chance that they could have been missed out. Since they are incomplete trips, these might not be meaningful data for all analyses, but it is still interesting to look into the differences between users. 

```sql
SELECT
	member_casual,
	rideable_type,
	COUNT(ride_id) AS num_trips
FROM cyclistic.year_data_new
WHERE
	trip_min < 1
GROUP BY member_casual, rideable_type
ORDER BY member_casual
```

_Initially I had thought of further defining false starts as having identical start and end stations. However, after further considerations, I realised that it may exclude trips which are considered false starts but have different start and end stations. Thus, I have decided to broaden the scope by only defining false starts using duration.

![[Trips due to false starts.png]]
Figure 2: Trips due to False starts

Figure 2 shows members made almost twice as many false starts as casual riders. This is likely due to the pricing [plan differences](https://divvybikes.com/pricing) between users; members do not face huge penalty for starting a bike compared to a casual rider. Assuming we have a member making false starts as opposed to a casual member making a false start. Cost incurred as follows:
- Classic bikes
	- Casual riders: $1.17 
	- Divvy members: $0.36 (daily base membership fee)
- Ebikes
	- Casual riders: $1.42
	- Divvy members: $0.53

Having trip duration < 1 minute can be due to several reasons; trips may be very well be completed and not be a false start. However, there is a higher chance that the trips are incomplete rather than complete. Since total number of trips taking < 1min makes up only <2% of total trips recorded, **all further analyses shall exclude this group of data to minimise chances of duplicate trips for same user.

Regardless, having a membership plan gives riders a lot more flexibility since unlocking bikes is free whereas casual riders have to pay $1 for that. It is possible using this feature as a marketing point. However, it will not be sufficient as a standalone reason to convince riders to get the membership because they will need 10 false starts a month to break even, and that is a rather unlikely occurrence. 

#### Analysis 2: Rider type
Now, I will move into the analysis proper. First, I want to find the difference in total trips taken by the 2 different users. Query as follows:
```sql
SELECT 
	member_casual,
	COUNT (ride_id) AS num_trips
FROM cyclistic.year_data_new
WHERE trip_min >= 1
GROUP BY member_casual
```

![[Proportion of trips taken by riders.png]]
Figure 3: Proportion of trips taken by riders

Figure 3 shows that throughout the year, members made more trips than casual riders. This suggests that in the subsequent analyses, there is a high possibility to find members generally taking more trips than casual riders.

#### Analysis 3: Bike preference
Next, I want to know if there was any preference for the 2 bikes offered on the programme. Thus, I will further stratify the riders into the type of bikes with the following query:

```sql
SELECT
	member_casual,
	rideable_type,
	COUNT(ride_id) AS num_trips
FROM cyclistic.year_data_new
WHERE trip_min >= 1
GROUP BY member_casual, rideable_type
```
![[Bike preference of riders.png]]
Figure 4: Bike preference between riders

Figure 4 shows that regardless of membership status, classic bike was preferred. In fact, the difference is even more significant for a member than a casual rider. The most probable reason for this observation is likely due the difference in price between the 2 bikes even though an electric bike offers greater parking flexibility and better ride features compared to a classic one. 

#### Analysis 4: Time analysis of trips
Now that I have an overview of the membership type and bike preference, I want to expand the understanding of the trips by breaking it down into monthly, weekly and hourly analyses. 

```sql
--- Monthly analysis
SELECT 
	member_casual,
	COUNT(ride_id) AS num_trips,
	CONCAT(FORMAT_TIMESTAMP('%b', started_at), ' ', EXTRACT(YEAR FROM started_at)) AS month_year
FROM cyclistic.year_data_new
WHERE trip_min >= 1 
GROUP BY member_casual, month_year
ORDER BY month_year

--- Weekly analysis
SELECT 
	member_casual,
	COUNT(ride_id) AS num_trips,
	FORMAT_TIMESTAMP('%A', started_at) AS day_week
FROM cyclistic.year_data_new
WHERE trip_min >= 1 
GROUP BY member_casual, day_week
ORDER BY day_week

--- Hourly analysis
SELECT 
	member_casual,
	COUNT(ride_id) AS num_trips,
	EXTRACT(HOUR FROM started_at) AS hour,
FROM cyclistic.year_data_new
WHERE trip_min >= 1
GROUP BY member_casual, hour
ORDER BY hour
```
![[Total trips taken on monthly basis.png]]
Figure 5: Total trips taken on monthly basis

Figure 5 shows that members generally took more trips than casual riders, which is consistent with findings from Figure 3. The observed trend for both groups of riders is likely seasonal; lower usage of bikes during the colder months November to April and higher usage during the warmer months May to October. This indicates that marketing campaigns should be launched within the months of May-October when casual riders are more active. 

![[Total trips taken on weekly basis.png]]
Figure 6: Total trips taken on weekly basis

Though members generally took more trips than casual riders, as seen from Figure 6, casual riders actually took more trips than members on the weekends. This interesting finding can potentially be incorporated into the marketing campaigns to convince casual riders to purchase the membership.

![[Total trips taken on hourly basis.png]]
Figure 7: Total trips taken on hourly basis

As expected, members generally took more trips than casual riders across the day. In terms of similarities, both riders experience a peak in number of trips taken in the morning at 8am, 12pm and again at 5pm. These are timings consistent with the peak hour commutes in the morning/evening when people go to/head home from work and the lunch hour when people head out for lunch. However, the spikes are a lot more significant for members than casual riders, suggesting that members likely comprise of the working class or students who have fixed daily schedules. 

On the other hand, the peaks observed for casual riders are smaller, suggesting that the proportion of working class/ students within this group is smaller compared to members. This is further substantiated with the continuous rise in usage after the morning peak hour, which is likely contributed by tourists or other locals using the bikes for recreational purposes. Since there is a group of people contributing to the upward trend in number of trips taken between 9am and 5pm, it may be beneficial to further understand the nature of these trips taken.

#### Analysis 5: Duration analysis
In addition to understanding the total trips taken, it will be useful to understand the duration of these trips since the bikes are charged per minute of usage. ^b

```sql
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
```

Above query revealed the longest trip took over 34000 minutes, which is equivalent to 23.9 days. Generally, it is rather unlikely that any user will be riding for more than 24 hours consecutively. The long trip durations recorded are likely a result of bikes not docked properly, and so the trip 'continued' until it was eventually terminated. Due to inaccurate end time for these trips, trips extending beyond 24h will be excluded in the duration analysis as maximum and average duration can be significantly affected. However, all previous analyses will not exclude these trips as they still represent useful data, and have lower probabilities of being false starts. Thus, I will run the [[Cyclistic Capstone project log#^b|above query]] again, with the WHERE clause 'trip_min between 1 and 1440 minutes'.

In addition, based on analysis 4, there were notable differences between riders when trips taken were arranged on a weekly and hourly basis. Thus, the average and median durations will also be calculated on these 2 fronts.

```sql
--- Average and median travel duration across the week
SELECT
	member_casual,
	AVG(trip_min) AS avg_trip,
	APPROX_QUANTILES(trip_min, 2)[OFFSET(1)] AS median_trip,
	FORMAT_TIMESTAMP('%A', started_at) AS day_week,
	rideable_type
FROM cyclistic.year_data_new
WHERE trip_min BETWEEN 1 AND 1440
GROUP BY member_casual, rideable_type, day_week

--- Average and median travel duration within the day
SELECT
	member_casual,
	AVG(trip_min) AS avg_trip,
	APPROX_QUANTILES(trip_min, 2)[OFFSET(1)] AS median_trip,
	EXTRACT(HOUR FROM started_at) AS hour,
	rideable_type
FROM cyclistic.year_data_new
WHERE trip_min BETWEEN 1 AND 1440
GROUP BY member_casual, rideable_type, hour
```

![[Overall average travel duration.png]]
Figure 8: Overall average travel duration

![[Average travel duration across the week.png]]
Figure 9: Average travel duration across the week

Above 2 figures demonstrate that casual riders generally took longer rides than members regardless of bike type. Notably, the overall average duration taken by casual riders on classic bikes is more than twice that of members on classic bikes. This is consistent with the speculation about the composition of people among members and casual riders. Working class adults and students are more likely to use these bikes to commute to and fro work/school, thus duration stays relatively constant throughout time. On the other hand, since some casual riders are likely tourists/ locals using for recreational purposes, the rides are generally longer. 

In addition, figure 9 further reveals that casual riders not only take more trips than members on the weekends (shown by Fig 3), but the average length of each trip is also longer, especially on classic bikes. This suggests that perhaps the membership plan can consider offering some free rides on the weekends for their subscribers to increase the subscription rate among casual riders. However, how should the number of free rides each member is entitled to each month be determined?

Since bike rentals are priced using the prices [here](https://divvybikes.com/pricing), a simplified calculation may be done. Median travel duration instead of average travel duration on the weekends (classic bike: 18 minutes; electric bike: 13 minutes, all of which are rounded up to nearest minute) will be used for calculations because the dataset consists of huge range in values that can skew average durations.

Let x be the number of free rides within a month
1. Assuming classic bike only
	- Total cost incurred by a casual rider: 0.17 (18) + x 
	- Break even point: 0.17 (18) + x = 10.91 
	- x = maximum 8 free rides a month (2 per week) can be offered before the company starts incurring losses
2. Assuming electric bike only
	- Total cost incurred by a casual rider: 0.42 (13) + x
	- Break even point: 0.42 (13) + x = 10.91
	- x = maximum 6 free rides a month can be offered before the company starts incurring losses

Since casual riders use classic bikes at a higher frequency than electric bikes, the classic bike model can be used as a benchmark. Thus, 2 free weekend rides per week can be offered part of the membership plan to entice people to subscribe.

In addition to the above 2 figures, the average travel duration on an hourly basis also reveals interesting findings (refer to Fig 10).

![[Average travel duration within the day.png]]
Figure 10: Average travel duration within the day

Though the average travel duration across the week stayed relatively constant, when broken down into each starting hour, a different trend is observed. Travel duration is longer between the hours of 12-4am and 9am-3pm. However, usage of bikes is not significant in the wee hours as seen in Fig 7. 9am-3pm, on the other hand, is the time period where number of trips continues increasing before the peak hour kicks in. Perhaps off-peak hour membership plan for the weekdays at a reduced price can be offered, so that this group of riders can be catered to as well. However, this group of riders will not be entitled to the free rides proposed for the base membership plan. 

In addition to the queries above, a calculation was also done to determine a cut-off point where a casual rider has a higher chance of subscribing to the membership. 
1. Assume each user type takes 1 trip per month
	- Classic bike
		- Let x be the travel duration
		- Cost incurred by casual rider = 0.17x + 1
		- Cost incurred by member = 0.17(x-45) + 10.91 
		- To break even membership fee, 0.17x + 1 = 0.17x + 3.26 
			- If a casual rider is only taking 1 ride a month, he/she is unlikely to subscribe to the plan because he/she will not break even the monthly fee regardless of travel duration. 
	- Electric bike
		- Let x be the duration taken 
		- Cost incurred by casual rider = 0.42x + 1
		- Cost incurred by member = 0.17x + 10.91
		- To break even membership fee, 0.42x + 1 = 0.17x + 10.91
		- x = 40 min (rounded up to nearest minute)
			- This indicates that when a casual rider uses an electric bike beyond 39 minutes, taking the subscription plan may be a better option. However, as seen in Fig 8-10, average and median travel duration of casual riders are between 10-20 minutes. Thus, though marketing the plan to this group of riders may have a higher chance of success, it may not be a wise use of resources.
2. Assume each user type takes 2 trips per month
	- Classic bike only
		- There are 2 possibilities for the duration taken for each ride 
			- #1 Each ride is shorter than 45 minutes
				- Let x be the total duration taken across 2 trips
				- Cost incurred by casual rider = 0.17x + 2
				- To break even membership fee, 0.17x + 2 = 10.91 
				- x = 53 minutes (rounded up to nearest minute), thus on average each ride is > 26 minutes 
					- This indicates that when a casual rider takes at least 2 rides within the month with each ride spanning more than 26 minutes on average, the subscription plan becomes a viable option. 26 minutes is also within the range of average durations casual riders took across the months. However, the median travel duration is shorter between 10-20 minutes, so targeting riders travelling >26 minutes may not be most ideal.
			- #2 One ride is longer than 45 minutes
				- This scenario will not be further explored since the proportion of riders with travel durations beyond 45 minutes is small (~5%).
	- Electric bike only
		- Let x be the total duration taken across 2 trips
		- Cost incurred by casual rider = 0.42x + 2
		- Cost incurred by member = 0.17x + 10.91
		- To break even membership fee, 0.42x + 2 = 0.17x + 10.91
		- x = 36 minutes (rounded up to nearest minute), thus on average each ride is more than 17 minutes
			- This indicates that when a casual rider takes at least 2 rides within the month with each ride spanning more than 17 minutes on average, the subscription plan becomes a viable option. Since casual riders prefer classic bikes to electric bikes to a much greater extent, focusing on this group of riders will not be an effective use of resources.
	- Electric bike and classic bike 
		- #1 Assuming duration taken for each trip is same, let it be x
			- Cost incurred by casual rider = 0.42x + 0.17x + 2
			- Cost incurred by member = 0.17x + 10.91 (no need to consider the price incurred by classic bike as each trip will be within 45 minutes)
			- To break even membership fee, 0.59x + 2 = 0.17x + 10.91
			- x = 22 minutes (rounded up to nearest minute)
				- This indicates that when a casual rider uses different bikes on 2 occasions, with each ride spanning spanning more than 21 minutes, the subscription plan becomes a viable option. 
		- #2 If classic bike trip takes longer than 45 minutes, let x be duration of ebike trip, and y be duration of classic bike trip
			- Cost incurred by casual rider: 0.42x + 0.17y + 2
			- Cost incurred by member: 0.17x + 0.17 (y-45) + 10.91
			- To break even membership fee, 0.25x = 1.26
			- x = 6 minutes(rounded up to nearest minute)
				- This indicates that when a casual rider uses a classic bike for durations > 45 minutes, if he/she decides to use an electric bike for the second ride, a trip taking more than 5 minute will cause total expense to exceed that of membership fee. However, as previously mentioned, only ~5% of casual riders had trips >45 minutes. Furthermore, 5 minutes is too short a duration; casual riders are unlikely to subscribe unless they are already taking at least 10 trips within the month.
		- #3 If classic bike trip is within 45 minutes, let x be duration of ebike trip, y be duration of classic bike trip
			- Cost incurred by casual rider: 0.42x + 0.17y + 2
			- Cost incurred by member: 0.17x + 10.91
			- To break even membership fee, 0.25x + 0.17y = 8.91 --> y = 52.41 - 1.47x 
				- Unlike #2, the membership fee will only be broken even if the duration of the rides using the 2 different bikes follow the above relationship. The longer the classic bike ride, the shorter the ride needed for ebike to break even the membership fee. However, there is too much ambiguity in this scenario, thus, disregarded.

Given the above scenarios, there are 2 scenarios where casual riders may be more easily convinced to subscribe to the membership plan. With durations 21 and 26 minutes, 20 minutes may be a good cut-off point to work with. Thus, the company can consider creating marketing campaigns for riders taking at least 1 ride beyond 20 minutes, emphasising on the potential cost savings with the subscription. This can be done within the app after the completion of first trip beyond 20 minutes. 

```sql
--- Calculating the proportion of casual riders riding >20 minutes, which is 34.4%
SELECT
	COUNTIF(trip_min BETWEEN 20 AND 1440) / COUNTIF(trip_min BETWEEN 1 AND 1440) AS proportion
	FROM cyclistic.year_data_new
WHERE member_casual = 'casual'
```

However, it can be argued that perhaps the company will be better off keeping these users as casual riders as the earnings are potentially greater. But by converting this group of riders towards an annual membership, it can help the company expand its user base. Besides, the effect on cost earnings are uncertain since it is ultimately dependent on the trips taken by the riders. 

An additional point for consideration is to modify the current pricing model for casual riders. Perhaps a tiered pricing model may be adopted; a fixed price is charged for the first 30 minutes of each trip on the classic bike regardless of the actual duration taken, then impose a price per minute after the first 30 minutes. Majority of users would have completed their trips within 30 minutes anyway, and this may provide greater incentive to switch to a membership plan since riders can take unlimited riders within 45 minutes in a day without incurring charge. 

```sql
--- To justify majority of casual riders ride <=30 minutes on classic bike (74.6%)
SELECT
	COUNTIF(trip_min BETWEEN 1 AND 30) / COUNTIF(trip_min BETWEEN 1 AND 1440) AS proportion
	FROM cyclistic.year_data_new
WHERE member_casual = 'casual' AND rideable_type = 'classic_bike'
```

#### Analysis 6: Station popularity
Next, I want to understand the differences in station popularity to understand the cycling patterns of the 2 rider types. Table named '[[Cyclistic Capstone project log#^d|docked_unique]]' is referenced here. 

```sql
--- Creating a temporary with all starting and ending stations combined
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
SELECT d.lat, d.lng, s.station_pop, s.station_name
FROM (SELECT COUNT(ride_id) AS total_visit, station_name
	FROM stations
	WHERE member_casual = 'casual'
	GROUP BY station_name, station_name
	ORDER BY station_pop DESC
	LIMIT 50) AS s
JOIN cyclistic.docked_unique AS d ON s.station_name = d.station_name
```

Duplicate station names (with distinct geographical coordinates) are removed from each table then combined on Excel before [visualization](https://public.tableau.com/views/Cyclisticcapstone/Sheet12?:language=en-GB&:display_count=n&:origin=viz_share_link) .
<iframe src="https://public.tableau.com/views/Cyclisticcapstone/Sheet12?:language=en-GB&:display_count=n&:origin=viz_share_link" width="800" height="600"></iframe>
Based on the above visualisation, it is evident that there are overlaps between the hotspots frequented by casual riders and members. However, the stations most frequently visited by casual riders are skewed a lot more towards the area along Lake Michigan with beaches, museums, planetarium and aquarium. On the other hand, the areas most frequented by members are mostly within the city in the working districts. This further supports the speculation about the composition of people in each rider group. Thus, to better convince casual riders who are riding for recreational uses to subscribe to the membership plan, perhaps partnerships can be established with the businesses within the hotspots. An example is to provide members with discounted entrance fees for tourist attractions in the area. 

# Step 5: Recommendations
Overall recommendations based on the analyses done as follow:
1. Marketing campaigns should be launched within the months of May-October when casual riders are more active.
2. 2 free weekend rides per week can be offered part of the membership plan to entice casual riders to subscribe.
	- Exact number of free rides per week can be re-assessed based on profit margin. Alternatively, weekend only membership is also a plausible option. However, that may complicate the number of membership options available.
3. In addition to the base membership plan which is currently available, a cheaper off-peak (weekdays 9am - 3pm ONLY) membership plan is an option that can be offered to casual riders. 
4. Marketing messages on the app that riders need to use can be triggered upon the completion of the first 20 minutes ride of the month; informing them about the potential cost savings if they were to subscribe to the membership plan plan instead of taking single rides each time. 
5. A tiered pricing model may be adopted; a fixed price is charged for the first 30 minutes of each trip on the classic bike regardless of the actual duration taken, then impose a price per minute after the first 30 minutes.
6. Partnerships with businesses along Lake Michigan can be established to provide members discounted fees/ bundle deals for the attractions/ food and services in the area.


Referencing: 
https://github.com/AryaSK98/Cyclistic-Case-Study 
