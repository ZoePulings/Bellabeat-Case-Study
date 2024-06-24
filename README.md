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

```
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
```

Now I have 5 data tables I will continue to check for duplicates and N/A values.
```
sum(duplicated(Activity))
 0
sum(duplicated(HourlyActivity))
 175
sum(duplicated(WeightLog)
 2
sum(duplicated(MinuteActivity))
 0
sum(duplicated(MinSle))
 0
```
As you can see there are duplicates in the HourlyActivity and WeightLog dataset. To remove these duplicates I use the unique function.
```
HourlyActivity <- unique(HourlyActivity)
Weightlog <- unique(Weightlog)
```
Now the N/A values
```
sum(is.na(Activity))
[1] 0
sum(is.na(HourlyActivity))
[1] 0
sum(is.na(Weightlog))
[1] 94
sum(is.na(MinuteActivity))
[1] 0
sum(is.na(MinSle))
[1] 0
```
The WeightLog data table contains 94 N/A values. I searched the table and found that the N/A values are settled in the 5th column. I deleted this column with the select function. Run the code again.
```
Weightlog <- select(Weightlog, -Fat)
sum(is.na(Weightlog))
[1] 0
```
Because I want to do research about each participant on each day and hour I'll do some more processing with the tables to contain columns like Date, Time , Weekday. To become this I need to change the format and seperate the strings.
```
MinuteSle <- MinSle %>% # separate date-time in two columns
  separate(date, into = c("Date", "Time"), sep = " ", remove = FALSE) %>%
  mutate(Date = lubridate::as_date(Date, format = "%m/%d/%Y"))
MinuteSle$date <- NULL # remove column "date"
MinuteSle$Weekday <- wday(MinuteSle$Date, label = TRUE ) # create assiciation weekdays

Activity <- Activity %>% 
  mutate(ActivityDate = lubridate::as_date(ActivityDate, format = "%m/%d/%Y" ))
  separate(ActivityDate, into = c("Year", "Month", "Day"), sep = "-", remove = FALSE)
Activity$weekday <- wday(Activity$ActivityDate, label = TRUE)

HourlyActivity <- HourlyActivity %>% # seperate date and time
  separate(`parse_date_time(ActivityHour...2, "m/d/y H:M:S p")`, 
           into = c("Date", "Time"), sep= " ", remove = FALSE)
HourlyActivity$ActivityHour...2 <- NULL
HourlyActivity[is.na(HourlyActivity)] <- "00:00:00"
HourlyActivity$Avaragestep <- HourlyActivity %>% 
  group_by(Time) %>% 
  sum(Steptotal)
HourlyActivity$Weekday <- wday(HourlyActivity$Date, label = TRUE)

Weekendhours<- HourlyActivity %>% # create new table for weekend (sat&sun)
  select(Calories, TotalIntensity, StepTotal, Time, Date, Weekday) %>% 
  group_by(Weekday) %>% 
  filter(Weekday == "Sun"| Weekday =="Sat")

Weekhours <- HourlyActivity %>% # create new table for week (mon-fri)
  select(Calories, TotalIntensity, StepTotal, Time, Date, Weekday) %>%
  group_by(Weekday) %>% 
  filter(Weekday== "Mon"| Weekday=="Tue" | Weekday=="Wed" | Weekday=="Thu" | Weekday=="Fri")
```










