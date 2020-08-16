 PyBer Challenge

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


This summary of the PyBer data highlighted a significant difference between the total number of drivers, rides, and fares for each city type.  For example, there are more than 10 times the number of rides in urban areas as there is in rural areas, and almost 6 times the number of suburban rides are there are rural rides.  The number of drivers in urban areas is 30 times more than in rural areas, and suburban drivers are more than 6 times those in rural areas.  The total fare is also quite different between each with urban fares being 9 times more than rural and suburban fares being 4.5 times more than rural.  With the difference between drivers and rides for rural vs. urban being significantly greater than the difference between fares, the result is that the fares per ride and fares per driver is actually greater for rural drivers/rides than it is for urban and suburban.   More specifically, the fare per driver is 3 times greater in rural areas than in urban areas, and the fare per ride is 1.4 times greater for rural rides than it is for urban rides.  This difference in average fare per ride and driver may be the result of customers requesting rides for longer distances due to their lack of proximity to their end destination.  We will need additional data, however, to include length of ride in minutes and miles to validate this assessment.


### Weekly Fares per Types of City

To assess the changes in total fare per week based on the type of city, we utilized the groupby function in addition to the pivot and resample functions.  Using the code below, we first grouped the data by type of city and then date to identify the total in fare for that specific date.  

```
# 9. Using groupby() on the pyber_data_df to create a new DataFrame showing the sum of the fares for each date where the indices are the city type and date.
fare_per_dtg_type = pyber_data_df.groupby(["type", "date"]).sum()[["fare"]]
```

Ensuring the index for our new dataframe is correct, we reset the index to identify a city type for each row, and then created a pivot table with the date as the index, the column as the types of cities, and the values as the total fare.  

```
# 10. Reset the index on the DataFrame you created in #1. This is needed to use the 'pivot()' function.
# df = df.reset_index()
fare_per_dtg_type = fare_per_dtg_type.reset_index()

# 11. Create a pivot table with the 'date' as the index, the columns ='type', and values='fare' to get the total fares for each type of city by the date. 
fare_per_dtg_type_pivot = fare_per_dtg_type.pivot(index = 'date', columns = 'type', values = 'fare')
```

We then used the loc function to select specific dates for our analysis, namely 1 Jan 2019 to 29 Apr 2019.

```
# 12. Create a new DataFrame from the pivot table DataFrame using loc on the given dates, '2018-01-01':'2018-04-29'.
fare_per_date_range = fare_per_dtg_type_pivot.loc['2019-01-01':'2019-04-29']
```

Finally, we set the index for the dataframe to date time and then resampled the dataframe based on a weekly basis with the values being the sum of the fares for that week.  This provided us the dataframe below and the basis for our line graph also listed below.

```
# 15. Create a new DataFrame using the "resample()" function by week 'W' and get the sum of the fares for each week.
fare_per_week = fare_per_date_range.resample('W').sum()
```


>**DataFrame: PyBer Total Weekly Fare per City Type**

![DataFrame: PyBer Total Weekly Fare per City Type](https://github.com/MaureenFromuth/PyBer_Analysis/blob/master/Total%20Weekly%20Fare%20per%20City%20Type.png)

>**Line Graph: PyBer Total Weekly Fare per City Type**

![PyBer Total Weekly Fare per City Type](https://github.com/MaureenFromuth/PyBer_Analysis/blob/master/Total%20Fares%20by%20City%20Type.png)

In general the weekly fares per type of city reflects similar differences as we see in the overall summary per type of city.  There are some interesting observations, however, to include spikes in both suburban and rural fares during holiday weekends and during the late spring/early summer timeframe.  More specifically, during the third week of January you see a significant jump in fares for rural rides.  This was Martin Luther King Jr. Day, a day in which many families take vacation and either travel out of the area or take advantage of the long weekend.  Likewise, you see a similar hike the week of 18 February which is Washington’s birthday, another long weekend.  You do not see the same level of spike in rides within rural and urban cities, however.  This could be for a myriad of reasons to include the proximity to events vs. rural drivers who would need to drive to those events either in the city or in suburban areas.  Finally, you also see an interesting increase in suburban drivers around the end of April.  This may be indicative of a rise in suburban drivers due to summer months and either spring break or vacation, but more data is needed for other years to validate this assessment.  


## Summary

With this data, we can make three recommendations to improve disparities among types of cities.  

First, one of the largest disparities is the number of drivers in rural and suburban cities.  By creating a campaign to increase drivers in these cities cities, you can increase the supply and thereby increase the amount of fares they bring in.  With greater fares per week, you can decrease the disparity in weekly fares in rural cities.  Additionally, with the data showing that the average fare per driver is the result of a small supply of drivers doing the majority of rides, the more drivers the less average fare per driver.  This will allow that to level out and decrease the disparity in fare per ride in urban cities as opposed to rural cities.

Another disparity is in the number of rides in rural and suburban areas.  You can attack this by creating a campaign that targets more rides in rural and suburban areas during holidays and the summer moths.  Understanding that these are the times with greater demand, you can use the additional drivers from above to increase rides in these weeks.  You can do this through deliberate marketing campaigns.  By increasing the rides during these weeks, you can increase the fares and rides, thereby decreasing the disparity in rides.

Finally, there is a disparity in the amount of fares in rural and suburban cities.  If you decrease the fare per ride in these cities, you may be able to draw up more demand.  When coupled with increased drivers and a deliberate marking campaign, you may be able to increase the amount of rides in these cities and thereby decrease the disparity in fares overall.   
