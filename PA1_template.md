---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---



## Load necessary packages

I am loading the dplyr package for further use in the assignment 


```r
# loading the dplyr package 
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

## Loading and preprocessing the data
1. Load the data 
2. Process/transform the data (if necessary) into a format suitable for analysis


```r
unzip <- unzip("activity.zip")
activity <- read.csv(unzip)

# change date variable 
activity$date <- as.Date(activity$date)
```
First unzip the data and then load it.I also converted date variable from factor to date.

## What is mean total number of steps taken per day?
1. Calculate the total number of steps taken per day 
2. Make a histogram of the total number of steps taken each day 
3. Calculate and report the mean and median of the total number of steps taken per day 

Calculate the total number of steps taken per day and make a histogram

```r
# create a new data set without the NAs
activity_rm <- na.omit(activity)
# calculate the total number of steps taken per day 
stepsperday <- activity_rm %>% group_by(date) %>% summarize(total_steps = sum(steps))

# histogram for total number of steps per day 
hist(stepsperday$total_steps,
     xlab = "Number of steps taken in a day", 
     main = "Histogram of number of steps taken in a day")
```

![](PA1_template_files/figure-html/total number and histogram-1.png)<!-- -->


```r
mean_steps <- mean(stepsperday$total_steps)
median_steps <- median(stepsperday$total_steps)
```
The mean total number of steps taken per day is 1.0766189\times 10^{4} and the median total number of steps taken per day is 10765.

## What is the average daily activity pattern?
1. Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains maximum number of steps?


Calculate the average number of steps taken and then make a time series plot 

```r
steps_per_interval <- activity_rm %>% group_by(interval) %>% summarize(average_steps = mean(steps))

plot(steps_per_interval$interval, steps_per_interval$average_steps,
     type = "l",
     xlab = "5-minute interval",
     ylab = "average number of steps taken, averaged across all days",
     main = "Average daily pattern")
```

![](PA1_template_files/figure-html/time series plot-1.png)<!-- -->

Calculate which interval has the maximum steps 


```r
maximum_steps <- steps_per_interval$interval[which.max(steps_per_interval$average_steps)]
```
The 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps is 835.

## Imputing missing values
1. Caculate and report the total number of missing values in the dataset
2. Devise a strategy for filling in all of the missing values in the dataset 
3. Create a new dataset that is equal to the original dataset but with missing data filled in 
4. Make histogram of the total number of steps taken each day and calculate the mean and median total number of steps per day. 


```r
missing_values <- sum(is.na(activity$steps))
```
In the dataset, there are a total of 2304 NAs.

In this case, since we calculated the mean for the 5-minute interval in the previous question; we will use that value and fill it in to replace the missing value. 

```r
new <- activity
# going through each row to find the index and assign value to replace the NA 
for (i in 1:nrow(new)){
  if (is.na(new$steps[i])){
    index <- which(new$interval[i] == steps_per_interval$interval)
    new$steps[i] <- steps_per_interval[index,]$average_steps
  }
}
new$date <- as.Date(new$date)
```


Plot the histogram for the new dataset

```r
new_steps_per_day <- new %>% group_by(date) %>% summarize(new_total_steps = sum(steps))
hist(new_steps_per_day$new_total_steps,
     xlab = "number of steps taken each day",
     main = "Total number of steps taken each day")
```

![](PA1_template_files/figure-html/new histogram-1.png)<!-- -->

Calculate the mean and median of the new dataset

```r
mean_new_steps <- mean(new_steps_per_day$new_total_steps)
median_new_steps <- median(new_steps_per_day$new_total_steps)
```
The mean total number of steps per day for new dataset is 1.0766189\times 10^{4} and the median total number of steps per day for new dataset is 1.0766189\times 10^{4}. The mean value did not differ from the estimates from the first part of the assignment. However after filling in the missing value, the median changed to be equal to the mean value.

## Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with 2 levels indicating whether given date is weekday or weekend 
2. Make a panel plot of 5-minute interval and average number of steps taken across all weekday or weekend 

Created a variable for days of the week which we used to make the 2 levels variable: weekends and weekdays

```r
# created new factor variable 
new$day <- weekdays(new$date)
# indicating below the 2 levels: weekday or weekend 
new$type <- "weekday"
new$type[new$day %in% c("Saturday","Sunday")] <-"weekend"
```

Calculate the average steps taken across all weekday or weekends

```r
average <- new %>% group_by(type, interval) %>% summarize(average_steps = mean(steps))
```

We will use ggplot2 to create the panel plot 

```r
library(ggplot2)

qplot(interval, average_steps, data = average,
      geom="line",
      xlab="5-minute interval",
      ylab= "Average Number of Steps taken, averaged across all weekdays and weekends",
      main= "Average steps taken: Weekdays vs Weekends",
      facets = type ~ .)
```

![](PA1_template_files/figure-html/time series plots-1.png)<!-- -->
