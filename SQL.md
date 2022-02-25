The data set contained the number of a varity of crimes that occurred in specific Toronto Wards. I had data from 2014 to 2020. I knew I wanted to eventually total all the crimes together and sort by ward. This started with the following query. 


SELECT -- The outer query pulls the total assualts for each neighbourhood as well as calculates the percentage of assualts that occur in each ward. 
    Neighbourhood,
    Total_Assaults,
    ROUND(SUM((Total_Assaults/129892) * 100),2) As Percentage_Of_Total_Assaults
FROM
    (SELECT -- This first query totals all the assualts for each neighbourhood between 2014 to 2020 
        Neighbourhood,
        SUM(Assault_2014 + Assault_2015 + Assault_2016 + Assault_2017 + Assault_2018+ Assault_2019 + Assault_2020) AS Total_Assaults
    FROM `sql-project-341315.toronto_crime.crime_2016`
    GROUP BY 1) -- This Groups the results by Neighbourhood
GROUP BY 1,2
ORDER BY 3 DESC


I followed a similar process to get the totals for all the different crimes. Once I had the totals for all crimes, I added a new table which contained the growth percentages for each crime type by neighbourhood. This was done in EXCEL then imported into Google BigQuery. Combining this table with all the others, I made one finally query to create a table that I could then pull into Tableau. 

WITH -- Using With Clauses, I made each of the crime queries into their own tables that I would be able to join together at the end
  Assault_Table AS (
  SELECT
    Neighbourhood,
    Population,
    Total_Assaults
  FROM (
    SELECT
      Neighbourhood,
      F2020_Population_Projection AS Population,
      SUM(Assault_2014 + Assault_2015 + Assault_2016 + Assault_2017 + Assault_2018+ Assault_2019 + Assault_2020) AS Total_Assaults
    FROM
      `sql-project-341315.toronto_crime.crime_2016`
    GROUP BY
      1,
      2)
  GROUP BY
    1,
    2,
    3),
  Auto_Theft_Table AS (
  SELECT
    Neighbourhood,
    Population,
    Total_AutoTheft,
  FROM (
    SELECT
      Neighbourhood,
      F2020_Population_Projection AS Population,
      SUM(AutoTheft_2014 + AutoTheft_2015 + AutoTheft_2016 + AutoTheft_2017 + AutoTheft_2018+ AutoTheft_2019 + AutoTheft_2020) AS Total_AutoTheft
    FROM
      `sql-project-341315.toronto_crime.crime_2016`
    GROUP BY
      1,
      2)),
  Break_And_Enter_Table AS (
  SELECT
    Neighbourhood,
    Population,
    RANK() OVER (ORDER BY Population DESC) AS Population_Ranking,
    ROUND(SUM((population / 3042042)* 100),2) AS Percentage_of_Population,
    Total_BE,
    ROUND(SUM((Total_BE/50292) * 100),2) AS Percentage_Of_Total_BE
  FROM (
    SELECT
      Neighbourhood,
      F2020_Population_Projection AS Population,
      SUM(BreakAndEnter_2014 + BreakAndEnter_2015 + BreakAndEnter_2016 + BreakAndEnter_2017 + BreakAndEnter_2018 + BreakAndEnter_2019 + BreakAndEnter_2020) AS Total_BE
    FROM
      `sql-project-341315.toronto_crime.crime_2016`
    GROUP BY
      1,
      2)
  GROUP BY
    1,
    2,
    5),
  Robberies_Table AS (
  SELECT
    Neighbourhood,
    Population,
    Total_Robberies,
  FROM (
    SELECT
      Neighbourhood,
      F2020_Population_Projection AS Population,
      SUM(Robbery_2014 + Robbery_2015 + Robbery_2016 + Robbery_2017 + Robbery_2018 + Robbery_2019 + Robbery_2020) AS Total_Robberies
    FROM
      `sql-project-341315.toronto_crime.crime_2016`
    GROUP BY
      1,
      2)),
  Theft_Over_Thousand_Table AS (
  SELECT
    Neighbourhood,
    Population,
    Total_Theft_Over_Thousand,
  FROM (
    SELECT
      Neighbourhood,
      F2020_Population_Projection AS Population,
      SUM(TheftOver_2014 + TheftOver_2015 + TheftOver_2016 + TheftOver_2017+ TheftOver_2018 + TheftOver_2019 + TheftOver_2020) AS Total_Theft_Over_Thousand
    FROM
      `sql-project-341315.toronto_crime.crime_2016`
    GROUP BY
      1,
      2)),
  Murders_Table AS (
  SELECT
    Neighbourhood,
    Population,
    Total_Murders,
  FROM (
    SELECT
      Neighbourhood,
      F2020_Population_Projection AS Population,
      SUM(Homicide_2014 + Homicide_2015 + Homicide_2016 + Homicide_2017 + Homicide_2018 + Homicide_2019 + Homicide_2020) AS Total_Murders
    FROM
      `sql-project-341315.toronto_crime.crime_2016`
    GROUP BY
      1,
      2)),
  Shootings_Table AS (
  SELECT
    Neighbourhood,
    Population,
    Total_Shootings,
  FROM (
    SELECT
      Neighbourhood,
      F2020_Population_Projection AS Population,
      SUM(Shootings_2014 + Shootings_2015 + Shootings_2016 + Shootings_2017 + Shootings_2018 + Shootings_2019 + Shootings_2020) AS Total_Shootings
    FROM
      `sql-project-341315.toronto_crime.crime_2016` -- using excel, I calculated the growth percentages of each crime.
    GROUP BY
      1,
      2))
