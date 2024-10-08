<h1>Bellabeat Case Study  🏃🏽‍♀️⌚</h1>
<p>Carmen Chow
<p>September 2024

<h2>Background</h2>
<p>Bellabeat is a high-tech manufacturer of health-focused products for women.  Founded in 2013, Bellabeat has grown rapidly and quickly positioned itself as a tech-driven wellness company for women.  Urška Sršen, co-founder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. Sršen knows that an analysis of Bellabeat’s available consumer data would reveal more opportunities for growth. She has asked the marketing analytics team to focus on a Bellabeat product and analyze smart device usage data in order to gain insight into how people are already using their smart devices. Then, using this information, she would like high-level recommendations for how these trends can inform Bellabeat's marketing strategy.</p>

<h2>Stakeholders</h2>

*  Urška Sršen: Bellabeat’s co-founder and Chief Creative Officer

*  Sando Mur: Mathematician and Bellabeat’s co-founder</p>

*  Bellabeat marketing analytics team: A team of data analysts responsible for collecting, analyzing, and reporting data that helps guide Bellabeat’s marketing strategy. 

<h2>Ask</h2>
<h3><b>Business Task</b></h3>
Analyze Fitbit tracker usage data to gain insights into how customers use non-Bellabeat smart devices. Then use these insights to shape Bellabeat's marketing strategy and influence Bellabeat customers.

<h2>Prepare</h2>

<h3><b>Data Source</b></h3>

