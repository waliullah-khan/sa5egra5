Research Question 1: Which U.S. metropolitan areas have a low density of General Medical and Surgical Hospitals?
Question 1.1: What is the count of General Medical and Surgical Hospitals in each metropolitan area?
By city:
Query:
```
SELECT city, COUNT(*) as hospital_count
FROM team12-fa24-mgmt58200-final.safegraph.places
WHERE naics_code = 622110 -- NAICS code for General Medical and Surgical Hospitals
GROUP BY city
ORDER BY hospital_count DESC;
```
Result:
Highest count is 308 in Cincinnati and 1 being the lowest count for multiple cities
By Region
```
###Query:
SELECT region, COUNT(*) as hospital_count
FROM team12-fa24-mgmt58200-final.safegraph.places
WHERE naics_code = 622110 -- NAICS code for General Medical and Surgical Hospitals
GROUP BY region
ORDER BY hospital_count DESC;
```
Result
Missouri being highest with 846 hospitals and Delaware is lowest with 8 hospitals (excluding United States Virgin Islands and GUAM being even lower with 1)
Question 1.2: What is the ratio of hospitals to population in each metropolitan area?
By city:
```
###Query:
WITH city_data AS (
  SELECT p.city,COUNT(DISTINCT p.safegraph_place_id) as hospital_count, SUM(DISTINCT c.pop_total) as total_population
  FROM team12-fa24-mgmt58200-final.safegraph.places p
  JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
  JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
  WHERE p.naics_code = 622110
  GROUP BY p.city
)
SELECT city,hospital_count,total_population,
  SAFE_DIVIDE(total_population, hospital_count) as population_per_hospital,
  SAFE_DIVIDE(hospital_count, total_population) as hospitals_per_capita    
FROM city_data
WHERE total_population > 1000
ORDER BY population_per_hospital DESC;
```
Result:
Riverview has the highest ratio of population to hospital at 7371 individuals/hospital and Crestview Hills having the lowest ratio of population to hospital 98 individuals/hospital