SELECT -- Now that all the tables are created using WITH clauses, I selected the columns I wanted from each and joined them all together
  ass.Neighbourhood,
  Total_Assaults,
  Total_AutoTheft,
  Total_BE,
  Total_Robberies,
  Total_Theft_Over_Thousand,
  Total_Murders,
  Total_Shootings,
  SUM(Total_Assaults + Total_AutoTheft + Total_BE + Total_Robberies+ Total_Theft_Over_Thousand + Total_Murders + Total_Shootings) AS Total_Crime,
  Assault_Increase,
  ROUND(Assault_Growth_Percentage * 100,2) AS Assault_Growth_Percentage,
  Auto_Theft_Increase,
  ROUND(Auto_Theft_Growth_Percentage_ * 100,2) AS Auto_Theft_Growth_Percentage,
  Break_And_Enter_Increase,
  ROUND(Break_And_Enter_Growth_Percentage * 100,2) AS Break_And_Enter_Growth_Percentage,
  Robbery_Increase,
  ROUND(Robbery_Growth_Percentage * 100,2) AS Robbery_Growth_Percentage,
  Theft_Over_Increase,
  ROUND(Theft_Over_Growth_Percentage* 100,2) AS Theft_Over_Growth_Percentage,
  Homicide_Increase_,
  ROUND(Homicide_Growth_Percentage_ * 100,2) AS Homicide_Growth_Percentage_,
  Shootings_Increase_,
  ROUND(Shooting_Growth_Percentage_ * 100,2) AS Shooting_Growth_Percentage_

FROM
  Assault_Table ass
JOIN
  Auto_Theft_Table auto
ON
  ass.Neighbourhood = auto.Neighbourhood
JOIN
  Break_And_Enter_Table be
ON
  ass.Neighbourhood = be.Neighbourhood
JOIN
  Robberies_Table rt
ON
  ass.Neighbourhood = rt.Neighbourhood
JOIN
  Theft_Over_Thousand_Table tt
ON
  ass.Neighbourhood = tt.Neighbourhood
JOIN
  Murders_Table mt
ON
  ass.Neighbourhood = mt.Neighbourhood
JOIN
  Shootings_Table st
ON
  ass.Neighbourhood = st.Neighbourhood
JOIN 
  `sql-project-341315.toronto_crime.growth_crime`g 
ON 
  ass.Neighbourhood = g.Neighbourhood
GROUP BY
  1,
  2,
  3,
  4,
  5,
  6,
  7,
  8,
  10,
  11,
  12,
  13,
  14,
  15,
  16,
  17,
  18,
  19,
  20,
  21,
  22,
  23
ORDER BY
  9 DESC


