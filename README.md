# Bellabeat-Case-Study
Data analysis about Bellabeat

## About the Company

A tech-driven wellness company named Bellabeat wants to play it smart. Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. Urška Sršen, cofounder and chief Creative Officer of Bellabeat, believes that analysing smart device fitness data could help unlock new growth opportunities. Earlier used Sršen her background as an artist to develop beautifully designed technology that informs and inspires women around the world. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their own health and habits.

### Business ask

You have been asked to focus on one of Bellabeat’s products and analyse smart device data in order to gain insight into how consumers are using their smart devices. With these insights they aks you to make high-level recommendations to unlock new growth.

* What are some trends in smart devices ?
* What useful insight can you use for a Bellabeat device and help customers use ?
* How could these insights inform market strategy to unlock new growth ?

### Prepare usage data

FitBit fitness tracker dataset is a public dataset provided by Möbius on Kaggle.com. It contains personal fitness tracker data from thirty Fitbit users.

Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring.
This dataset might have some limitations with only 33 unique ID’s tracked over just 2 months. Sampling bias could be present with this small dataset and this short time period. This could lead to misleading trends or not being able to find a trend.

I’ll go through the data and check for duplicates, it’s completeness and incorrect data. Also assuming the data integrity is valid due being tracked by the smart device, Fitbit.
***
After installing the packages I start to import all csv files from Fitbit fitness dataset. Also I choose to merge Fitabase Data 3.12.16-4.11.16 and Fitabase Data 4.12.16-5.12.16 which contains data from 2 months. I will make 4 different data tables separated in weight log , daily , hourly and minute activity data.

To merge the datasets I’ll use bind_rows in the first place. Then I combine the tables from each time period together with bind_cols. Because more tables contain the same column name I removed the duplicated columns with the select function.
***
Activity <- bind_rows(dailyActivity_merged, dailyActivity_merged1)

WeightLog <- bind_rows(weightLogInfo_merged, weightLogInfo_merged1)

HourlyActivityCAL <- bind_rows(hourlyCalories_merged, hourlyCalories_merged1)

HourlyActivityINT <- bind_rows(hourlyIntensities_merged, hourlyIntensities_merged1)

HourlyActivitySTE <- bind_rows(hourlySteps_merged, hourlySteps_merged1)

HourlyActivity <- bind_cols(HourlyActivityCAL, HourlyActivityINT, HourlyActivitySTE)

MinCal <- bind_rows(minuteCaloriesNarrow_merged, minuteCaloriesNarrow_merged1)

MinInt <- bind_rows(minuteIntensitiesNarrow_merged, minuteIntensitiesNarrow_merged1)

MinMET <- bind_rows(minuteMETsNarrow_merged, minuteMETsNarrow_merged1)

MinSle <- bind_rows(minuteSleep_merged, minuteSleep_merged1)

Minste <- bind_rows(minuteStepsNarrow_merged, minuteStepsNarrow_merged1)

MinuteActivity <- bind_cols(MinCal, MinInt, MinMET, Minste)
***
Now I have 5 data tables I will continue to check for duplicates and N/A values.
***
sum(duplicated(Activity))

 0

sum(duplicated(HourlyActivity))

175

sum(duplicated(WeightLog))

2

sum(duplicated(MinuteActivity))

0

sum(duplicated(MinSle))

0
***
As you can see there are duplicates in the HourlyActivity and WeightLog dataset. To remove these duplicates I use the unique function.
***
HourlyActivity <- unique(HourlyActivity)

Weightlog <- unique(Weightlog)
***


