# Bellabeat-Case-Study
Complete data analysis about Bellabeat


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

### Exploring and summarising

Before we go any further in the process fase, I want to explore a bit more about the data I am working with. I’ll use the n_distinct function to find out how many participants I have in each data table. Before was told this was a dataset from 30 Fitbit users. Let’s check this out.


```
n_distinct(Activity$Id)
[1] 35
n_distinct(HourlyActivity$Id)
[1] 35
n_distinct(MinuteActivity$Id)
[1] 35
n_distinct(Weightlog$Id)
[1] 13
n_distinct(MinuteSle$Id)
[1] 25
```

We have three data tables with 35 participants. The WeightLog data table contains 13 and MinuteSle contains 25, this is something to think about.. For some reason a lot customers don’t feel to share their weight. And less participant share sleep minutes. We’ll look further in this later in the study.
***

### Analyse & Visualize 

Let's start with creating new tables , using the group_by(), mean() and sum() functions. 
Starting with the Activity table. 

      ! info: First I didn't make all these different data tables. Afterwards I
      decided to separate things to make it more easy when creating a plot.

      
```
ActivitySummary <- Activity %>% # Table grouped by weekday
  group_by(weekday) %>% 
  summarise(
    Steps = sum(TotalSteps),
    Distance = sum(TotalDistance),
    VeryActive = sum(VeryActiveMinutes),
    LightActive = sum(LightlyActiveMinutes),
    MeanSteps = mean(TotalSteps),
    MeanDistance = mean(TotalDistance),
    MeanveryActive = mean(VeryActiveMinutes),
    MeanLightActive = mean(LightlyActiveMinutes))

DailySummarizeAvarage <- Activity %>% # Table grouped by Id 
  group_by(Id) %>% 
  summarise(
    TotalSteps_count = mean(TotalSteps),
    Totaldistance_count = mean(TotalDistance),
    TrackerDistance_count = mean(TrackerDistance),
    LoggedActivitiesDistance_count = mean(LoggedActivitiesDistance),
    Calories_count = mean(Calories))

ActivityDaily <- Activity %>% # Table grouped by day, month & weekday
  group_by(Day, Month, weekday) %>% 
  summarise(
   Totalsteps = sum(TotalSteps),
   Calories = sum(Calories),
   SedentaryMinutes = sum(SedentaryMinutes))
```

With all these numbers given I created a plot to gave me a better vision of what these tables are telling me.


```
DailySummarizeAvarage%>% # CALORIES VS STEPS, red line = avarage steps
    ggplot(aes(reorder(factor(Id), TotalSteps_count))) +
    geom_col(aes(y = TotalSteps_count, fill = Calories_count)) +
    scale_fill_gradient2(low = "red", high = "purple", midpoint = median(DailySummarizeAvarage$Calories_count),  name = "Total Calories") +
    coord_flip() +
    geom_hline(yintercept = 6982, linetype = 2, color = "red")+
    labs(x = "Id", y = "Total Steps", title = "Calories burnt by total steps")+
    theme( panel.background = element_rect(fill = "black"),
        plot.background = element_rect(fill = "black"), 
        panel.grid.major= element_line(color = "darkgrey", size = 0.2), 
        panel.grid.minor = element_line(color = "darkgrey", size = 0.2),
        axis.title = element_text(color = "white"),
        axis.text = element_text(color = "lightgrey"),
        legend.background = element_rect(fill = "black"),
        legend.text = element_text(color = "white"),
        legend.title = element_text(color = "white"),
        plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
        title = element_text(color = "white"))
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/a89d0132-e254-4f13-aca6-ae2ff97f9d93)


More participant contain the color purple in the upper half which means calories burned between 2500 and 3000. The other half contains more red colors, which mean less than 2000 calories burned.

We can confirm that the more steps a participant takes, more calories are burned. With an average amount of 6982 steps.

This is a very logical result and confirms that our data is reliable. With that said I created a second plot to dubble check if the data is correct.


```
DailySummarizeAvarage %>% 
   ggplot(aes(reorder(factor(Id),TotalSteps_count))) +
   geom_col(aes(y = TotalSteps_count, fill = Totaldistance_count)) +
   scale_fill_gradient(low = "white", high = "darkgreen", name = "Total distance (Km)") +
   coord_flip() +
   labs( x = "Id", y = "Total Steps", title = "Quality over Quantity control")+
   theme(
         panel.background = element_rect(fill = "black"),
         plot.background = element_rect(fill = "black"), 
         panel.grid.major= element_line(color = "darkgrey", size = 0.2), 
         panel.grid.minor = element_line(color = "darkgrey", size = 0.2),
         axis.title = element_text(color = "white"),
         axis.text = element_text(color = "lightgrey"),
         legend.background = element_rect(fill = "black"),
         legend.text = element_text(color = "white"),
         legend.title = element_text(color = "white"),
         plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
         title = element_text(color = "white"))
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/6e34d16f-834f-42a8-9249-90bf27d3fbe5)


