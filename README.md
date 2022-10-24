# **Bellabeat Case Study (Google Data Analytics Proffesionn Certificate)**

### How Can a Wellness Technology Company Play it Smart?

###### By: Katie Little


## **Background**

Bellabeat, founded in 2013, is a tech-driven wellness company created to empower women and enlighten them about their health and daily habits. Bellabeat utilizes technology to track women’s activity, sleep, stress, wellness, reproductive health, and water consumption.

#### Bellabeat products:
* **Bellabeat app**: Tracks users' health data for activity, sleep, stress, menstrual cycle, mindfulness habits
* **Leaf:** A wellness tracker which can be worn as a bracelet, necklace, or clip
* **Time:** A wellness watch to track health data
* **Spring:** A water bottle that tracks daily water intake and ensures proper hydration 
* **A subscription-based membership:** Gives users 24/7 access to tips guiding healthier daily living habits based on one’s lifestyle and goals


## **ASK**

#### *Guiding Questions*
1. What are some trends in smart device usage? 
2. How could these trends apply to Bellabeat customers? 
3. How could these trends help influence Bellabeat marketing strategy?

#### *Primary Stakeholders*

* Urška Sršen: The company’s co-founder and Chief Creative Officer
* Sando Mur: The company’s co-founder and mathemetician 

#### *Secondary Stakeholders*

* Bellabeat marketing analytics team: Team of data analysts who help guide the company’s marketing strategy 

#### *Business Task*

* Based on the data generated from Bellabeat’s smart devices, how are consumers using these products?
* Influence data-driven decisions and guide marketing strategy based on the insights gathered from the analysis.


#### *This analysis will focus on 5 different relationships:* 
* What pecentage of the day is spent on each level of activity?
* What is the relationship between daily number of steps and total calories burned per day?
* What is the relationship between calories burned throughout the day and activity level?
* How does activity level affect users' BMI?
* Does activity level infleunce longer sleep time?

## **PREPARE**

The data used for this analysis is the FitBit Fitness Tracker Data from Kaggle and is licensed under CC0: Public Domain by the author. This includes 18 datasets from 30 users who consented to having their personal data tracked and responses recorded through a distributed survey via Amazon Mechanical Turk between 03.12.2016-05.12.2016. The datasets are organized in long format and record physical activity, heart rate, sleep, and weight. All datasets are in .csv file format. 

The focus of this analysis will highlight users' daily habits. The data will include users' sleep and weight logs and measures of daily activity (including total steps, total distance, level of intensities, calories burned, and duration of exercise).

#### *Data Credibility*

A good data source is one that is reliable, original, comprehensive, current and cited. I will use the ROCCC method to see if the FitBit Fitness Tracker Data is credible. 

* Reliability: **UNKOWN.** Based on the Central Limit Theorem, a population of 30 may suffice, but the method for selecting these participants is unkown. The group of participants may make the data biased based on similar demographic characteristics and/or not using randomization as the sampling method. 
* Originality: **NO.** Data is collected from Amazon Mechanical Turk, a third-party source.
* Comprehensiveness: **NO.** As there are limitations to the data, another data source is needed to accurately answer how users' are using these devices. 
* Current: **NO.** As the data was collected in 2016, it is outdated. The usefulness of data decreases as time passes. Though, It is hard to predict if participants’ habits have changed across the years. 
* Cited: **SOMEWHAT.** The dataset comes from the Amazon Mechnical Turk, which is a credible organization. But, the data has not been refreshed. 

#### *Data Integrity*

There are some limitations to the data, as very little is known about the methodology utilized and biases may exist within the data. For future analysis, another dataset regarding Bellabeat's fitness data may be beneficial to ensure the data's integrity. 

#### *Data Sources Used in Analysis*

* dailyActivity_merged.csv
* sleepDay_merged.csv
* WeightLogInfo_merged.csv
* avgdata.csv
* sleepactivitylevel.csv


## **PROCESS**

Google Sheets, R, and SQL will be used to process the data for analysis. 

