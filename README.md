# PyBer Challenge

## Overview of Project

Our primary customer for this project is PyBer, a ride sharing service, and the executive leadership of the company who has asked us to analyze data from January to May of 2019 and make analytic conclusions regarding the number of drivers and riders as well as the percentage of total fares, drivers, and riders based off of type of city.  This analysis will help PyBer improve access to its ride sharing service and determine affordability to underserved neighborhoods.    

V. Isualize, the CEO of PyBer, has asked us to provide visualizations using two sets of data, [city data](https://github.com/MaureenFromuth/PyBer_Analysis/blob/master/city_data.csv) and [ride data](https://github.com/MaureenFromuth/PyBer_Analysis/blob/master/ride_data.csv).  Each data set provided key fields needed to provide visualizations of PyBer’s business, as seen below.  

The city data provides an overview of data for each city and contains the following information:
- City Name (‘city’) 
- Number of Drivers in that City (‘driver_count’)
- Type of City, e.g. Urban, Suburban, Rural (‘type’)


The ride data provides data on each ride and contains the following information:
- City Name the Ride was Executed In (‘city’)
- Date of Ride (‘date’)
- Fare for the Ride (‘fare’)
- ID of the Ride (‘ride_id’)

To facilitate this calculation, we conducted a left merged of the city data into the ride data on the field ‘city’ using the code below.  Of note, with this merge, the field ‘number of riders’ from the city data is repeated for each ride (e.g. every ride that is conducted in Port Angela will have ’67’ in that field).  As such, when calculating the total number of drivers, we will not be able to use the merged data set but will need to reference the original city data.

```
# Combine the data into a single dataset
pyber_data_df = pd.merge(ride_data_df, city_data_df, how="left", on=["city", "city"])
```

Using this data, V. Isualize specifically requested a summary DataFrame identifying the total number of rides, total drivers, total fares, average fare per ride and average fare per driver for each city type.   She also asked for a line chart visualizing the total fares per week for each type of city.  This data will help provide insight into the supply of drivers in each type of city and the differences in fee per ride in that type of city.  Likewise, the line chart will provide insight into the times of year in which each type of city sees the highest number of rides and if there are any differences between types of cities and rides.

## Results


### Summary per Type of City

 To visualize a summary of data per city type, we used the groupby function within python using ‘type’ as our primary index.  Using this data, we were able to calculate the total rides, total drivers, and total fare per city type.  See below for the code used to calculate these totals.  Of note, as identified above to calculate the total number of drivers we had to use the city data rather than the merged data.  

 
```
#  1. Get the total rides for each city type
total_ride_count = pyber_data_df.groupby(["type"]).count()["ride_id"]

# 2. Get the total drivers for each city type
total_drivers = city_data_df.groupby(["type"]).sum()["driver_count"]

#  3. Get the total amount of fares for each city type
total_fare = pyber_data_df.groupby(["type"]).sum()["fare"]

```

Once we calculated the totals, we used these to calculate the average fare per ride and average fare per driver.  Of note, while we wrote out / combined the code below, you can in fact replace the code with each of the variables (e.g. avg_fare_ride = total_fare / total_ride_count).

```
#  4. Get the average fare per ride for each city type. 
avg_fare_ride = pyber_data_df.groupby(["type"]).sum()["fare"] / pyber_data_df.groupby(["type"]).count()["ride_id"] 

# 5. Get the average fare per driver for each city type. 
avg_fare_driver = pyber_data_df.groupby(["type"]).sum()["fare"] / city_data_df.groupby(["type"]).sum()["driver_count"]

```

We then created a new data frame including these variables and added a few formatting changes to arrive a the summary below.

>**PyBer Summary per City Type**

![PyBer Summary per City Type](https://github.com/MaureenFromuth/PyBer_Analysis/blob/master/Summary_PyBer.png)


This summary of the PyBer data highlighted a significant difference between the total number of drivers, rides, and fares for each city type.  For example, there are more than 10 times the number of rides in urban areas as there is in rural areas, and almost 6 times the number of suburban rides are there are rural rides.  The number of drivers in urban areas is 30 times more than in rural areas, and suburban drivers are more than 6 times those in rural areas.  The total fare is also quite different between each with urban fares being 9 times more than rural and suburban fares being 4.5 times more than rural.  With the difference between drivers and rides for rural vs. urban being significantly greater than the difference between fares, the result is that the fares per ride and fares per driver is actually greater for rural drivers/rides than it is for urban and suburban.   More specifically, the fare per driver is 3 times greater in rural areas than in urban areas, and the fare per ride is 1.4 times greater for rural rides than it is for urban rides.


### Weekly Fares per Types of City

>**DataFrame: PyBer Total Weekly Fare per City Type**

![DataFrame: PyBer Total Weekly Fare per City Type](https://github.com/MaureenFromuth/PyBer_Analysis/blob/master/Total%20Weekly%20Fare%20per%20City%20Type.png)

>**Line Graph: PyBer Total Weekly Fare per City Type**

![PyBer Total Weekly Fare per City Type](https://github.com/MaureenFromuth/PyBer_Analysis/blob/master/Total%20Fares%20by%20City%20Type.png)



## Summary

There are a few overall conclusions we can draw from the comparisons between the summaries with the original data and the updated data.  The following outlines four key areas in which we identify changes. 

*Change 1 - Performance Per School Drop*
 
With the original data, Thomas High School ranked number two out of fifteen schools based off of overall percentage of students who passed both the math and reading tests.  With the new data, however, they dropped to number 8 out of fifteen, with their overall percentage of passing dropping from 90.9% to 65.8%.  As identified above this significant drop is due to the fact that our calculations included the original total student population for Thomas High School.  This drop indicates how important the 9th grade scores were in their initial #2 ranking, but, due to the method of calculating the overall percentage of passing, it is difficult to identify how much of an impact their scores had.

*Change 2 - Decrease in the % of Students Passing Math, Reading, and Both within their Size, Spending, and Type Bin*

As identified above, although the average scores did not have a noticeable change, there was a significant decrease in the percentage of students who pass reading, math, and both reading and math.  These decreases were not significant within the spending and size bins, and slightly less so within the type.  This indicates that the 9th grade scores were significantly higher than many of their peers from other schools in these bins. 

*Change 3 - Small Increase in the Average Reading Scores within their Size, Spending, and Type Bin*

Within each of these bins there was a negligible increase between the old and the updated data for the average reading scores.  This change indicated that the 9th grade reading scores were likely lower than the rest of the grades’ reading scores.  The math and reading scores per grade summary confirmed this, with the 9th grade reading scores at Thomas High School being slightly less than the 10th and 12th graders.  

*Change 4 - The Average Math Scores for the District Decreased while the Average Reading Scores Increased *

Although both of these changes were minimal and negligible depending on the formatting, there was a small decrease in average math scores for the district and a negligible increase in average reading scores when we removed the erroneous data.  As mentioned above, the minuscule increase in average reading scores is a result of removing the 9th grade scores that were below the average and enough so that it was pulling the overall average reading scores down.  