Again we get an even more clear confirmation that the data is reliable. The more steps a participant took, the longer the distance they achieved.

With above given I was curious how many participant logged their activity distance.


```
DailySummarizeAvarage %>% 
   ggplot(aes(reorder(factor(Id), Totaldistance_count))) +
   geom_col(aes(y = Totaldistance_count, fill = LoggedActivitiesDistance_count)) +
   scale_fill_gradient(low = "darkred", high = "green", name = "Total Logged Distance") +
   coord_flip() +
   labs(x = "Id", y = " Total distance (Km)", title = " Total Logged distance") +
   theme(panel.background = element_rect(fill = "black"),
         plot.background = element_rect(fill = "black"), 
         panel.grid.major= element_line(color = "darkgrey", size = 0.2), 
         panel.grid.minor = element_line(color = "darkgrey", size = 0.2),
         axis.title = element_text(color = "white"),
         axis.text = element_text(color = "lightgrey"),
         legend.background = element_rect(fill = "black"),
         legend.text = element_text(color = "white"),
         legend.title = element_text(color = "white"),
         plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
         title = element_text(color = "white"))
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/92012a62-a5ef-4bac-98d0-82e4e5ae9e45)


The plot tell us that only 20% of the participant have logged their activity distance. This makes me question why not?


```
DailySummarizeAvarage %>% 
   ggplot(aes(reorder(factor(Id), Totaldistance_count))) +
   geom_col(aes(y = Totaldistance_count, fill = LoggedActivitiesDistance_count)) +
   scale_fill_gradient(low = "darkred", high = "green", name = "Total Logged Distance") +
   geom_point(data = AvarageWeight, aes(x = factor(Id), y = meanBMI), color = "yellow") +
   coord_flip() +
   labs(x = "Id", y = " Total distance (Km)", title = " Total Logged distance", subtitle = " BMI " ) +
   theme(plot.subtitle = element_text(color = "yellow", hjust = 1),
         panel.background = element_rect(fill = "black"),
         plot.background = element_rect(fill = "black"), 
         panel.grid.major= element_line(color = "darkgrey", size = 0.2), 
         panel.grid.minor = element_line(color = "darkgrey", size = 0.2),
         axis.title = element_text(color = "white"),
         axis.text = element_text(color = "lightgrey"),
         legend.background = element_rect(fill = "black"),
         legend.text = element_text(color = "white"),
         legend.title = element_text(color = "white"),
         plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
         title = element_text(color = "white"))
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/3f06ddec-d7ba-4bf9-a573-450b1d53e61d)


No positive correlation can be found. We only can confirm that participant with higher BMI intend to take less steps , achieve less distance. 

    ! I choose for BMI instead of weight because this previews more correctly the size of a participant.

Let's see if we can go further with the steps participants took and search for some useful trends. 


