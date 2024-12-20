# World-Life-Expectancy-Project

# World Life Expectancy Project (Data Cleaning)


#Deleting duplicates 

-- Country and year were joined to identify duplicates - if a country name and year was found to have duplicates those would have been duplicates - since each country is represented for each year. 
SELECT * 
FROM world_life_expectancy
;


SELECT country, year, CONCAT(country, year), COUNT(CONCAT(country, year))
FROM world_life_expectancy
GROUP BY country, year, CONCAT(country, year)
HAVING COUNT(CONCAT(country, year)) > 1
;

SELECT *
FROM (
SELECT row_id,
CONCAT(country, year),
ROW_NUMBER() OVER(PARTITION BY CONCAT(country, year) ORDER BY CONCAT(country, year)) AS row_num
FROM world_life_expectancy
) AS row_table
WHERE row_num > 1
;

DELETE FROM world_life_expectancy
WHERE 
	Row_ID IN (
	SELECT Row_ID
FROM (
	SELECT row_id,
	CONCAT(country, year),
	ROW_NUMBER() OVER(PARTITION BY CONCAT(country, year) ORDER BY CONCAT(country, year)) AS row_num
	FROM world_life_expectancy
	) AS row_table
WHERE row_num > 1
)
;

# Standardizing Data


-- Missing Data
SELECT * 
FROM world_life_expectancy
WHERE status = ''
;

SELECT DISTINCT(status)
FROM world_life_expectancy
WHERE status <> ''
;

SELECT DISTINCT(country)
FROM world_life_expectancy
WHERE status = 'Developing'
;

-- This query is joining the table unto itself determining that where in table 1 the value is blank it will then be developing, but not where in table 2 the value is not blank and also has the word developing - The important thing also is that the join specifies the same country. 

UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2
	ON t1.country = t2.country
SET t1.status = 'Developing'
WHERE t1.status = ''
AND t2.status <> ''
AND t2.status = 'Developing'
;

SELECT * 
FROM world_life_expectancy
WHERE country = 'United States of America'
;

UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2
	ON t1.country = t2.country
SET t1.status = 'Developed'
WHERE t1.status = ''
AND t2.status <> ''
AND t2.status = 'Developed'
;

SELECT * 
FROM world_life_expectancy
#WHERE status = NULL
;


SELECT country, year, `Life expectancy`
FROM world_life_expectancy
#WHERE `Life expectancy` = ''
;

SELECT t1.country, t1.year, t1.`Life expectancy`, t2.country, t2.year, t2.`Life expectancy`, t3.country, t3.year, t3.`Life expectancy`,
ROUND((t2.`Life expectancy` + t3.`Life expectancy`) /2 ,1 )
FROM world_life_expectancy t1
JOIN world_life_expectancy t2
	ON t1.country = t2.country
    AND t1.Year = t2.year - 1
JOIN world_life_expectancy t3
	ON t1.country = t3.country
    AND t1.Year = t3.year + 1
WHERE t1.`Life expectancy` = ''
;

UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2
	ON t1.country = t2.country
    AND t1.Year = t2.year - 1
JOIN world_life_expectancy t3
	ON t1.country = t3.country
    AND t1.Year = t3.year + 1
SET t1.`Life expectancy` = ROUND((t2.`Life expectancy` + t3.`Life expectancy`) /2 ,1 )
WHERE t1.`Life expectancy` = ''
;

# World Life Expectancy Exploratory Data Analysis



SELECT *
FROM world_life_expectancy
;


# Here I looked at mininum life expectancy vs max life expenctancy per country

SELECT country, MIN(`Life expectancy`), 
MAX(`Life expectancy`),
ROUND(MAX(`Life expectancy`) - MIN(`Life expectancy`), 1) AS Life_Increase_15_Years
FROM world_life_expectancy
GROUP BY country
HAVING MIN(`Life expectancy`) <> 0
AND MAX(`Life expectancy`) <> 0
ORDER BY Life_Increase_15_Years ASC
;

# Here I looked at average life expectancy year by year from all countries together

SELECT Year, ROUND(AVG(`Life expectancy`),2)
FROM world_life_expectancy
WHERE `Life expectancy` <> 0
AND `Life expectancy` <> 0
GROUP BY Year
ORDER BY Year
;

SELECT *
FROM world_life_expectancy
;


# Here I looked at average life expectancy vs average GP per country - higher GDP positively correlates with higher life expectancy
SELECT country, ROUND(AVG(`Life expectancy`), 1) AS AVG_Life_Exp, ROUND(AVG(GDP),1) AS AVG_GDP
FROM world_life_expectancy
GROUP BY country
HAVING AVG_Life_Exp > 0
AND AVG_GDP > 0
ORDER BY AVG_GDP DESC
;

# Here I divided the table into high and low - to compare differences between high life expectancy / GDP vs low life expectancy / GDP - the difference is about 10 year increase in life expectancy on countries with high GDP
SELECT 
SUM(CASE WHEN GDP >= 1500 THEN 1 ELSE 0 END) High_GDP_Count,
AVG(CASE WHEN GDP >= 1500 THEN `Life expectancy` ELSE NULL END) High_GDP_Life_expectancy,
SUM(CASE WHEN GDP <= 1500 THEN 1 ELSE 0 END) Low_GDP_Count,
AVG(CASE WHEN GDP <= 1500 THEN `Life expectancy` ELSE NULL END) Low_GDP_Life_expectancy
FROM world_life_expectancy
ORDER BY GDP
;

SELECT *
FROM world_life_expectancy
;

# Here I looked at the life expectancy and status (developed-developing) - developed country had higher life expectancy
SELECT status, ROUND(AVG(`Life expectancy`),1)
FROM world_life_expectancy
GROUP BY status
;

# Here I checked the count of countries who are developed vs developing. Data is skewed because there is 32 countries developed vs 161 developing bringing the average down for developing
SELECT status, COUNT(DISTINCT Country), ROUND(AVG(`Life expectancy`),1)
FROM world_life_expectancy
GROUP BY status
;


#Here I checked life expectancy vs BMI - the correlation is weak
SELECT country, ROUND(AVG(`Life expectancy`), 1) AS AVG_Life_Exp, ROUND(AVG(BMI),1) AS AVG_BMI
FROM world_life_expectancy
GROUP BY country
HAVING AVG_Life_Exp > 0
AND AVG_BMI > 0
ORDER BY AVG_BMI ASC
;


SELECT *
FROM world_life_expectancy
;


# Here I used a window function - rolling_total to see how adult mortality increased over the years adding the previous year to the next year giving a visual representation of that increase with a final total over the 15 years
SELECT country,
Year,
`Life expectancy`,
`Adult Mortality`,
SUM(`Adult Mortality`) OVER (PARTITION BY Country ORDER BY Year) AS Rolling_total
FROM world_life_expectancy
;