**NOTE** Big Query was used to generate SQL code. But, the sqldf() function in R will be used to display the code.

#### Cleaning the Data (Google Sheets)

The 3 files listed above were all imported into a single spreadsheet in Google Sheets. Each column of the six sheets were checked for blank cells using a filter. In the WeightLogInfo_merged.csv sheet, the ‘Fat’ column was removed as it contained 65 blank rows and only 2 cells with corresponding numbers. The format for date/time was changed to data alone in the sleepDay_merged.csv and WeightLogInfo_merged.csv files, as only some rows contained the time and time isn’t an important variable since the focus of this analysis is to track daily habits.

* Changing Column Names *
* In the dailyActvitiy_merged, ActivityDate -> Date
* In sleepDay_merged, SleepDay -> Date

This will make it easier to join files in my analysis. Now, each .csv file shares two of the same column names: Id and Date.

#### Cleaning the Data (R Studio)

Packages Installed in R Studio:

* install.packages("tidyverse")
* install.packages("here")
* install.packages("skimr")
* install.packages("janitor")

The packages, "here", "skimr", and "janitor" are used to clean the data, make file referencing easier, and allows us to skim through the data more efficiently. 

#### Loading packages

```R
library("tidyverse")
library("here")
library("skimr")
library("janitor")
library("readr")
library("ggplot2")
library("lubridate")
```
#### *Importing the Datasets*

The following datasets were first imported into Rstudio and were named accordingly. The read.csv() function imports each .csv file in the form of a data frame. 

#### Importing data
```R
daily_activity <- read.csv("../input/bellabeat-dataset/dailyActivity_merged.csv")
sleep_log <- read.csv("../input/bellabeat-dataset/sleepDay_merged.csv")
weight_log <- read.csv("../input/bellabeat-dataset-edited/weightLogInfo_merged.csv")
sleep_activity_level <- read.csv("../input/sleepvsactivitylevel/sleepvsactivitylevel.csv")
averaged_data <- read.csv("../input/averaged-data2/avgdata.csv")
```

I will use the duplicated() function to check for duplicate values in each .csv file. 

#### Checking for Duplicates 
```R
sum(duplicated(daily_activity))
sum(duplicated(weight_log))
sum(duplicated(sleep_log))
```
0
0
3
#### Removing duplicates from sleep_log .csv file using the drop_na() function.
```R
sleep_log <- sleep_log %>%
    distinct()%>%
    drop_na()
```

#### Using duplicated() function to check that duplicates were removed
```R
sum(duplicated(sleep_log))
```
0
#### Counting how many unique Id's there are in the daily_activity .csv file
```R
library(sqldf)
sqldf('SELECT DISTINCT Id FROM daily_activity ORDER BY Id')
```

A strange finding that I discovered, using the sqldf() function in R, is that there are 33 distinct Id's in the daily_activity .csv file. In the dataset on kaggle, it is stated that 30 people consented to participate in the study. It cannot be determined if this was merely an issue in the description of the dataset or within the data itself. Because each Id is unique and has 10 #'s, I will move forward with 33 as our sample size.

#### Counting how many unique Id's there are in the weight_log .csv file
```R
library(sqldf)
sqldf('SELECT DISTINCT Id FROM weight_log ORDER BY Id')
```

8 unique Id's were pulled from the weight log table. This number is unrepresentative of our sample size of 33 and will most likely skew results.

#### Counting how many unique Id's there are in the sleep_log .csv file
```R
library(sqldf)
sqldf('SELECT DISTINCT Id FROM sleep_log ORDER BY Id')
```

24 unique Id's were found in the sleep log table.

## **ANALYSIS**

#### Calculating Summary Statistics 
In order to get the minimum, maximum, median, and mean values for each column, I will use the summary() function. This will also help to identify any potential outliers. 

#Summary statistics for daily_activity
```R
daily_activity %>%
    select(-Id,-Date)%>%
    summary()
````
![](images/summarystats_dailyactivity)
    
#Summary Statistics for sleep_log
```R
sleep_log %>%
    select(-Id, -Date) %>%
    summary()