```
Activity %>% 
  ggplot(aes(weekday,TotalSteps)) +
  geom_col(aes(fill= weekday)) + scale_fill_brewer(palette="Set2")+
  geom_point(ActivitySummary, mapping = aes(weekday, MeanDistance*50000), color = "red",shape = 8, group = "identidy")+
  geom_point(ActivitySummary, mapping = aes(weekday, MeanSteps*100), color = "blue",shape = 8 , group = "identidy")+
  annotate("text", x = "Sat" , y = 870000, label = "7753 Steps", colour = "blue", size = 2.7)+
  annotate("text", x = "Sun", y = 600000, label = "6607 Steps", colour = "blue", size = 2.7)+
  annotate("text", x = "Sun", y = 190000, label = "4,8 Km", colour = "red", size = 3)+
  annotate("text", x = "Sat", y = 350000, label = "5,5 Km", colour = "red", size = 3)+
  labs(title = "Count & Average Steps")+ theme(plot.title = element_text(color = "white"))+
  theme(panel.background = element_rect(fill = "black"),
        plot.background = element_rect(fill = "black"), 
        panel.grid.major= element_line(color = "darkgrey", size = 0.2), 
        panel.grid.minor = element_line(color = "darkgrey", size = 0.2),
        axis.title = element_text(color = "white"),
        axis.text = element_text(color = "lightgrey"),
        legend.position = "none",
        legend.background = element_rect(fill = "black"),
        plot.margin = margin(t = 30, l=10, r=10, b=20))
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/5c20979f-b153-49a9-b8f6-701b6e8d7435)


It seems that the most steps where taken on Tuesday and Saturday. With on Saturday the highest average steps of 7753. And the least steps on Sunday with an average of 6607 steps. Participants intent to use Saturday for going out and do stuff, while on Sunday they take it more slow and rest out.

I want to analyse this deeper, from days to hours. But first I have to create the right table.


```
HourlyActivity_summary <- HourlyActivity %>% # create new data table 
  group_by(Id...1, Weekday) %>% 
  summarise(
    Intensity = sum(TotalIntensity),
    Calories = sum(Calories),
    Steps = sum(StepTotal))

Weekendhours<- HourlyActivity %>% 
  select(Calories, TotalIntensity, StepTotal, Time, Date, Weekday) %>% 
  group_by(Weekday) %>% 
  filter(Weekday == "Sun"| Weekday =="Sat")

Weekhours <- HourlyActivity %>% 
  select(Calories, TotalIntensity, StepTotal, Time, Date, Weekday) %>%
  group_by(Weekday) %>% 
  filter(Weekday== "Mon"| Weekday=="Tue" | Weekday=="Wed" | Weekday=="Thu" | Weekday=="Fri")
```

Let's make a plot with this.


```
ggplot(HourlyActivity, aes(Time, TotalIntensity))+ # Intensity/hour
  geom_col(HourlyActivity, mapping=aes(fill= StepTotal))+
  scale_x_discrete(label = labels)+
  labs(title = " Total intensity during 24 Hours" , x= "Hour of the day", y= "Intensity", fill= "Steps")+
  theme(panel.background = element_rect(fill = "black"),
        plot.background = element_rect(fill = "black"), 
        panel.grid.major= element_line(color = "black", size = 0.2), 
        panel.grid.minor = element_line(color = "black", size = 0.2),
        axis.title = element_text(color = "white"),
        axis.text = element_text(color = "lightgrey"),
        legend.background = element_rect(fill = "black"),
        legend.text = element_text(color = "white"),
        legend.title = element_text(color = "white"),
        plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
        title = element_text(color = "white"))+
  annotate("segment", x="12:00:00", y= 42000, xend="12:00:00", yend= 40000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="17:00:00", y= 42000, xend="17:00:00", yend= 40000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="18:00:00", y= 42000, xend="18:00:00", yend= 40000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="19:00:00", y= 42000, xend="19:00:00", yend= 40000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("text", x= "15:00:00", y= 44000, label = "peak hours" , colour= "green")
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/42e63b6d-e079-4490-94c3-866662bb29ad)

Above plot shows you a total intensity from a 2 month range over 24 hours. The peak intensity hours are 12 AM , this is for a lot people the lunch break they take in a work day, which could mean they go for a walk or something. And 17-19 PM, this are the first 3 hours after people have done working. This could mean they often do some exercise after work, like going to the gym, take a run or maybe letting the dog out.

