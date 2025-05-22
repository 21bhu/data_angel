# Bellabeat Smart Device Usage Analysis

# Overview

This project, completed as part of the Google Data Analytics Professional Certificate, analyzes fitness tracker data from Bellabeat, a high-tech company manufacturing health-focused smart products for women. The primary goal is to identify trends in smart device usage and provide high-level recommendations to inform Bellabeat's marketing strategy for future growth.

# Table of Contents


[Problem Statement](#problem-statement)

## Business Task

## Data Source

## Tools

## Analysis Process

    1. Data Loading and Initial Inspection

    2. Data Cleaning and Preprocessing

    3. Data Exploration and Summary Statistics

    4. Data Merging

    5. Data Visualization

        a. Relationship Between Minutes Asleep and Time in Bed

        b. Relationship Between Total Steps and Calories Burned

        c. Average Total Intensities by Hour of the Day

        d. Steps vs. Sedentary Minutes

Key Insights and Recommendations

Conclusion


## Problem Statement

Bellabeat, while successful, has the potential to expand its presence in the global smart device market. Urška Sršen, cofounder and Chief Creative Officer, believes that analyzing fitness data from smart devices can unlock new growth avenues. The core questions guiding this analysis are:

    What are some trends in smart device usage?

    How could these trends apply to Bellabeat customers?

    How could these trends help influence Bellabeat's marketing strategy?

Business Task

Identify potential growth opportunities and recommendations for Bellabeat's marketing strategy improvement based on observed trends in smart device usage.
Data Source

The analysis utilizes public fitness tracker data available on Kaggle, specifically a subset of the data from the study: "Daily Activity, Sleep, and Weight Log Information from a Wearable Device". The datasets used include:

    dailyActivity_merged.csv

    hourlyCalories_merged.csv

    hourlyIntensities_merged.csv

    minuteStepsNarrow_merged.csv

    minuteSleep_merged.csv

    weightLogInfo_merged.csv

Tools

The analysis was conducted using:

    R (for data manipulation, analysis, and visualization)

    RStudio (Integrated Development Environment)

    Key R packages: dplyr, ggplot2, lubridate, patchwork, viridis

Analysis Process
1. Data Loading and Initial Inspection

The first step involved loading all necessary datasets and performing initial checks to understand their structure and content.

# Load necessary libraries
library(dplyr)
library(ggplot2)
library(patchwork) # For combining plots
library(tidyverse)
library(viridis)
library(lubridate)
library(readr)

# Load your data
dailyActivity_merged <- read.csv("/kaggle/input/bhumikas-portfolio/dailyActivity_merged.csv")
dailyCalories_merged <- read.csv("/kaggle/input/bhumikas-portfolio/hourlyCalories_merged.csv")
dailyIntensities_merged <- read.csv("/kaggle/input/bhumikas-portfolio/hourlyIntensities_merged.csv")
dailySteps_merged <- read.csv("/kaggle/input/bhumikas-portfolio/minuteStepsNarrow_merged.csv")
sleepDay_merged <- read.csv("/kaggle/input/bhumikas-portfolio/minuteSleep_merged.csv")
weightloginfo_merged <- read.csv("/kaggle/input/bhumikas-portfolio/weightLogInfo_merged.csv")

# Display head of dailyActivity_merged
head(dailyActivity_merged)

2. Data Cleaning and Preprocessing

A crucial part of the analysis involved converting ActivityDate columns to a Date format and handling inconsistencies in data types, particularly for numerical columns that were read as characters due to comma delimiters. Missing values were addressed by replacing NA with 0 or omitting rows where appropriate to ensure data integrity for analysis.

# Convert ActivityDate column to Date type
dailyActivity_merged$ActivityDate <- as.Date(dailyActivity_merged$ActivityDate, format="%m/%d/%Y")

# Convert columns from character to numeric, handling any issues with formatting
# This process was done iteratively for all relevant columns,
# including TotalDistance, TrackerDistance, LoggedActivitiesDistance, etc.
dailyActivity_merged$TotalDistance <- as.numeric(gsub(",", "", dailyActivity_merged$TotalDistance))
dailyActivity_merged$TotalDistance[is.na(dailyActivity_merged$TotalDistance)] <- 0

# Similarly for other distance and minutes columns
dailyActivity_merged$TrackerDistance <- as.numeric(gsub(",", "", dailyActivity_merged$TrackerDistance))
dailyActivity_merged$TrackerDistance[is.na(dailyActivity_merged$TrackerDistance)] <- 0

# ... (repeat for other columns like LoggedActivitiesDistance, VeryActiveDistance, etc.)

dailyActivity_merged$VeryActiveMinutes <- as.numeric(gsub(",", "", dailyActivity_merged$VeryActiveMinutes))
dailyActivity_merged$VeryActiveMinutes[is.na(dailyActivity_merged$VeryActiveMinutes)] <- 0

dailyActivity_merged$FairlyActiveMinutes <- as.numeric(gsub(",", "", dailyActivity_merged$FairlyActiveMinutes))
dailyActivity_merged$FairlyActiveMinutes[is.na(dailyActivity_merged$FairlyActiveMinutes)] <- 0

dailyActivity_merged$LightlyActiveMinutes <- as.numeric(gsub(",", "", dailyActivity_merged$LightlyActiveMinutes))
dailyActivity_merged$LightlyActiveMinutes[is.na(dailyActivity_merged$LightlyActiveMinutes)] <- 0

dailyActivity_merged$SedentaryMinutes <- as.numeric(gsub(",", "", dailyActivity_merged$SedentaryMinutes))
dailyActivity_merged$SedentaryMinutes[is.na(dailyActivity_merged$SedentaryMinutes)] <- 0

dailyActivity_merged$Calories <- as.numeric(gsub(",", "", dailyActivity_merged$Calories))
dailyActivity_merged$Calories[is.na(dailyActivity_merged$Calories)] <- 0


# Drop any rows with missing values that couldn't be appropriately handled
dailyActivity_merged <- dailyActivity_merged %>% drop_na()

# Check the structure of the data
str(dailyActivity_merged)

3. Data Exploration and Summary Statistics

Key metrics were explored to understand user behavior patterns. This involved looking at unique IDs, summarizing active minutes, calories burned, sleep patterns, and weight information.

# Explore number of unique participants in each dataset
n_distinct(dailyActivity_merged$Id)
n_distinct(sleepDay_merged$Id)
n_distinct(dailyCalories_merged$Id)
n_distinct(weightloginfo_merged$Id)
n_distinct(dailyIntensities_merged$Id)

# Summarize active minutes per category
dailyActivity_merged %>%
  select(VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes) %>%
  summary()

# Summarize calories burned
dailyCalories_merged %>%
  select(Calories) %>%
  summary()

# Summarize sleep data
sleepDay_merged %>%
  select(TotalSleep, TotalTimeInBed) %>% # Assuming these are the correct column names for sleep duration
  summary()

# Summarize weight data
weightloginfo_merged %>%
  select(WeightKg, BMI) %>%
  summary()

Key Findings from Data Exploration:

    Average Sedentary Time is High: Participants spend approximately 1172 minutes (about 19.5 hours) per day being sedentary, indicating a significant amount of inactivity.

    Predominantly Lightly Active: The average lightly active minutes (22.65 minutes) significantly outweigh very active and fairly active minutes, suggesting most users engage in low-intensity activities.

    Average Sleep Patterns: The average sleep stage value is close to 1, indicating predominantly light sleep and less representation of deeper sleep stages.

    Variability in Calories Burned: The mean calories burned per hour is around 94.27, with a wide range, showing diverse activity levels.

    Low Average Steps: The average daily steps (654) are considerably lower than recommended health guidelines (e.g., 8,000-12,000 steps).

4. Data Merging

To facilitate a comprehensive analysis, relevant datasets were merged based on common identifiers (Id) and Date (after consistent formatting).

# Rename 'ActivityDate' to 'Date' in dailyActivity_merged for consistency
dailyActivity_merged <- dailyActivity_merged %>%
  rename(Date = ActivityDate)

# Convert 'Date' columns to Date type in both dataframes
sleepDay_merged$Date <- as.Date(sleepDay_merged$Date, format="%m/%d/%Y")
dailyActivity_merged$Date <- as.Date(dailyActivity_merged$Date, format="%m/%d/%Y")

# Merge data frames by 'Id' and 'Date'
merged_data <- merge(sleepDay_merged, dailyActivity_merged, by=c('Id', 'Date'))

# Assign to 'data' for further use
data <- merged_data

# Check the merged data structure and column names
print(head(data))
print(colnames(data))

5. Data Visualization

Several visualizations were created to illustrate the trends and relationships within the data.
a. Relationship Between Minutes Asleep and Time in Bed

This plot explores the correlation between the actual time slept and the total time spent in bed.

# Rename columns for clarity (adjust based on actual names if different in your loaded CSV)
names(sleepDay_merged) <- c("Id", "Date", "TotalSleep", "TotalTimeInBed")

# Plot relationship between Total Minutes Asleep and Total Time in Bed
ggplot(data=sleepDay_merged, aes(x=TotalSleep, y=TotalTimeInBed)) +
  geom_point() +
  geom_smooth(method='lm', color='blue') +
  labs(title="Relationship Between Minutes Asleep and Time in Bed",
       x="Total Minutes Asleep",
       y="Total Time in Bed") +
  theme_minimal()

Insight: The correlation between TotalSleep and TotalTimeInBed was found to be 0.0367, indicating a weak relationship. This suggests that simply being in bed for a long time doesn't necessarily translate to more actual sleep for these users.
b. Relationship Between Total Steps and Calories Burned

This visualization demonstrates the intuitive positive correlation between physical activity (steps) and energy expenditure (calories).

# Plot Total Steps vs. Calories with enhanced aesthetics
ggplot(data = dailyActivity_merged, aes(x = TotalSteps, y = Calories)) +
    geom_point(aes(color = Calories), size = 3, alpha = 0.6) +
    geom_smooth(method = "lm", color = "blue", se = FALSE) +
    labs(
        title = "Relationship Between Total Steps and Calories Burned",
        x = "Total Steps",
        y = "Calories Burned"
    ) +
    theme_minimal() +
    theme(
        plot.title = element_text(hjust = 0.5, size = 14, face = "bold"),
        axis.title = element_text(size = 12),
        axis.text = element_text(size = 10),
        panel.grid.major = element_line(linewidth = 0.5, color = "gray"),
        panel.grid.minor = element_blank()
    ) +
    scale_color_viridis_c()

Insight: As expected, there is a positive correlation between Total Steps and Calories Burned. The more active users are, the more calories they tend to burn. However, the data also shows variability, with some users burning fewer calories for similar step counts, highlighting differences in activity intensity or individual metabolism.
c. Average Total Intensities by Hour of the Day

This bar chart highlights the periods of peak and low activity intensity throughout a typical day.

# Ensure 'ActivityHour' and 'Time' are correctly formatted and consistent
# If 'ActivityHour' needs conversion to a proper time format, do it here.
# Assuming hourlyIntensities_merged already has 'ActivityHour' as a factor or can be converted.
hourlyIntensities_merged$ActivityHour <- as.factor(hourlyIntensities_merged$ActivityHour)

# Plot Average Total Intensity by Hour of the Day
ggplot(data = hourlyIntensities_merged, aes(x = ActivityHour, y = AverageIntensity)) +
  geom_bar(stat = "identity", fill = 'darkblue') +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  labs(
    title = "Average Total Intensity by Hour of the Day",
    x = "Hour of the Day",
    y = "Mean Intensity"
  ) +
  theme_minimal() +
  theme(
    plot.title = element_text(hjust = 0.5, size = 14, face = "bold"),
    axis.title = element_text(size = 12),
    axis.text.x = element_text(size = 8),
    axis.text.y = element_text(size = 10),
    panel.grid.major = element_line(linewidth = 0.5, color = "gray"),
    panel.grid.minor = element_blank()
  )

Insight: Activity intensity is generally low during early morning and late night (before 06:00 and after 20:00). Peaks in activity are observed in the late morning and early evening, suggesting these are prime times for user engagement.
d. Steps vs. Sedentary Minutes

This scatter plot investigates the relationship between the number of steps taken and the amount of time spent in sedentary activities.

# Scatter plot without log transformation
ggplot(data = dailyActivity_merged, aes(x = TotalSteps, y = SedentaryMinutes)) +
    geom_point(aes(color = SedentaryMinutes), size = 3, shape = 16, alpha = 0.7) +
    geom_smooth(method = "loess", color = "darkblue", se = TRUE, linewidth = 1.2) +
    labs(
        title = "Steps vs. Sedentary Minutes",
        x = "Total Steps",
        y = "Sedentary Minutes"
    ) +
    theme_minimal() +
    theme(
        plot.title = element_text(hjust = 0.5, size = 16, face = "bold"),
        axis.title = element_text(size = 14),
        axis.text = element_text(size = 12),
        legend.title = element_text(size = 12),
        legend.text = element_text(size = 10)
    )

Insight: This visualization highlights a clear inverse relationship: as sedentary minutes increase, total steps generally decrease. This underscores the significant impact of inactivity on overall daily movement. The data reveals that users spend a large portion of their day being sedentary, with a median of 1229 steps, far below health recommendations.
Key Insights and Recommendations

Based on the analysis, here are key insights and recommendations for Bellabeat's marketing strategy:

    Address High Sedentary Time:

        Insight: Users are highly sedentary (averaging ~19.5 hours/day).

        Recommendation: Bellabeat can develop marketing campaigns that highlight the benefits of breaking up sedentary periods. This could include features like "sedentary alerts" or "movement challenges" within the app. Promote short, frequent activity breaks.

    Encourage Increased Activity Levels:

        Insight: Most users are lightly active, and average daily steps are significantly below health recommendations.

        Recommendation: Market the "active minutes" feature more prominently. Introduce challenges and rewards for achieving moderate and very active minute goals. Highlight how small increases in light activity can contribute to overall health.

    Optimize Engagement During Peak Hours:

        Insight: Activity peaks in the late morning and early evening.

        Recommendation: Schedule push notifications, targeted challenges, and premium content delivery during these high-engagement periods. For example, a "mid-morning movement break" or an "after-work unwind" guided activity.

    Emphasize Quality Sleep:

        Insight: While users spend a lot of time in bed, the actual sleep quality (as inferred from the weak correlation between time in bed and minutes asleep) could be improved, with light sleep dominating.

        Recommendation: Promote Bellabeat's sleep tracking features that go beyond just time in bed. Offer tips and guidance on improving sleep quality, such as consistent sleep schedules, pre-sleep routines, and minimizing screen time before bed. Marketing can focus on the link between quality sleep and overall well-being.

    Personalized Goal Setting:

        Insight: Significant variability exists in individual activity levels and calorie burn.

        Recommendation: Bellabeat should emphasize personalized goal setting based on individual user data. Marketing can highlight how the device adapts to the user's current fitness level and helps them gradually increase activity for sustainable improvement.

Conclusion

This analysis provides valuable insights into the current usage patterns of smart devices, revealing opportunities for Bellabeat to refine its marketing strategy. By focusing on promoting increased activity, reducing sedentary time, optimizing engagement during peak hours, and emphasizing quality sleep, Bellabeat can better meet the needs of its female users and drive growth in the competitive smart device market.