We will look at publicly available [Fitbit Fitness Tracker Data](https://www.kaggle.com/arashnic/fitbit ) from 30 Fitbit users who consented to the submission of personal tracker data. The data includes daily physical activity, step count, calories burned, and hours slept. The dataset, provided under the Mobius license, consists of 18 wide format CSV files of anonymized user information. 

<h3><b>Data Bias and Credibility. Does it ROCCC?</b></h3>

*  <b>R</b>eliable - Low; the data is from only 30 Fitbit participants.

*  <b>O</b>riginal - Low; the data was collected by a third-party provider, Amazon Mechanical Turk.
  
*  <b>C</b>omprehensive - Medium; key demographic details such as gender, age, and location of participants were not collected.
  
*  <b>C</b>urrent - Low; the data is from April to May 2016, making it over 8 years old.

* <b>C</b>ited - Low; not known.
            
<h3><b>Data Limitations</b></h3>

*  Some data sets have a small sample size of 33 Fitbit users, while the weight data has an even smaller sample size of 8 users.
  
*  The data is over 8 years old and therefore no longer current.
  
*  Data points such as the age, gender, overall health of the participants, and hours users wore their Fitbit each day is also missing.

<h3><b>R packages and libraries</b></h3>
<p>We’ll be using <b>R</b> for our data analysis and data visualization. Let’s start by installing and running the following R packages: 

```
# install packages
install.packages("tidyverse")
install.packages("dplyr")
install.packages("ggplot2")
install.packages("tidyr")
install.packages("janitor")
install.packages("readr")
install.packages("lubridate")

# run packages
library(tidyverse)
library(dplyr)
library(ggplot2)
library(tidyr)
library(janitor)
library(readr)
library(lubridate)
```

Out of the 18 available files, we’ll focus on 3. The remaining files either contain information outside the scope of the business task or duplicate data that is already found in `dailyActivity_merged.csv`.

```
dailyActivity_merged.csv
sleepDay_merged.csv
weightLogInfo_merged.csv

```

<h2>Process</h2>
<p>
In R Studio, we'll read and save each CSV file as a data frame:

```
dailyActivity_merged <- read_csv("C:/Users/carme/OneDrive/Desktop/TurkFitBit/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
sleepDay_merged <- read_csv("C:/Users/carme/OneDrive/Desktop/TurkFitBit/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
weightLogInfo_merged <- read_csv("C:/Users/carme/OneDrive/Desktop/TurkFitBit/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
```

We'll use `ls()` to check that the files have been correctly imported into the environment pane of R Studio. Then, we'll use `head()` to return the first few rows of each data frame. Here are the ones from the `sleepDay_merged` data: 

```
          Id SleepDay              TotalSleepRecords TotalMinutesAsleep TotalTimeInBed
       <dbl> <chr>                             <dbl>              <dbl>          <dbl>
1 1503960366 4/12/2016 12:00:00 AM                 1                327            346
2 1503960366 4/13/2016 12:00:00 AM                 2                384            407
3 1503960366 4/15/2016 12:00:00 AM                 1                412            442
4 1503960366 4/16/2016 12:00:00 AM                 2                340            367
5 1503960366 4/17/2016 12:00:00 AM                 1                700            712
6 1503960366 4/19/2016 12:00:00 AM                 1                304            320
```

We'll use `colnames()` to view the column headers to check if there are any formatting issues that need to be addressed. We can see that column names in the `weightLogInfo_merged` data frame need to be changed from camel case to snake case to align with R’s naming conventions.

```> colnames(weightLogInfo_merged)
[1] "Id"             "Date"           "WeightKg"       "WeightPounds"   "Fat"           
[6] "BMI"            "IsManualReport" "LogId"  
```

Each Fitbit user should have a unique `id`. Let's see if that's the case. 

```
activity_ids <- n_distinct(dailyActivity_merged$Id)
print(activity_ids)
[1] 33

sleep_ids <- n_distinct(sleepDay_merged$Id)
print(sleep_ids)
[1] 24

weight_ids <- n_distinct(weightLogInfo_merged$Id)
print(weight_ids)
[1] 8
```

We have data from 33 users in the <i>dailyActivity</i> data frame, 24 in the <i>sleepDay</i> data frame, and data from only 8 users in the <i>weightLogInfo</i> data frame. Due to the smaller sample size, we will not include weight data in our final analysis.

Below is a summary of the data cleansing we'll conduct to clean and transform our raw, unclean data.

<h3><b>Data Cleaning</b></h2>

*  Use the `clean_names()` function to convert column names to snake_case.

```
daily_activity <- clean_names(dailyActivity_merged)
daily_sleep <- clean_names(sleepDay_merged)
```
          
*  Formate dates, create a new `weekdays` variable, and put the days of the week in chronological order</p>

```
daily_activity <- daily_activity %>%
  rename(date = activity_date) %>%
  mutate(date = as_date(date, format = "%m/%d/%Y")) %>%
  mutate(weekday = weekdays(date)) %>% 
  mutate(weekday = ordered(weekday, levels=c("Monday", "Tuesday", "Wednesday", "Thursday","Friday", "Saturday", "Sunday")))
```

*  Split the `sleep_day column` into `hour` and `date` columns.

```
daily_sleep <- daily_sleep %>%
  separate(sleep_day, c("date", "hour"),sep = "^\\S*\\K") %>%
  mutate(date = as.Date(date, format = "%m/%d/%Y"))
```

*  Add a new `min_fall_asleep`  column that calculates the time it takes for participants to fall asleep

```
daily_sleep <- daily_sleep %>%
  mutate( 
    min_fall_asleep = total_time_in_bed - total_minutes_asleep
  )
```
*  Use `distinct()` and `drop_na()` functions to remove duplicate rows and NA values.

```
daily_activity <- daily_activity %>%
  distinct() %>%
  drop_na()

daily_sleep <- daily_sleep %>%
  distinct() %>%
  drop_na()
```
*  Exclude records that show 0 calories burned and 0 steps taken 

```
daily_activity <- daily_activity %>%
  filter(calories > 0 & total_steps > 0)
```

<br>

After processing our data, we are now ready to merge the data frames to better understand the relationships between different dimensions such as sleep, activity level, and total steps. We'll perform a `merge()` based on the common `id` and `date` columns. This will be a left join, which will include all the rows from the `daily_activity` data frame and only the matching rows from the `daily_sleep` data frame.

```
daily_df <-
  merge(
    x = daily_activity, 
    y = daily_sleep,
    by = c ("id", "date"),
    all.x = TRUE
  )
```

<h2>Analyze & Share </h2>

Let's run the `summary()` function to get an overview of our new dataset’s distribution: 

```
  total_steps    total_distance     calories    total_minutes_asleep
 Min.   :    4   Min.   : 0.00   Min.   :  52   Min.   : 58.0       
 1st Qu.: 4923   1st Qu.: 3.37   1st Qu.:1856   1st Qu.:361.0       
 Median : 8053   Median : 5.59   Median :2220   Median :432.5       
 Mean   : 8319   Mean   : 5.98   Mean   :2361   Mean   :419.2       
 3rd Qu.:11092   3rd Qu.: 7.90   3rd Qu.:2832   3rd Qu.:490.0       
 Max.   :36019   Max.   :28.03   Max.   :4900   Max.   :796.0       
                                                NA's   :453
```
```
 sedentary_minutes lightly_active_minutes fairly_active_minutes very_active_minutes
 Min.   :   0.0    Min.   :  0.0          Min.   :  0.00        Min.   :  0.00     
 1st Qu.: 721.5    1st Qu.:146.5          1st Qu.:  0.00        1st Qu.:  0.00     
 Median :1021.0    Median :208.0          Median :  8.00        Median :  7.00     
 Mean   : 955.8    Mean   :210.0          Mean   : 14.78        Mean   : 23.02     
 3rd Qu.:1189.0    3rd Qu.:272.0          3rd Qu.: 21.00        3rd Qu.: 35.00     
 Max.   :1440.0    Max.   :518.0          Max.   :143.00        Max.   :210.00     
 moderate_vigorous_minutes
 Min.   :  0.00           
 1st Qu.:  0.00           
 Median : 26.00           
 Mean   : 37.79           
 3rd Qu.: 61.00           
 Max.   :275.00
```
<br>

<i><b>Key takeaways:</b></i>

*  <b>8,319</b> - the average number of steps taken per day. This number falls below the recommended 10,000 steps. Alos, 25% of participants averaged less than 4,923 steps a day.

*  <b>5.98</b> - the average number of kilometers walked per day. 

*  <b>210.0</b> - the average number of minutes spent being lightly active  

*  <b>14.78</b> - the average number of minutes spent being fairly active 

*  <b>23.02</b> - the average number of minutes spent being very active 

*  <b>15.9 hours</b> (or 955.8 minutes) - the average time spent sedentary.

*  <b>2,361</b> - the average number of calories burned per day, which equates to approximately 98.3 calories burned per hour.

<br>

We’ll use the `ggplot2` package we installed to create visualization of our data that will help us determine if there are positive or negative relationships between different variables. 

<br>

<p><i><b>i) Physical Activity (Step Count)</b></i></p>
Let’s take a look at the relationship between daily step count and daily calories burned. 
<br>
<p>