I can make the conclusion that a lot participant have a 9-5 job. Probably working a lot at a desk. Maybe it's interesting to give those people an opportunity to learn about desk yoga or a chair workout to increase the intensity at low peak hours like 15 PM.

Also this plot show us that people stay up quite late. Only between midnight and 5 AM there where a low amount of steps taken. When do these people sleep??


```
ggplot(HourlyActivity, aes(Time, Calories))+ #Calories/hour
  geom_col(HourlyActivity, mapping=aes(fill= StepTotal))+
  scale_x_discrete(label = labels)+
  labs(title = " Calories burned during 24 Hours" , x= "Hour of the day", y= "Calories burned", fill= "Steps")+
  theme(panel.background = element_rect(fill = "black"),
        plot.background = element_rect(fill = "black"), 
        panel.grid.major= element_line(color = "black", size = 0.2), 
        panel.grid.minor = element_line(color = "black", size = 0.2),
        axis.title = element_text(color = "white"),
        axis.text = element_text(color = "lightgrey"),
        legend.background = element_rect(fill = "black"),
        legend.text = element_text(color = "white"),
        legend.title = element_text(color = "white"),
        plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
        title = element_text(color = "white"))+
  annotate("segment", x="12:00:00", y= 240000, xend="12:00:00", yend= 230000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="17:00:00", y= 240000, xend="17:00:00", yend= 230000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="18:00:00", y= 240000, xend="18:00:00", yend= 230000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="19:00:00", y= 240000, xend="19:00:00", yend= 230000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("text", x= "15:00:00", y= 250000, label = "peak hours" , colour= "green")
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/031fe190-91bd-43ed-bc41-f46f1e1a93ff)

This plot shows us the same as before. The only thing we can learn from this is that people do burn calories when they sleep. Almost the same amount of calories are burned as when people do just before bedtime and right after waking up.

Maybe this could be a way to motivate people more to take better/more sleep, as this affects not only their energy levels but also could support weight loss.

Before we found out that the most steps where taken on Saturday and the least on Sunday. The peak hours where 12 AM and 17-19 PM , let's see if this differences when we separate the intensity on weekday (mon-fri) and weekends (sat-sun).


```
ggplot(Weekhours, aes(Time, TotalIntensity))+ #Mon-Fri INTENSITY
  geom_col(Weekhours, mapping=aes(fill= StepTotal))+
  scale_x_discrete(label = labels)+
  labs(title = " Week intensity" , x= "Hour of the day", y= "Intensity", fill= "Steps")+
  theme(panel.background = element_rect(fill = "black"),
        plot.background = element_rect(fill = "black"), 
        panel.grid.major= element_line(color = "black", size = 0.2), 
        panel.grid.minor = element_line(color = "black", size = 0.2),
        axis.title = element_text(color = "white"),
        axis.text = element_text(color = "lightgrey"),
        legend.background = element_rect(fill = "black"),
        legend.text = element_text(color = "white"),
        legend.title = element_text(color = "white"),
        plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
        title = element_text(color = "white"))+
  annotate("segment", x="17:00:00", y= 30000, xend="17:00:00", yend= 29000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="12:00:00", y= 30000, xend="12:00:00", yend= 29000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="18:00:00", y= 30000, xend="18:00:00", yend= 29000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="19:00:00", y= 30000, xend="19:00:00", yend= 29000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("text", x= "15:00:00", y= 31000, label = "peak hours" , colour= "green", hjust=0.5)
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/10c45a81-d0ae-4e6a-825a-27c3281263c4)

On weekdays Monday-Friday we get the peak hours. 12 AM and 17-19PM. This is quit logical.

Let's see the intensity for the weekend.