By Region
```
###Query:
WITH region_data AS (
  SELECT p.region,COUNT(DISTINCT p.safegraph_place_id) as hospital_count, SUM(DISTINCT c.pop_total) as total_population
  FROM team12-fa24-mgmt58200-final.safegraph.places p
  JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
  JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
  WHERE p.naics_code = 622110
  GROUP BY p.region
)
SELECT region,hospital_count,total_population,
  SAFE_DIVIDE(total_population, hospital_count) as population_per_hospital,
  SAFE_DIVIDE(hospital_count, total_population) as hospitals_per_capita    
FROM region_data
WHERE total_population > 1000
ORDER BY population_per_hospital DESC;
```
Result:
As a region, New Hampshire has the highest ratio of population to hospital at 1599 individuals/hospital and Missouri has the lowest ratio of population to hospital 98 individuals/hospital
Question 1.3: What is the average distance between hospitals in each metropolitan area?
By city
```
SELECT city, 
       AVG(distance) as avg_distance_km
FROM (
    SELECT p1.city,p1.safegraph_place_id,p2.safegraph_place_id,
           ST_Distance(ST_GEOGPOINT(p1.longitude, p1.latitude),
               ST_GEOGPOINT(p2.longitude, p2.latitude)) / 1000 as distance
    FROM team12-fa24-mgmt58200-final.safegraph.places p1
    JOIN team12-fa24-mgmt58200-final.safegraph.places p2 ON p1.city = p2.city AND p1.safegraph_place_id < p2.safegraph_place_id
    WHERE p1.naics_code = 622110 AND p2.naics_code = 622110
) subquery
GROUP BY city
ORDER BY avg_distance_km DESC;
```
Result:
Palmer has the highest average distance between hospital at 5328 km and Chicora having the lowest average distance between hospital at 0.0004 km
By Region
```
SELECT region, AVG(distance) as avg_distance_km
FROM (
    SELECT p1.region,p1.safegraph_place_id,p2.safegraph_place_id,
           ST_Distance(ST_GEOGPOINT(p1.longitude, p1.latitude),
               ST_GEOGPOINT(p2.longitude, p2.latitude)) / 1000 as distance
    FROM team12-fa24-mgmt58200-final.safegraph.places p1
    JOIN team12-fa24-mgmt58200-final.safegraph.places p2 ON p1.city = p2.city AND p1.safegraph_place_id < p2.safegraph_place_id
    WHERE p1.naics_code = 622110 AND p2.naics_code = 622110
) subquery
GROUP BY region
ORDER BY avg_distance_km DESC;
```
Result:
As a region, Maine (ME) has the highest average distance between hospital at 3247 km and Ohio having the lowest average distance between hospital at 36.7 km (excluding Puerto Rico and Hawai)
Question 1.4: What is the trend of hospital visits over time in areas with low hospital density?
By city
```
WITH low_density_cities AS (
  SELECT city
  FROM team12-fa24-mgmt58200-final.safegraph.places
  WHERE naics_code = 622110  -- NAICS code for General Medical and Surgical Hospitals
  GROUP BY city
  ORDER BY COUNT(*) ASC
  LIMIT 100  -- Select the 100 cities with the lowest number of hospitals
)
SELECT p.city,AVG(v.raw_visit_counts) as avg_weekly_visits,MIN(v.date_range_start) as start_date,MAX(v.date_range_end) as end_date
FROM team12-fa24-mgmt58200-final.safegraph.places p
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
WHERE p.naics_code = 622110
  AND p.city IN (SELECT city FROM low_density_cities)
GROUP BY p.city
ORDER BY avg_weekly_visits DESC;
```
Result:
Tuscaloosa has the highest average weekly visit at 19058 and Dewitt having the lowest average weekly visit at 1
By Region
```
WITH low_density_cities AS (
  SELECT region
  FROM team12-fa24-mgmt58200-final.safegraph.places
  WHERE naics_code = 622110  -- NAICS code for General Medical and Surgical Hospitals
  GROUP BY region
  ORDER BY COUNT(*) ASC
  LIMIT 20  -- Select the 20 cities with the lowest number of hospitals
)
SELECT p.region,AVG(v.raw_visit_counts) as avg_weekly_visits,MIN(v.date_range_start) as start_date,MAX(v.date_range_end) as end_date
FROM team12-fa24-mgmt58200-final.safegraph.places p
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
WHERE p.naics_code = 622110
  AND p.region IN (SELECT region FROM low_density_cities)
GROUP BY p.region
ORDER BY avg_weekly_visits ASC;
```
Result:
As a region, Nevada has the highest average weekly visit at 3156 vists and Montana has the lowest average weekly visit at 580 visits (excluding US Virgin Islands)
Question 1.5: What is the average dwell time in hospitals in low-density areas compared to high-density areas?
By city
Query:
```
WITH hospital_density AS (
  SELECT city, COUNT(*) as hospital_count,CASE WHEN COUNT(*) < 5 THEN 'Low' ELSE 'High' END as density
  FROM team12-fa24-mgmt58200-final.safegraph.places
  WHERE naics_code = 622110
  GROUP BY city
)
SELECT hd.density, AVG(v.median_dwell) as avg_median_dwell
FROM hospital_density hd
JOIN team12-fa24-mgmt58200-final.safegraph.places p ON hd.city = p.city
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
WHERE p.naics_code = 622110
GROUP BY hd.density;
```
Results
density	avg_median_dwell	
1	High	128.89315541031226
2	Low	108.48634280476

