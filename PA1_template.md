---
title: "RR Week 2 Project"
author: "Kevin Clancy"
date: "August 31, 2018"
output: html_document
---


```r
#Read in the dataset
act <- read.csv("./data/activity.csv")

#turn date values from factor to date objects
act$date <- as.Date(as.character(act$date), "%Y-%m-%d")
```

What is the mean total number of steps taken per day? (Ignore missing values)

```r
#Calculate sum of steps taken per day
library(dplyr)

sumsteps <- act %>%
  group_by(date) %>%
  summarise(steps = sum(steps))

#Make a histogram of total number of steps taken each day
hist(sumsteps$steps, xlab = "Number steps taken each day", col = "blue", main = "Steps by Day")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png)

```r
#Calculate and report the mean and median of the total number of steps taken per day
meanmed <- act %>%
  group_by(date) %>%
  summarise(mean = mean(steps, na.rm = TRUE), median = median(steps, na.rm = TRUE))

head(meanmed)
```

```
## # A tibble: 6 x 3
##   date          mean median
##   <date>       <dbl>  <dbl>
## 1 2012-10-01 NaN         NA
## 2 2012-10-02   0.438      0
## 3 2012-10-03  39.4        0
## 4 2012-10-04  42.1        0
## 5 2012-10-05  46.2        0
## 6 2012-10-06  53.5        0
```

What is the average daily activity pattern?

```r
#Make a time series plot(type = "l") of the 5 minute interval (x-axis) and the average number of steps take, averaged across all days

#group by 5-minute interval instead of days
stepint <- act %>%
  group_by(interval) %>%
  summarise(avgstep = mean(steps, na.rm = TRUE))

#make time-series plot
with(stepint,plot(interval, avgstep, type = "l", xlab = "Interval", ylab = "Average # Steps", main = "Average Steps Broken Down by Interval"))
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png)


Imputing missing values

```r
library(dplyr)
#Calculate and report the total number of missing values in dataset
na.act <- is.na(act)

apply(na.act, 2, sum)
```

```
##    steps     date interval 
##     2304        0        0
```

```r
#2304 missing steps

#Devise a strategy for filling in all of missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval

#input mean of 5-minute interval for missing values


meanreplace <- function(x) {
  replace(x, is.na(x), mean(x, na.rm = TRUE))
}
act_imputted <- act %>%
  group_by(interval) %>%
  mutate(steps = meanreplace(steps))

#create function that takes a dataset "x" and for the values that are true in is.na, replaces the values with the mean of the interval (when mean calculated, it is the mean of the interval value)


#Create a dataset wiht missing values replaced (already created in last step)
head(act_imputted)

#Make a histogram of the total number of steps taken each day and calculate teh mean and medain total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

#calculating sum of steps for each date (with imputted values)
sum_finalact <- act_imputted %>%
  group_by(date) %>%
  summarise(steps = sum(steps))

#create histogram of sum dataset
hist(sum_finalact$steps, xlab = "Number Steps by Day", main = "Steps Taken per Day (Imputted Missing Values")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png)

```r
#Calculate mean and median for total number steps taken per day
meanmed_act_imputted <- act_imputted %>%
  group_by(date) %>%
  summarise(mean = mean(steps, na.rm = TRUE), median = median(steps, na.rm = TRUE))

head(meanmed_act_imputted)
```

```
## # A tibble: 6 x 3
##   date         mean median
##   <date>      <dbl>  <dbl>
## 1 2012-10-01 37.4     34.1
## 2 2012-10-02  0.438    0  
## 3 2012-10-03 39.4      0  
## 4 2012-10-04 42.1      0  
## 5 2012-10-05 46.2      0  
## 6 2012-10-06 53.5      0
```

```r
  ##mean and median values are drastically different between the two. Because of how many zeros/NAs there were, median = 0 or NA for almost all the values in the first dataset (with missing values)
```



```r
#Are there any differences in activity patterns between weekdays and weekend?

#Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day

weekdayact <- mutate(act_imputted, DayWeek = weekdays(date))
weekdayact$weektime <- ifelse(weekdayact$DayWeek == "Saturday" | weekdayact$DayWeek == "Sunday", "Weekend", "Weekday")

#Make a panel plot containing a time series plot( type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weeday/weekend days (y-axis).

##First calculate average steps for each interval, broken by weekday/end

final <- weekdayact %>%
  group_by(weektime, interval) %>%
  summarise(mean = mean(steps))

library(ggplot2)

g <- ggplot(final,aes(interval, mean))
g + geom_line() + facet_grid(weektime ~ .) +
  xlab("Interval") + ylab("Average Steps Taken") + ggtitle("Average Steps Taken Across Intervals Broken by Weekday or Weekend")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)

```r
knit2html()
```

```
## Error in file(input, encoding = encoding): argument "input" is missing, with no default
```


