---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

Load libraries.  Check to see if the csv file is already present in the working 
directory and if not, unzip from the current directory (zip is assummed to be 
present).  csv is then read into the AMD (Activity Monitoring Data) variable.


```r
library(ggplot2)
library(lattice)

if (!file.exists("activity.csv")) {
    unzip("activity.zip")
}

AMD <- read.csv("activity.csv")
```


## What is mean total number of steps taken per day?

Aggregate the steps by day and then produce a histogram showing the frequency
of steps for a day given the sample data.  Next, calc the mean and median.


```r
steps_per_day <- aggregate(steps ~ date, data = AMD, sum, na.rm = TRUE)
hist(steps_per_day$steps, main = "Histogram of Steps per Day", 
     xlab = "Total Steps")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

```r
mean_steps <- mean(steps_per_day$steps)
median_steps <- median(steps_per_day$steps)
```

The mean number of steps per day is 1.0766189 &times; 10<sup>4</sup> and the median steps per day 
is 10765.

## What is the average daily activity pattern?

Aggregate the mean steps per interval across all days.  Nex plot using a line 
plot, Interval vs Steps.  Calculate the interval with the maximum average steps.


```r
steps_per_interval <- aggregate(steps ~ interval, data = AMD, mean, na.rm = TRUE)
plot(steps_per_interval$interval, steps_per_interval$steps, type = "l", 
     xlab = "Interval", ylab = "Average Daily Steps")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

```r
max_step_mean <- steps_per_interval[which.max(steps_per_interval$steps), ]$interval
```

The interval with the highest average steps is interval 835.

## Imputing missing values

Start with counting the number of NA values in the dataset.  The approach to 
filling in the NA values will be to use the mean steps per daily interval which 
is already known from the above exercise.  The for loop will step through each 
row and if an NA is found it will identify the interval and then grab the mean 
interval and replace.


```r
NA_count <- sum(is.na(AMD$steps))
AMD_adj <- AMD

for (i in 1:nrow(AMD_adj)) {
    if (is.na(AMD_adj[i, ]$steps)) {
        ## get the interval position for the NA value
        int_pos <- AMD_adj[i, ]$interval
        ## swap for the daily mean for that interval already calcualted above
        AMD_adj[i, ]$steps <- steps_per_interval[steps_per_interval$interval == int_pos, ]$steps
    }
}

adj_steps_per_day <- aggregate(steps ~ date, data = AMD_adj, sum)
hist(adj_steps_per_day$steps, main = "Histogram of Steps per Day", 
     xlab = "Total Steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

```r
adj_mean_steps <- mean(adj_steps_per_day$steps)
adj_median_steps <- median(adj_steps_per_day$steps)
```

The mean number of steps per day after replacing NAs with the average values for 
the interval is 1.0766189 &times; 10<sup>4</sup> and the median steps per day after replacing 
NAs with the average values for the interval is 1.0766189 &times; 10<sup>4</sup>.

The new histogram with the inferred data is nearly identical except the frequencies 
are slighty higher - which is expected now that we are amplifying the data with 
additinal values.

## Are there differences in activity patterns between weekdays and weekends?

Use the Date and POSIXlt functions to prepare the data for a logic test.  The wday 
component of the POSIXlt object will return the date in terms of the day so the 
week (0-6) with Sunday = 0.  If you use the modulus operator and 6 you can tell 
if the day is either Sunday (0) or Saturday (6) as the remainder will be zero. 
The next steps are to create the factor and a panel plot using the lattice 
library.


```r
AMD_adj$weeknd_or_day <- ifelse(as.POSIXlt(as.Date(AMD_adj$date))$wday%%6 == 
    0, "Weekend", "Weekday")
AMD_adj$weeknd_or_day <- factor(AMD_adj$weeknd_or_day, levels = c("Weekday", 
                                                                  "Weekend"))
steps_per_int_w_days <- aggregate(steps ~ interval + weeknd_or_day, data = AMD_adj, mean)
xyplot(steps ~ interval | factor(weeknd_or_day), data = steps_per_int_w_days, 
       type = "l", xlab = "Interval", ylab = "Steps")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

It's hard to tell, but it looks like the subject might have more intense activity 
during the week (a higher spike) and sleeps in during the weekend.