Average Dwell Time By city
Query:
```
WITH hospital_density AS (
  SELECT city, COUNT(*) as hospital_count,CASE WHEN COUNT(*) < 5 THEN 'Low' ELSE 'High' END as density
  FROM team12-fa24-mgmt58200-final.safegraph.places p
  WHERE naics_code = 622110
  GROUP BY city
)
SELECT p.city, AVG(v.median_dwell) as avg_median_dwell
FROM hospital_density hd
JOIN team12-fa24-mgmt58200-final.safegraph.places p ON hd.city = p.city
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
WHERE p.naics_code = 622110
GROUP BY city 
ORDER BY city ASC;
```
Result:
Gibson City has the highest average median dwell time at 900 minutes and Lake Success having the lowest average median dwell time at 5 minutes
Average Dwell Time By state
Query:
```
WITH hospital_density AS (
  SELECT region, COUNT(*) as hospital_count,CASE WHEN COUNT(*) < 5 THEN 'Low' ELSE 'High' END as density
  FROM team12-fa24-mgmt58200-final.safegraph.places p
  WHERE naics_code = 622110
  GROUP BY region
)
SELECT p.region, AVG(v.median_dwell) as avg_median_dwell
FROM hospital_density hd
JOIN team12-fa24-mgmt58200-final.safegraph.places p ON hd.region = p.region
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
WHERE p.naics_code = 622110
GROUP BY region 
ORDER BY avg_median_dwell ASC;
```
Result:
As a region, Utah has the lowest average median dwell time at 82 minutes and Wisconsin has the highest average median dwell time at 155 minutes

Research Question 4: Which of these metro areas have a large number of households with incomes over $200,000?
Question 4.1: What is the count of households with incomes over $200,000 in each metropolitan area with low hospital density?
By city
Query:
```
WITH low_hospital_density_cities AS (
  SELECT city
  FROM team12-fa24-mgmt58200-final.safegraph.places
  WHERE naics_code = 622110  -- NAICS code for General Medical and Surgical Hospitals
  GROUP BY city
  HAVING COUNT(*) < 5  -- Assuming low density is less than 5 hospitals
)
SELECT 
  p.city,
  COUNT(DISTINCT p.safegraph_place_id) as hospital_count,
  SUM(c.inc_gte200) as households_over_200k
FROM team12-fa24-mgmt58200-final.safegraph.places p
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
WHERE p.city IN (SELECT city FROM low_hospital_density_cities)
  AND p.naics_code = 622110
GROUP BY p.city
ORDER BY households_over_200k ASC;
```
Result:
Sugar Land is the city with low hospital density but high number of households whose yearly income is above 200k with 3920 households with income above 200k. Jeffersonville is a city with 2 hospitals but 0 households with income greater than 200k
By Region
```
WITH low_hospital_density_cities AS (
  SELECT region
  FROM team12-fa24-mgmt58200-final.safegraph.places
  WHERE naics_code = 622110  -- NAICS code for General Medical and Surgical Hospitals
  GROUP BY region
  HAVING COUNT(*) < 50  -- Assuming low density is less than 5 hospitals
)
SELECT 
  p.region,
  COUNT(DISTINCT p.safegraph_place_id) as hospital_count,
  SUM(c.inc_gte200) as households_over_200k
FROM team12-fa24-mgmt58200-final.safegraph.places p
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
WHERE p.region IN (SELECT region FROM low_hospital_density_cities)
  AND p.naics_code = 622110
GROUP BY p.region
ORDER BY households_over_200k ASC;
```
Result:
In the regions with low hospital density, Vermont is the region with low hospital density with only 6 hospitals and low number of households having yearly income is above 200k with 864 households(excluding DC and PR). Utah is the region with a relatively high number of hospital count with 38 hospitals and the highest number of household having income greater than 200k with 6476 households.
Question 4.2: What is the count of households with incomes over $200,000 in each metropolitan area?
By city
Query:
```
SELECT p.city, SUM(c.inc_gte200) as high_income_households
FROM team12-fa24-mgmt58200-final.safegraph.places p
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
GROUP BY p.city
ORDER BY high_income_households DESC
LIMIT 10;
```
Result:

Row	city	high_income_households
1	New York	18901940
2	Chicago	12423292
3	Brooklyn	6017733
4	Houston	4104129
5	Washington	3315405
6	Dallas	3224964
7	Seattle	3164055
8	San Antonio	2681550
9	Portland	2520751
10	Las Vegas	2439381