```
ggplot(Weekendhours, aes(Time, TotalIntensity))+ # SAT-SUN INTENSITY
  geom_col(Weekendhours, mapping=aes(fill= StepTotal))+
  scale_x_discrete(label = labels)+
  labs(title = " Weekend intensity" , x= "Hour of the day", y= "Intensity", fill= "Steps")+
  theme(panel.background = element_rect(fill = "black"),
        plot.background = element_rect(fill = "black"), 
        panel.grid.major= element_line(color = "black", size = 0.2), 
        panel.grid.minor = element_line(color = "black", size = 0.2),
        axis.title = element_text(color = "white"),
        axis.text = element_text(color = "lightgrey"),
        legend.background = element_rect(fill = "black"),
        legend.text = element_text(color = "white"),
        legend.title = element_text(color = "white"),
        plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
        title = element_text(color = "white"))+
  annotate("segment", x="13:00:00", y= 12500, xend="13:00:00", yend= 12000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="12:00:00", y= 12500, xend="12:00:00", yend= 12000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("segment", x="14:00:00", y= 12500, xend="14:00:00", yend= 12000, arrow = arrow(type = "closed", length = unit(0.02, "npc")), colour = "green")+
  annotate("text", x= "13:00:00", y= 13000, label = "peak hours" , colour= "green", hjust=0.5)
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/294bef6b-3403-46b2-84d2-6b877c6f5425)

This gives us something different. Which is quit obvious because there are only 2 weekend-days and 5 weekdays. We see that the peak hours change to the early afternoon, from 12 AM - 15 PM.

We also see that people intent to sleep at least 2 hours longer than on weekdays. Besides that they still keep waking up early in the morning around 7 AM, probably because they have a build rithm of sleep during the week.

![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/db142566-feee-465e-ba82-81ae0141e474)
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/103547dc-af1b-41e0-aee3-e3156f8f6694)


With above two plots we can see that the intensity in Saturday is 1000 higher then on Sunday.

Now.. let's analyse the sleep of the participants. But first again the tables.


```
MinuteSle_summary <- MinuteSle %>% # create new data table 
  group_by(Id, Weekday) %>% 
  summarise(
    TotalSleepMinutes = sum(value),
    AvarageSleepMinutes = mean(value),
    TotalSleepHours = TotalSleepMinutes/60,
    Meanmitues = mean(AvarageSleepMinutes))

Hoursleep <- MinuteSle_summary %>% # create new data table
  group_by(Id) %>% 
  summarise(Sleephours = sum(TotalSleepHours), Sleepminutes= sum(TotalSleepMinutes))


AvarageWeight <- Weightlog %>% # create new data table 
  group_by(Id) %>% 
  summarise(
    mean(WeightKg),
    mean(BMI),
    names(AvarageWeight) <- c("Id", "meanweight", "meanBMI"))
```

Before we see how people sleep I want to make a plot from the 13 participants who logged their weight and BMI. This could help us further in the analyse process. To achieve such plot I had to make a new data table with only the 13 participants.


```
GroupId <- DailySummarizeAvarage %>% 
  select(Totaldistance_count, Calories_count, Id ) %>% 
  group_by(Id) %>% 
  filter(Id== "1503960366"|Id=="1927972279" |Id=="2347167796"| Id=="2873212765"|
         Id=="2891001357" |Id=="4319703577"|Id=="4445114986"| Id=="4558609924" |Id=="4702921684" |Id=="5577150313"|
         Id=="6962181067" |Id=="8253242879" |Id=="8877689391")
```
```
ggplot(AvarageWeight, aes(factor(Id)), meanBMI)+ 
  geom_col(AvarageWeight, mapping=aes(y= meanBMI, fill= meanweight))+
  scale_fill_gradient(low="white", high="red")+
  geom_point(GroupId, mapping=aes(x=factor(Id), y=Totaldistance_count, color="Distance"))+
  geom_point(GroupId, mapping=aes(x=factor(Id), y=Calories_count/100, color="Calories Burned"))+
  coord_flip()+
  labs(title= " BMI vs Calories vs Distance", x= "Id", y="BMI", fill= "Weight" )+
  theme(panel.background = element_rect(fill = "black"),
        plot.background = element_rect(fill = "black"), 
        axis.title = element_text(color = "white"),
        axis.text = element_text(color = "lightgrey"),
        legend.background = element_rect(fill = "black"),
        legend.text = element_text(color = "white"),
        legend.title = element_text(color = "white"),
        plot.margin = margin(t = 30, l= 10, r= 10, b = 20), 
        title = element_text(color = "white"),
        panel.grid.major= element_line(color = "black", size = 0.2), 
        panel.grid.minor = element_line(color = "black", size = 0.2))
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/01a686b8-e4a0-4af6-8ebb-0dfe14a07834)

