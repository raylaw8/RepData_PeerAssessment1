---
title: 'Reproducible Research: Peer Assessment 1'
output:
  html_document:
    keep_md: yes
  pdf_document: default
---

## Clean up workspace

```r
rm(list=ls())
```
## set working directory 
```
setwd("D:\\Documents\\Course5project1");

```r
## Loading and preprocessing the data
## Unzip of data
```
outDir="D:\\Documents\\Course5project1\\data"
unzip("./data/repdata_data_activity.zip",exdir=outDir)
```
### Read in the data from files
### 1. Load the data (i.e. read.csv())
### 2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
activity = na.omit(read.csv("./data/activity.csv",header=TRUE)) 
activity$date = as.Date(activity$date)
```
## What is mean total number of steps taken per day?
### 1. Calculate the total number of steps taken per day

```r
agg = aggregate(x=activity$steps, by=list(date=activity$date), FUN=sum)
```
### 2. If you do not understand the difference between a histogram and a barplot, research the difference between 
###    them. Make a histogram of the total number of steps taken each day

```r
hist(agg$x, xlab="Total Steps", ylab="Frequency", main="Total Number of Steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->
### 3. Calculate and report the mean and median of the total number of steps taken per day

```r
print(mean(agg$x, na.rm=TRUE))
```

```
## [1] 10766.19
```

```r
print(median(agg$x, na.rm=TRUE))
```

```
## [1] 10765
```
## What is the average daily activity pattern?
### 1. Make a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and 
###    the average number of steps taken, averaged across all days (y-axis)

```r
aggPerInterval = aggregate(steps~interval, data=activity, mean)
plot(steps~interval, data=aggPerInterval, type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->
### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
print(aggPerInterval[which.max(aggPerInterval$steps),]$interval)
```

```
## [1] 835
```
## Imputing missing values
### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs

```r
activity_NA = read.csv("./data/activity.csv",header=TRUE)
print (sum(is.na(activity_NA$steps)))
```

```
## [1] 2304
```
### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be   sophisticated. 
###    For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
activity_NA$FillNA = ifelse(is.na(activity_NA$steps), round(aggPerInterval$steps[match(activity$interval, aggPerInterval$interval)],0), activity_NA$steps)
```
### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activity_ALL = data.frame(steps=activity_NA$FillNA, interval=activity_NA$interval, date=activity_NA$date)
```
### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of
###    steps taken per day. Do these values differ from the estimates from the first part of the assignment? 
###    What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
aggactivity_ALL = aggregate(steps ~ date, data=activity_ALL, sum)
hist(aggactivity_ALL$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

```r
print(mean(aggactivity_ALL$steps))
```

```
## [1] 10765.64
```

```r
print(median(aggactivity_ALL$steps))
```

```
## [1] 10762
```
## Are there differences in activity patterns between weekdays and weekends?
### 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating 
###    whether a given date is a weekday or weekend day.

```r
activity_ALL$weekday = weekdays(as.Date(activity_ALL$date))
activity_ALL$Daytype = ifelse(activity_ALL$weekday=='Saturday' | activity_ALL$weekday=='Sunday', 'weekend','weekday')
stepsByDaytype <- aggregate(activity_ALL$steps ~ activity_ALL$interval + activity_ALL$Daytype, activity_ALL, mean)
```
### 2. Make a panel plot containing a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval 
###    (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 
###    See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
names(stepsByDaytype) <- c("interval", "Daytype", "steps")
library(lattice)
xyplot(steps ~ interval | Daytype, stepsByDaytype, type = "l", layout = c(1, 2), 
    xlab = "Interval", ylab = "Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->