By Region
Query:
```
SELECT p.region, SUM(c.inc_gte200) as high_income_households
FROM team12-fa24-mgmt58200-final.safegraph.places p
JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
GROUP BY p.region
ORDER BY high_income_households DESC
LIMIT 10;
```
Results:
Row	region	high_income_households
1	NY	53584544
2	TX	34841736
3	IL	34183724
4	NJ	30379401
5	FL	28493955
6	PA	22043970
7	MA	20067270
8	VA	19588054
9	MD	16509304
10	NC	13329839

Question 4.3: What is the percentage of high-income households in each metropolitan area?
By City
Query:
```
WITH city_income_data AS (
  SELECT p.city, SUM(c.inc_gte200) as high_income_households,SUM(c.pop_total) as total_households
  FROM team12-fa24-mgmt58200-final.safegraph.places p
  JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
  JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
  GROUP BY p.city
)
SELECT city,high_income_households,total_households,SAFE_DIVIDE(high_income_households * 100.0, total_households) as high_income_percentage
FROM city_income_data
WHERE total_households > 1000  
ORDER BY high_income_percentage DESC
LIMIT 10;
```
Results:

city	high_income_households	total_households	high_income_percentage
Crystal Bay	4440	15040	29.521276595744681
North Truro	9310	32374	28.757645023784519
Rosslyn	2640	9872	26.74230145867099
North Chevy Chase	1128	4412	25.566636446056211
Vinings	3048	12780	23.849765258215964
Yarrow Point	1568	6800	23.058823529411764
Sands Point	3368	14612	23.049548316452231
Gladwyne	34578	157307	21.981221433248361
Pound Ridge	42092	196378	21.434172870688162
Weddington	4068	19056	21.347607052896727

By Region
Query:
```
WITH region_income_data AS (
  SELECT p.region, SUM(c.inc_gte200) as high_income_households,SUM(c.pop_total) as total_households
  FROM team12-fa24-mgmt58200-final.safegraph.places p
  JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
  JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
  GROUP BY p.region
)
SELECT region,high_income_households,total_households,SAFE_DIVIDE(high_income_households * 100.0, total_households) as high_income_percentage
FROM region_income_data
WHERE total_households > 1000  
ORDER BY high_income_percentage DESC
LIMIT 10;
```
region	high_income_households	total_households	high_income_percentage
CT	12328	126160	9.7717184527584013
DC	2955930	37392835	7.9050705837094188
NY	53584544	941230540	5.6930307425001319
NJ	30379401	534443220	5.6843084284987277
MA	20067270	373675357	5.3702417416838113
MD	16509304	347532981	4.7504279888762557
VA	19588054	465986737	4.2035647036022832
IL	34183724	822761335	4.1547557652305089
CA	1544	38772	3.9822552357371297
NH	2900765	75805927	3.8265675453055272