```

 
#Summary Statistics for weight_log
```R
weight_log %>%
    select(WeightPounds, BMI)%>%
    summary()
```


Based on the summary statistics, the average user takes 7,638 steps and burns 2,304 calories a day. The average user spends 991.2 minutes being sedentary, 192.8 minutes being lightly active, 13.56 minutes being fairly active, and 21.16 minutes being very active per day. The average user spends 458.5 minutes in bed and 419.2 minutes asleep. The average user's weight, in pounds, is 158.8, while the average BMI is 25.19. A limitation to the data is that the unit of length for Total Distance is unknown; therefore, we cannot draw a conclusion from this.

#### Counting how many people completed at least 8000 steps
```R
library(sqldf)
sqldf('SELECT Id,COUNT(*) AS completed_8k FROM daily_activity WHERE TotalSteps >= 8000 GROUP BY Id
ORDER BY Id')
```
After using the sqldf() function, I see that there are 31 users out of the 33 total users who completed 8,000 steps a day. This is based on the discovery that there are 33 participants included in the data and not 30. If you look closely at how many times each person completed 8,000 steps each day, it appears relatively low for most users. Most adults should aim to complete 8,000 steps each day, as reccomended by the NIH. It is a common misconception that adults should complete 10,000 steps a day, when health benefits (i.e preventing heart disease) hold true from completing 8,000 steps a day (H.H.S, 2020).

Based on the data from the weight_log .csv file, 8 unique Id's are reported, with each Id containing one or more log entries across various dates. I will use SQL to calcuate the average for Weight (in pounds) and BMI. I will then calculate the averages for Calories burned, Very Active Minutes, and Sedentary Minutes per Id.

#### Finding the average BMIs and average Weights (in Pounds) for each ID
```R
sqldf('SELECT Id,
AVG(BMI) AS avg_bmi,
AVG(WeightPounds) AS avg_weight
 FROM weight_log
 WHERE Id IN (1503960366, 1927972279, 2873212765, 4319703577, 4558609924,5577150313, 6962181067, 8877689391)
 GROUP BY Id')
 ```
 
#### Finding the averages for Calories burned, Very Active Minutes, and Sedentary Minutes per Id
 ```R
sqldf('SELECT Id,
AVG(Calories) AS avg_cal,
AVG(VeryActiveMinutes) AS avg_active_min,
AVG(SedentaryMinutes) AS sed_min
FROM daily_activity
WHERE Id IN (1503960366, 1927972279, 2873212765, 4319703577, 4558609924,5577150313, 6962181067, 8877689391)
 GROUP BY Id')
 ```
After calculating the average weight, BMI, and activity levels for each ID, I copied the results and made a new .csv file for later analysis -> avgdata.csv.

#### Using a left join to return all records from sleep log table and matching records from daily activity 
```R
head(sqldf('SELECT sleep_log.Id, VeryActiveMinutes, SedentaryMinutes, LightlyActiveMinutes, FairlyActiveMinutes, TotalMinutesAsleep
FROM sleep_log AS sleep_log
LEFT JOIN daily_activity AS daily_activity
ON sleep_log.Id = daily_activity.Id and sleep_log.Date = daily_activity.Date
ORDER BY sleep_log.Id, sleep_log.Date'))
```

This query returned 413 rows. I used the head() function to pull the first 6 rows as an example to show what the data looks like. Based on the results generated from this table, I saved it as its own .csv file -> sleepactivitylevel.csv. I will use this file later in my analysis.

#### Checking for duplicate values
```R
sum(duplicated(sleep_activity_level))
```
3
#### Removing duplicate values
```R
sleep_activity_level <- sleep_activity_level %>%
    distinct()%>%
    drop_na()
```
#### Checking for duplicate values
```R
sum(duplicated(sleep_activity_level))
```
0

## SHARE
### Identifying Trends and Relationships

**What percentage of the day is spent on each level of activity?**

```R
library(plotly)
labels = c("SedentaryMinutes", "LightlyActiveMinutes", "FairlyActiveMinutes", "VeryActiveMinutes")
values = c(991.2, 192.8, 13.56, 21.56)

fig <- plot_ly(type='pie', labels=labels, values=values, 
               textinfo='label+percent', title='Activity Level in Minutes',
               insidetextorientation='radial')
fig
```

The pie chart is based on the average values calculated for each activity level. 81.3% of users' days are spent being sedentary, while a total of 18.68% involves some activity (15.8% for Lightly Active Minutes, 1.77% for Very Active Minutes, and 1.11% for Fairly Active Minutes). It is difficult to infer what each of the definitions mean for activity level. While it is reccomended that adults complete 30 minutes of "moderate exercise" everyday, it appears that each adult, on average, is surpassing that amount. Users complete 34.72 minutes of fairly active and very active minutes combined. However, if light activity is to be included within the definition, then 227.52 minutes of total activity are completed.

**What is the relationship between daily number of steps and total calories burned per day?**

```R
ggplot(daily_activity, aes(x=TotalSteps, y=Calories)) + 
  geom_point()+
  geom_smooth(method=lm, se=FALSE)+
labs(title= "Relationship Between Daily Steps and Total Calories Burned")
```

This scatterplot shows there is a positive relationship between total steps taken and calories burned per day. The more steps a user takes throughout the day, the more calories they will burn.

**What is the relationship between calories burned throughout the day and  activity level?**
```R
library("gridExtra")
library("ggpubr")

ggp1 <- ggplot(data=daily_activity, aes(x=Calories, y=VeryActiveMinutes)) +
  geom_point(color="blue") + 
  geom_smooth(method=lm,se= FALSE)
ggp2 <- ggplot(data=daily_activity, aes(x=Calories, y=FairlyActiveMinutes)) +
  geom_point(color="orange") +
  geom_smooth(method=lm,se= FALSE)
ggp3 <- ggplot(data=daily_activity, aes(x=Calories, y=LightlyActiveMinutes)) +
  geom_point(color="red") +
  geom_smooth(method=lm,se= FALSE)
ggp4 <- ggplot(data=daily_activity, aes(x=Calories, y=SedentaryMinutes)) +
  geom_point(color="green") +
  geom_smooth(method=lm,se= FALSE)
grid.arrange(ggp1, ggp2, ggp3, ggp4, ncol=4, top= "Relationship Between Calories Burned and Activity Level")
```

There is a positive correlation between Very Active Minutes and Calories, Fairly Active Minutes and Calories, and Lightly Active Minutes and Calories. The more minutes throughout the day that were spent involving some level of activity, the more calories were burned. Opposingly, there is a negative relationship between Sedentary Activity and Calories burned throughout the day. Therefore, the more minutes users' spend being sedentary, the less calories they are burning throughout the day.

**How does activity level affect users' BMI?**

```R
ggp1 <- ggplot(data=averaged_data, aes(x=SedentaryMin, y=AvgBMI)) +
  geom_point(color="green", cex=4) + 
  geom_smooth(method=lm,se= FALSE)
ggp2 <- ggplot(data=averaged_data, aes(x=VeryActiveMin, y=AvgBMI)) +
  geom_point(color="red", cex=4) +
  geom_smooth(method=lm,se= FALSE)
grid.arrange(ggp1, ggp2, ncol=2, top= "Relationship Between Activity Level and BMI")
```

Based on the two scatterplots above, there is a positive relationship between users' average Sedentary Minutes and average BMI and a negative relationship between users' average Very Active Minutes and average BMI. The more time users spend in their day being sedentary, the higher their BMI. On the other hand, the more time users spend being very active, the lower their BMI. A limitation to this analysis is based on there being only 8 out of 33 users actively calculating their BMI's and recording their weights. It may be difficult to drawn conclusions based on this small sample size. However, physical Activity tends to have an inverse relationship with Body Mass Index and studies have shown that the more time dedicated towards physical actvitity, the lower an indivdual's BMI will be (Bradbury et al., 2017).

**Does Activity Level Influence Longer Sleep Time?**
```R
ggp1 <- ggplot(data=sleep_activity_level, aes(x=TotalMinutesAsleep, y=VeryActiveMinutes)) +
  geom_point(color="blue") + 
  geom_smooth(method=lm,se= FALSE)
ggp2 <- ggplot(data=sleep_activity_level, aes(x=TotalMinutesAsleep, y=LightlyActiveMinutes)) +
  geom_point(color="red") +
  geom_smooth(method=lm,se= FALSE)
ggp3 <- ggplot(data=sleep_activity_level, aes(x=TotalMinutesAsleep, y=SedentaryMinutes)) +
  geom_point(color="green") +
  geom_smooth(method=lm,se= FALSE)
ggp4 <- ggplot(data=sleep_activity_level, aes(x=TotalMinutesAsleep, y=FairlyActiveMinutes)) +
  geom_point(color="orange") + 
  geom_smooth(method=lm,se= FALSE)
grid.arrange(ggp1, ggp2, ggp3, ggp4, ncol=4)
```

Three out of four scatterplots reveal a negative correlation. Opposingly, there is a positive correlation for lightly active minutes and total minutes asleep. Activity level does not impact how long a user remains asleep for, unless they have increased minutes dedicated towards being lightly active. As discovered earlier, users' dedicate more time being lightly active than being fairly active or very active, so duration of activity may mediate this relationship. But, perhaps the time of day a user exercises explains why there is a negative corelation between being very active or fairly active. Although aerobic exercise promotes better sleep quality, strenuous exercise before beddtime should be avoided as it may not allow body temperature to cool, which thereby may delay sleep and/or affect sleep quality (Sleep Foundation, 2020).

## ACT
In conclusion, users' positively benefit from some level of activity, as opposed to being sedentary. Users' who dedicated more time towards being active burned more calories throughout the day and overall had lower BMI's. Sleep was not an important variable in this analysis. The dataset, however, contained several limitations. The data was not accurately described, containing 3 more people in the dataset than listed. The methodology used for selecting participants was not stated and we do not know if participants were fairly chosen. Lastly, not every user included in the analysis logged necessary information, such as the logs for weight and sleep. For future analysis, this study should be replicated to include a larger and randomized sample with all users' consenting to track all required information for the full 30 days.

**Key findings from my analysis:**
* A positive relationship between daily number of steps taken and total calories burned per day.
* A series of positive relationships between Calories burned per day and Very Active Minutes, Calories burned per day and Fairly Active Minutes, and Calories burned per day and Lightly Active Minutes.
* A negative relationship between Sedentary Minutes and Calories burned per day.
* A positive relationship between Sedentary Minutes and BMI.
* A negative relationship between Very Active Minutes and BMI.
* A positive relationship between Lightly Active Minutes and Total Minutes Asleep.
* A series of negative relationships bbetween Very Active Minues and Total Minutes Asleep, Fairly Active Minutes and Total Minutes Asleep, and Lightly Active Minutes and Total Minutes Asleep. 
​

**Reccomendations for Bellabeat**
1. Provide an incentive for users' to update weight and sleep logs on a daily basis
2. Allow users' to share activity with friends
3. Include daily and weekly challenges to encourage users' to exercise
4. Add a feature that incorprates daily food intake
5. Allow users' to add goals to complete (i.e weight loss, better sleep habits)
6. Alerting users' to be active when they have been sedentary for too long

### Works Cited

Bradbury, K. E., Guo, W., Cairns, B. J., Armstrong, M. E., & Key, T. J. (2017). *Association between physical activity and body fat percentage, with adjustment for BMI: a large cross-sectional analysis of UK Biobank.* BMJ open, 7(3), e011843.

Sleep Foundation. (2020). *The best time of day to exercise for sleep.*

U.S. Department of Health and Human Services. (2020). *Number of steps per day more important than step intensity.* National Institutes of Health. 