![calories](https://github.com/user-attachments/assets/d77ea900-2d68-4e6d-9a6f-3674a9428630)

As expected, the chart shows a strong positive correlation between calorie expenditure and step count. This supports the recommendation to walk 10,000 steps a day due to its calorie-burning benefits. Next, let's examine whether the day of the week impacts our participants’ total daily step count.</p>

```
steps_by_day <- daily_df %>%
  group_by(weekday) %>%
  summarise(steps = sum(total_steps)) %>%
  arrange(desc(steps))
print(steps_by_day)
```
Tuesday appears to be the day when Fitbit users walked the most. 

```
  weekday     steps
  <ord>       <dbl>
1 Tuesday   1235001
2 Wednesday 1133906
3 Thursday  1088658
4 Saturday  1010969
5 Friday     938477
6 Monday     933704
7 Sunday     838921
```
<br>

Let's take a look at calories burned throughout the week. We would expect the order of the days to be identical or fairly similar to the days that had the highest step counts. In fact, with the exception of Tuesday, they aren’t. This is likely because calories burned is an aggregate of several factors: calories burned through physical activity (including steps walked and non-step activities like swimming or weight training), calories burned while performing basic functions (Basal Metabolic Rate), and calories burned through non-exercise activity thermogenesis (NEAT), such as fidgeting. 

![avcals](https://github.com/user-attachments/assets/2a700a3c-5b31-4887-9330-44074d9063b9)

<br>
Although the average calories burned doesn't vary much from day to day, there is a slight increase in calorie expenditure on Tuesday and Saturday. 

<p>

We will bring in the `hourlySteps_merged.csv` file to explore hourly trends and determine when users are most active. Let's create a data frame and perform the same data cleansing steps outlined in the <b>Process</b> section of this analysis. 

The `hourlySteps_merged` data frame also includes `33 id`s. Let's return the first 6 rows below.

```
          id activity_hour         step_total
       <dbl> <chr>                      <dbl>
1 1503960366 4/12/2016 12:00:00 AM        373
2 1503960366 4/12/2016 1:00:00 AM         160
3 1503960366 4/12/2016 2:00:00 AM         151
4 1503960366 4/12/2016 3:00:00 AM           0
5 1503960366 4/12/2016 4:00:00 AM           0
6 1503960366 4/12/2016 5:00:00 AM           0
```

We'll perform an additional cleaning step using the `separate()` function to split the `activity_hour` column into two separate columns: `date` and `hour`.  

```
           id date      hour           step_total
        <dbl> <chr>     <chr>               <dbl>
 1 1503960366 4/12/2016 " 12:00:00 AM"        373
 2 1503960366 4/12/2016 " 1:00:00 AM"         160
 3 1503960366 4/12/2016 " 2:00:00 AM"         151
 4 1503960366 4/12/2016 " 3:00:00 AM"           0
 5 1503960366 4/12/2016 " 4:00:00 AM"           0
 6 1503960366 4/12/2016 " 5:00:00 AM"           0
 7 1503960366 4/12/2016 " 6:00:00 AM"           0
 8 1503960366 4/12/2016 " 7:00:00 AM"           0
 9 1503960366 4/12/2016 " 8:00:00 AM"         250
10 1503960366 4/12/2016 " 9:00:00 AM"        1864
```

Now, we'll take a look at the distribution of total steps per day:

![stepcount](https://github.com/user-attachments/assets/caaa6899-74a3-4779-843c-bab3b102c2bd)

We have right skewed histogram that shows 50% of participants get fewer than 8,319 steps a day, which is below the recommended daily step count of 10,000 steps. There are a few data points on the right-hand side showing days when a few users logged a high number of steps, which causes the mean to be higher than the median.

Let's determine when participants are most active throughout the day. We’ll use the `summarize()` and `arrange()` functions to sort the average daily step count from highest to lowest to find the peak hours of activity:


```
avg_steps <- hourly_steps %>%
  group_by(hour) %>%
  summarize(avg_steps = mean(step_total)) %>%
  mutate(avg_steps = as.integer(avg_steps)) %>%
  arrange(desc(avg_steps)) 
```

<br>

We see that the highest average step count was between 5pm and 7pm, peaking at 6 pm with 599 average steps. The next period of high step count happened in the afternoon between 12 and 2 pm. 


<br>

![steps](https://github.com/user-attachments/assets/f0ca58e4-d9d4-44c7-8125-d3f841aee601)

<i><b>ii) Physical Activity (Moderate to Vigorous Activity)</b></i>
<p>Let’s shift our focus to physical activity levels and their effect on the total number of calories burned per day. 
          
<br>

![very](https://github.com/user-attachments/assets/0d294861-b0e4-4a90-8c07-badbf67ebf80)

<br>

![lightly](https://github.com/user-attachments/assets/6801e5d2-43b6-454e-b85d-ec6a363f7025)

<br>

![sed_cals](https://github.com/user-attachments/assets/bc00bd6e-0be0-4e65-b9fd-2532a71bde11)

<br>

Beginning with the first scatterplot, we see a strong positive correlation between minutes of vigorous activity and total calories burned. We can expect people who spend more time participating in vigorous activities to burn more calories throughout the day. Although  the correlation is not as strong, a positive relationship also exists for light activities. Surprisingly, the correlation between sedentary minutes and calories burned only appears negative after 1,000 minutes of sedentary activity.

Since the World Health Organization recommends [150 to 300 minutes of moderate to vigorous aerobic activity per week](https://www.who.int/initiatives/behealthy/physical-activity), we can create a `moderate_vigorous_minutes` variable that combines fairly active and very active minutes.  We’ll plot a histogram to show the frequency of moderate to vigorous minutes over the span of a month and review the updated summary statistics for this column. 

```  
 very_active_minutes moderate_vigorous_minutes
 Min.   :  0.00      Min.   :  0.00           
 1st Qu.:  0.00      1st Qu.:  0.00           
 Median :  7.00      Median : 26.00           
 Mean   : 23.02      Mean   : 37.79           
 3rd Qu.: 35.00      3rd Qu.: 61.00           
 Max.   :210.00      Max.   :275.00
```

![modvig](https://github.com/user-attachments/assets/711ab08c-e267-4708-ab8e-852adc4b7bde)

We again have a right-skewed histogram with most of the data falling on the left-side of the graph. What is very noticeable is the disproportionately high number of participants that recorded <b>0 minutes of moderate to vigorous activity on 300 occasions</b>. Given that our data covers 33 participants over 4 weeks, this equates to roughly <b>one-third of the month that participants did not engage in any moderate to vigorous activity</b>.

```
33 participants x 7 days a week x 4 weeks = 924 total occasions in a month

300/924 x 100 = 32.47%
```

Our right-skewed histogram shows several occasions when participants had <i>hours</i> of moderate to vigorous activity in a single day. However, the histogram also tells us that 50% of the participants had 26 minutes or less of moderate to vigorous activity, with the average time being 37.8 minutes. 

<br>
<b><i>iii) Sedentary Behavior</i></b>
<p>
<p>When we look at the distribution of sedentary minutes in a day, we see that the median is 1,021 minutes, which means <b>50% of participants are sedentary for at least 17.02 hours a day</b>. The majority of participants seem to spend 600 to 1,200 minutes (or 10 to 20 hours) being inactive.</p>

![sedentary](https://github.com/user-attachments/assets/66ccbf2d-b4a4-46fc-95d9-e1265d9b1e6e)

<b><i>iv) Sleep</i></b>
<p>
  
Are participants getting the recommended 7-9 hours of sleep? Let's run the `summary()` function again to get an overview of the distribution of sleep times from our `daily_df` data frame. 
</p>

```
 total_minutes_asleep total_time_in_bed min_fall_asleep 
 Min.   : 58.0        Min.   : 61.0     Min.   :  0.00  
 1st Qu.:361.0        1st Qu.:403.8     1st Qu.: 17.00  
 Median :432.5        Median :463.0     Median : 25.50  
 Mean   :419.2        Mean   :458.5     Mean   : 39.31  
 3rd Qu.:490.0        3rd Qu.:526.0     3rd Qu.: 40.00  
 Max.   :796.0        Max.   :961.0     Max.   :371.00  
 NA's   :453          NA's   :453       NA's   :453
```
<i>Key takeaways</i> 
*  <b>6.9 hours</b> (or 419.5 minutes) - the average sleep time.
*  <b>39.17 minutes</b> - the average time needed to fall asleep.
*  <b>25%</b> of participants are sleeping <b>6 hours</b> or less (361 minutes or less). 
*  The shortest sleep time recorded was 58 minutes while the highest was 796 minutes.

![minsleep](https://github.com/user-attachments/assets/8265026e-b607-4d38-8560-93dfc9ccbc59)

<br>
The National Sleep Foundation recommends 7 to 9 hours of sleep for young adults and adults. The histogram aboves shows that 50% of participants are getting 7.2 hours (432.5 minutes) or more of sleep. However, half of the participants are not getting the recommended minimum, while some are sleeping more than 9 hours. Overall, participants tend to sleep longer on weekends and on Wednesdays.

<br>
<p>

![sleep](https://github.com/user-attachments/assets/c0bd916d-3834-4c3e-8571-741ff1c3e78d)

</p>
<br>

Let's examine the impact of different activity levels on total minutes of sleep in the following four scatterplots:

<p>

![sedentary](https://github.com/user-attachments/assets/1ab3e6c1-0280-4987-a89b-7f99ddabc24d)

![lightly_sleep](https://github.com/user-attachments/assets/90986f51-c931-41bf-96d7-f3b519e6f8ef)

![farily](https://github.com/user-attachments/assets/b2d70e34-8818-48a6-88b5-1f28a6bae33f)

![very_sleep](https://github.com/user-attachments/assets/436e1782-e8a1-4865-af88-ada47368367d)

We see that sedentary minutes negatively impact sleep duration: more time spent being inactive results in less time spent asleep. Conversely, sleep duration improves with more time spent engaging in moderately active and very active activities. 

<h3><b>Final Summary</b></h3>

*  50% of participants take fewer than 8,053 steps per day and 25% take fewer than 4,923.
*  50% of participants sleep less than 7.2 hours a day.
*  50% of participants are sedentary for more than 15.9 hours a day.
*  50% of participants get less than 26 minutes of moderate to vigorous activity a day.
*  For about one-third of the month, participants recorded 0 minutes of moderate to vigorous activity. 

<h2>Act</h2>

The following recommendations are based on the insights gathered from our Fitbit user data analysis.

<h3>Recommendations</h3>
  
*  <b>AI Integration & Health Reporting</b>
Bellabeat should implement an AI health chatbot that can help customers stay on track. This enhanced feature assist in setting goals, answering health questions, intepreting health data, and recommending workouts. Most importantly, users will have access to weekly reports that show how they are progressing across key health and wellness metrics.

*  <b>Built-in Workouts</b>
Bellabeat could introduce moderate to high intensity workouts designed to help all users reach the recommended 150-300 minutes of moderate to vigorous activity per week.  Short workouts that fit into anyone's schedule,  with the message "Every Move Counts" - in reference to the World Health Organization's <i>Every Move Counts</i> initiative - could help users incorporate more active minutes into their day. 

*  <b>Special Features</b>
To motivate users to reach their daily and weekly fitness goals, Bellabeat can reward those who achieve their daily or weekly fitness goals with bonuses such as unlocking exclusive features or inviting participants to participate in special contests and give-aways.

*  <b>Notifications and Reminders</b>
Bellabeat should make participants aware of the dangers of prolonged sedentary behavior by alerting users when no activity has been detected for more than a certain number of hours. Alerts and notifications can also be used to inform users how close or far they are from achieving their daily and weekly goals in terms of steps walked, hours slept, and minutes exercised.

Moving forward, Bellabeat should collect data provided by its actual customers and encourage them to wear their fitness tracker consistently for greater data accuracy. By incorporating these recommendations and continuing to evolve its products, Bellabeat can remain relevant and competitive among other wellness technology companies. 

<h2>References</h2>

<p>National Sleep Foundation's Updated Sleep Duration Recommendations 
<p>https://pubmed.ncbi.nlm.nih.gov/29073398/#:~:text=Seven%20to%209%20hours%20is,is%20recommended%20for%20older%20adults 

<p>World Health Organization Physical Activity Report
<p>https://www.who.int/initiatives/behealthy/physical-activity#:~:text=Should%20do%20at%20least%2060,least%203%20times%20per%20week.

<p>World Health Organization Every Move Counts - Launch of the new WHO Guidelines on Physical Activity and Sedentary Behaviours
<p>https://www.who.int/news-room/events/detail/2020/11/26/default-calendar/webinar-who-2020-guidelines-on-physical-activity-and-sedentary-behaviour

<p>Walking 10K Steps a Day is a Health Sweet Spot
<p>https://www.cbc.ca/radio/quirks/sep-17-10-000-steps-really-are-good-for-you-astronomers-thrilled-by-jwst-garbage-picking-cockatoos-and-more-1.6584419/walking-10k-steps-a-day-is-a-health-sweet-spot-study-finds-and-walking-faster-is-even-better-1.6584679