With keeping in mind that this is a analyses from only 13 participants, we should consider this as not reliable. Next to that I can see a few very clear things.

Participants with a BMI higher then 25 seem to weight more. Also we can assume that participants with a higher BMI easier burns calories. They have a lower distance achieved but still more calories burned compared to the other participants.


```
ggplot(HourlyActivity_summary, aes(factor(Id...1), Intensity))+
  geom_col(HourlyActivity_summary, mapping=aes(fill = Weekday))+
  geom_point(Hoursleep, mapping=aes(x= factor(Id), y= Sleepminutes, color="red"))+
  coord_flip()+
  labs( title= "Intensity VS Sleep", x= "Id", colour= "Minutes Sleep")+
  theme(panel.background = element_rect(fill = "black"),
        plot.background = element_rect(fill = "black"), 
        panel.grid.major= element_line(color = "black", size = 0.2), 
        panel.grid.minor = element_line(color = "black", size = 0.2),
        axis.title = element_text(color = "white"),
        axis.text = element_text(color = "lightgrey"),
        legend.background = element_rect(fill = "black"),
        legend.text = element_text(color = "white"),
        legend.title = element_text(color = "white"),
        plot.margin = margin(t = 10, l= 10, r= 10, b = 10), 
        title = element_text(color = "white"))
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/f3ae700b-6dd8-46b4-99d6-beeb6304a00a)

There is no clear relationship between the total intensity and minutes each participant slept. Which is quit surprising. After analyzing this plot twice I found that if participants have an intensity higher then 15000, they intent to sleep more.

This is a good indication to use notifications, in the Bellabeat app, to motivate participants to achieve at least an intensity of 242 a day, to improve their sleep.

    ! calculation: So we have a total intensity bases on 62 days (n_distinct(HourlyActivity*Date)). 
    To know the intensity each participant should achieve a day : 15000/62 = 242. 
I'll do the same with the total calories each participant burned.


```
ggplot(HourlyActivity_summary, aes(factor(Id...1), Calories))+
  geom_col(HourlyActivity_summary, mapping=aes(fill = Weekday))+
  geom_point(Hoursleep, mapping=aes(x= factor(Id), y= Sleepminutes*5, color ="red"))+
  coord_flip()+
  labs( title= "Calories burned VS Sleep", x= "Id", colour= "Minutes Sleep")+
  theme(panel.background = element_rect(fill = "black"),
      plot.background = element_rect(fill = "black"), 
      panel.grid.major= element_line(color = "black", size = 0.2), 
      panel.grid.minor = element_line(color = "black", size = 0.2),
      axis.title = element_text(color = "white"),
      axis.text = element_text(color = "lightgrey"),
      legend.background = element_rect(fill = "black"),
      legend.text = element_text(color = "white"),
      legend.title = element_text(color = "white"),
      plot.margin = margin(t = 10, l= 10, r= 10, b = 10), 
      title = element_text(color = "white"))
