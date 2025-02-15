---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
## Libraies Used

```r
library(zip)
library(dplyr)
library(ggplot2)
library(lubridate)
```

## Loading and preprocessing the data

```r
unzip("activity.zip")
# Read and convert data into a data frame
activityData <- tbl_df(read.csv("activity.csv"))
# Convert Dates variable from char to Date
activityData$date <- ymd(activityData$date)
```

## What is mean total number of steps taken per day?

```r
# Group activityDate by it's Date
activityData <- group_by(activityData,date)
# Summarize data to contain total steps per day
StepsPerDay <- summarise(activityData, steps = sum(steps,na.rm = TRUE))
# Plot Data on a Histogram
ggplot(StepsPerDay,aes(y = steps, x = date))+ 
  geom_histogram(stat = "identity")+
  labs(title = "Total Steps Taken per Day", x = "Date", y = "Total Steps")+
  geom_hline(aes(yintercept =  mean(steps)), col = "red")+
  geom_text(aes(label = round(mean(steps)),
                y = mean(steps)+1000,
                x = ymd("2012-11-1")) , col = "red")+
  geom_hline(aes(yintercept =  median(steps)), col = "blue")+
  geom_text(aes(label = round(median(steps)),
                y = median(steps)+1000,
                x = ymd("2012-10-1")) , col = "blue")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
print(paste("Red line = Mean = ",round(mean(StepsPerDay$steps))))
```

```
## [1] "Red line = Mean =  9354"
```

```r
print(paste("Blue line = Median = ",round(median(StepsPerDay$steps))))
```

```
## [1] "Blue line = Median =  10395"
```

## What is the average daily activity pattern?

```r
# Summarize data to contain total steps per day
AverageStepsPerDay <- summarise(subset(activityData, !is.na(steps)), AvgSteps = mean(steps))
# Plot Data on a Histogram
ggplot(AverageStepsPerDay,aes(y = AvgSteps, x = date))+
  geom_line()+
  labs(title = "Average Total Steps Taken per 5 min Interval per Day", x = "Date", y = "Average Total Steps")+
  geom_hline(aes(yintercept =  max(AvgSteps)), col = "red")+
  geom_text(aes(label = round(max(AvgSteps)),
                 y = max(AvgSteps)-5,
                 x = ymd("2012-11-1")) , col = "red")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->
The number in red, 74, represents the maximum number of average steps taken per 5 minute interval.


## Imputing missing values

```r
# Count all NA records
NumMissingVal <- sum(is.na(activityData$steps))
print(paste("Number of missing Values = ",NumMissingVal))
```

```
## [1] "Number of missing Values =  2304"
```

```r
# Group activityDate by it's Date
activityDataTemp <- group_by(activityData,date)

# Replace all NA records number of steps with the average of that day
for (index in length(AverageStepsPerDay$date)) 
{
        activityDataTemp$steps <- replace(activityDataTemp$steps,
                                          is.na(activityDataTemp[activityDataTemp$date == AverageStepsPerDay$date[index],]$steps),
                                          AverageStepsPerDay$AvgSteps[index])
}
# Replace remaining dates that only contains NA with 0
activityDataTemp$steps <- replace(activityDataTemp$steps,
                                  is.na(activityDataTemp$steps),
                                  0)

# Summarize data to contain total steps per day
StepsPerDay <- summarise(activityDataTemp, steps = sum(steps))
# Plot Data on a Histogram
ggplot(StepsPerDay,aes(y = steps, x = date))+ 
  geom_histogram(stat = "identity")+
  labs(title = "Total Steps Taken per Day", x = "Date", y = "Total Steps")+
  geom_hline(aes(yintercept =  mean(steps)), col = "red")+
  geom_text(aes(label = round(mean(steps)),
                y = mean(steps)+1000,
                x = ymd("2012-11-1")) , col = "red")+
  geom_hline(aes(yintercept =  median(steps)), col = "blue")+
  geom_text(aes(label = round(median(steps)),
                y = median(steps)+1000,
                x = ymd("2012-10-1")) , col = "blue")
```

![](PA1_template_files/figure-html/code-1.png)<!-- -->

```r
print(paste("Red line = Mean = ",round(mean(StepsPerDay$steps))))
```

```
## [1] "Red line = Mean =  9354"
```

```r
print(paste("Blue line = Median = ",round(median(StepsPerDay$steps))))
```

```
## [1] "Blue line = Median =  10395"
```
Graph Stayed the same because all NA values existed in dates were all steps were NA. Thus all the NA would have been replaced with 0. If there were missing values between the dates that actually had some steps, then those values would have been replaced with the average number of steps per interval for that day. This will increase the total number of steps, but the overall relationship between all the days would be more or less same.  

## Are there differences in activity patterns between weekdays and weekends?

```r
# create new factor with 2 levels "weekday" and "weekend"
tempWeekdays <- wday(activityDataTemp$date,week_start = 1)
tempWeekdays <- replace(tempWeekdays,tempWeekdays < 5,"weekday")
tempWeekdays <- replace(tempWeekdays,tempWeekdays %in% c(5,6,7),"weekend")
activityDataTemp$DayType <- factor(tempWeekdays,labels = c("weekday","weekend") )

# Summarize data to contain total steps per day
activityDataTemp<- group_by(activityDataTemp,date,DayType)
AverageStepsPerDay <- summarise(activityDataTemp, AvgSteps = mean(steps))
```

```
## `summarise()` has grouped output by 'date'. You can override using the `.groups` argument.
```

```r
# Plot Data
qplot(data = AverageStepsPerDay,y = AvgSteps, x = date,facets = DayType~.)+
  geom_line()+
  labs(title = "Average Total Steps Taken per 5 min Interval per Day", x = "Date", y = "Average Total Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->