Question 4.4: What is the number of hospitals in areas with a high percentage of high-income households?
By City:
Query:
```
WITH high_income_cities AS (
  SELECT p.city,SUM(c.inc_gte200) AS high_income_households,SUM(c.pop_total) AS total_population,
    ROUND(SAFE_DIVIDE(SUM(c.inc_gte200) * 100.0, SUM(c.pop_total)),2) AS high_income_percentage
  FROM team12-fa24-mgmt58200-final.safegraph.places p
  JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
  JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
  GROUP BY p.city
  HAVING SAFE_DIVIDE(SUM(c.inc_gte200) * 100.0, SUM(c.pop_total)) > 10 
    AND SUM(c.pop_total) > 0 
)
SELECT h.city,h.high_income_percentage,COUNT(DISTINCT p.safegraph_place_id) AS hospital_count
FROM high_income_cities h
JOIN team12-fa24-mgmt58200-final.safegraph.places p ON h.city = p.city
WHERE p.naics_code = 622110
GROUP BY h.city, h.high_income_percentage
ORDER BY hospital_count ASC;
```
Results:
city	high_income_percentage	hospital_count
Nichols Hills	11.66	1
Vista	12.3	1
Mission Hills	19.38	1
Bellaire	13.42	1
Terrace Park	10.99	1
Baldwin Place	13.19	1
Hoboken	18.14	1
Coppell	11.97	1
West Lake Hills	16.73	1
Avalon	10.66	1
Roslyn	15.27	1
Bellmore	10.59	1
Bronxville	10.85	1
Lake Success	13.04	1
Roslyn Heights	11.95	1
Newtown	10.66	1
Kensington	10.07	1
Whitefish Bay	11.21	1
Westwood	10.21	1
Marco Island	11.48	1
Lake Forest	17.26	1
Grapevine	10.66	1
Trophy Club	14.05	1
Des Peres	12.23	1
Andover	10.7	1
Wantagh	10.64	1
Needham	14.62	1
Loxahatchee	12.67	1
Hinsdale	15.74	1
Southlake	15.3	1
Bloomfield Township	11.97	1
McLean	16.0	1
Kamuela	14.32	2
Sandy Springs	11.97	2
Leawood	11.95	2
Manhasset	12.73	2
Slingerlands	11.17	2
The Woodlands	15.92	2
Jericho	12.93	2
Syosset	11.75	3
Bethesda	13.33	3
Reston	11.74	3
Anchorage	11.6	3
Moorestown	12.7	5
New York	13.33	13
Birmingham	13.5	33

By Region:
Query:
```
WITH high_income_regions AS (
  SELECT p.region,SUM(c.inc_gte200) AS high_income_households,SUM(c.pop_total) AS total_population,
    ROUND(SAFE_DIVIDE(SUM(c.inc_gte200) * 100.0, SUM(c.pop_total)),2) AS high_income_percentage
  FROM team12-fa24-mgmt58200-final.safegraph.places p
  JOIN team12-fa24-mgmt58200-final.safegraph.visits v ON p.safegraph_place_id = v.safegraph_place_id
  JOIN team12-fa24-mgmt58200-final.safegraph.cbg_demographics c ON v.poi_cbg = c.cbg
  GROUP BY p.region
  HAVING SAFE_DIVIDE(SUM(c.inc_gte200) * 100.0, SUM(c.pop_total)) > 1 
    AND SUM(c.pop_total) > 0 -- 
)
SELECT h.region,h.high_income_percentage,COUNT(DISTINCT p.safegraph_place_id) AS hospital_count
FROM high_income_regions h
JOIN team12-fa24-mgmt58200-final.safegraph.places p ON h.region = p.region
WHERE p.naics_code = 622110
GROUP BY h.region, h.high_income_percentage
ORDER BY hospital_count ASC;
```
Result:
region	high_income_percentage	hospital_count
DE	2.43	8
DC	7.91	10
RI	3.11	11
VT	2.29	12
ME	2.02	27
NH	3.83	28
WY	1.5	30
NV	2.54	31
ND	2.08	40
SD	1.4	41
HI	2.86	42
ID	1.31	45
UT	2.14	49
WV	1.22	59
NM	1.49	61
MT	1.67	63
NE	1.94	68
SC	2.02	72
MA	5.37	74
MD	4.75	85
OR	2.27	105
NC	2.32	108
MS	1.23	110
NJ	5.68	111
VA	4.2	112
TN	2.2	133
IA	1.74	135
IN	1.36	139
CT	9.77	146
MN	2.66	153
KS	2.18	168
LA	1.5	168
GA	2.42	177
IL	4.15	191
OK	1.33	193
WA	3.2	203
FL	2.34	211
AL	1.23	214
KY	1.85	233
PA	2.99	260
NY	5.69	290
MI	2.22	336
TX	2.72	601
WI	1.92	602
CA	3.98	616
OH	2.06	787
MO	2.19	846