```
![image](https://github.com/ZoePulings/Bellabeat-Case-Study/assets/173679723/0a131c7e-64c6-4b98-9463-e1fd03516bc6)

This doesn't match with my previous findings. Also this couldn't match because you can't relate calories burned with achieved intensity.

Earlier we found that people with an higher BMI burns faster calories. So they burn more calories with the same intensity. To make a correct plot with above given we need to add the BMI of each participant to recalculate how much calories each participant need to burn a day , to improve their sleep. Because we don't have enough data about participants BMI and we already have the intensity value they need to achieve I will not go further with this.

But it could be a good suggestion for the future. Even further we could recalculate how many steps each participant should need to take, based on their personal information ( BMI ), every day to achieve better sleep.

***

### Recommendations for Bellabeat

Bellabeat's beautifully designed technology that provides women an opportunity to gain knowledge about their own health and habits has already a great impact for those who chose to empower their overall health.


#### Key message for Bellabeat's marketing

Since their products focuses strongly on a female audience I would suggest to use their own marketing data , provided by their uses to gain more and better insights in trends. It will be necessary to create a market plan, to empower these users to share their personal data. Which will result in better analyses and better support for their overall health and habits. Creating an opportunity (good be a discount) that leads to a larger sample size will take Bellabeat to the next level.

With that said I could analyse some trends in the FitBit Fittness Tracker Dataset that gave me valuable insights.


#### Insights

* My analysis showed that the average amount of steps taken a day is 6982. I also can confirm that the average steps taken a day only slightly differ from each weekday (monday-sunday). Saturday is the most active day with an average of 7753 steps. On Sunday people take it slow with an average of 6607 steps. A notification on Sunday to motivate them to at least take 1000 steps more would be a good start to achieve better health.

* After analyzing the intensity, I can confirm that (most if not all) participants have a 9-5 job, probably working a lot at a desk. With only a high intensity hour at 12 AM , their lunch break, and after work from 5 till 7 PM. Bellebeat should provide, through the app, a collection guided video's of desk yoga and/or chair workouts, compared with motivation notifications. This way can Bellabeat help increase more intensity throughout the work hours which will result in more activity at the end of the day. These workouts will not only provide users who want to loose weight but their overall health. ( think about flexibility, muscle pain caused by lack of movement or the same posture for to long) Adults also need at least 2 days a week of muscle-strengthening activities (according to CDC), this is a great opportunity to reach this goal.

* Analyses showed that the intensity in the weekend has peak hours from 11 am until 15 pm. Further analysis shows that the peak intensity on Saturday is around 6000 while on Sunday around 5000. Again through notification we can remind/motivate people on Sunday to be more active due their inactivity throughout their work hours in the week.

* Also have my analysis confirmed that the amount of calories burned during their sleep is almost the same as calories burned just before bed time and right after waking up. With app notifications this given could stimulate users to take more sleep as this affects not only their energy levels but also support weight loss. Especially for those who lay down in the evening compared with a lovely snack. Besides my analysis also confirm that users only intend to sleep 5-6 hours a night which is not enough. Getting enough sleep can help you improve your heart health and metabolism while also reducing your risk of chronic conditions among other things. Users between age 18-60 should have at least 7 hours sleep. (according to CDC)

* When analyzing the BMI from the only 13 participants, I can confirm that people with higher BMI (above 25) easier burns calories. I recommend to send notifications to these people, providing them with these information could help them motivate to keep active. This could help them achieve faster and more effectively their goals.

* At least I performed analysis on participants sleep compared with their intensities. Analysis showed that people who have at least an intensity of 242 a day have better/more sleep. I recommend to send people in the late afternoon a notification if they seem to not reach the goal of an intensity of 242. This will remind them to do something more active and achieve better sleep.

* I recommend to motivate users to share their weight and BMI. With the BMI of each participant I can perform an even better analysis which can provide users the amount of steps to take AND calories to burn to achieve better sleep at night. This will make it easier to create an insight of what intensity is needed to achieve the goal of better sleep.


With all above recommendations I suggest to give the Bellabeat app a refreshing design. This new design will show users not only that Bellabeat is taking steps for new growth. But also provides them with a dashboard. On this dashboard (which each participant can personalize) they follow their goals. Such as: a count down of the taken steps, burned calories, intensity,... A count down works for most people as motivation , it gives a more clear sight of what they at least have to do to reach goals. Besides that the app should use notifications to remind people of the goal they want to achieve, how to achieve, and congrats when they achieve. At least I recommend to create an effective market strategy plan to convince people to share their personal data. This will not only help them but also spread a word in the world , because of the positive reactions of already users.




























































