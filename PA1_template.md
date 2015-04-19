# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data

    Show any code that is needed to

    1. Load the data(i.e. `read.csv()`)


```r
rawdata <- read.csv("./activity.csv")
```

    2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
#no processing needed
```


## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day

```r
#names(totalSteps) <- c("date", "total_steps")
totalSteps <- aggregate(steps ~ date, data = rawdata, FUN=sum, na.rm=TRUE)
names(totalSteps) <- c("Date", "Steps")
```

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
hist(totalSteps$Steps, main="Histogram of total steps by day", xlab="Total steps/day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day

**Mean**:

```r
mean(totalSteps$Steps)
```

```
## [1] 10766.19
```

**Median**:

```r
median(totalSteps$Steps)
```

```
## [1] 10765
```



## What is the average daily activity pattern?

1. Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
avgSteps <- aggregate(steps ~ interval, data = rawdata, FUN=mean, na.rm=TRUE)
names(avgSteps) <- c("interval", "avg_steps")
plot(avgSteps, type = "l", main="Average Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
maxSteps <- which.max(avgSteps$avg_steps)
avgSteps$interval[maxSteps]
```

```
## [1] 835
```


## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
missingVals <- sum(is.na(rawdata))
missingVals
```

```
## [1] 2304
```


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
naVals <- is.na(rawdata$steps)
dailyAvg <- aggregate(steps ~ date, data = rawdata, FUN=mean, na.rm=FALSE)
names(dailyAvg) <- c("date", "steps_avg")

workingdata <- rawdata

for (i in 1:nrow(workingdata)){
  if (naVals[i]==TRUE){
    date <- workingdata$date[i]
    date <- as.Date(date, "%Y-%m-%d")
    if (is.na(dailyAvg$steps_avg[date])){
      workingdata$steps[i] <- 0
    }
    else {
      workingdata$steps[i] <- stepsDay_mean$steps_avg[date]
    }
  }
}
```


3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
filleddata <- aggregate(steps ~ date, data = workingdata, FUN=sum, na.rm=TRUE)
names(filleddata) <- c("date", "total_steps")
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
hist(filleddata$total_steps, main="Histogram of total steps/day", xlab="Total Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png) 


## Are there differences in activity patterns between weekdays and weekends?

For this part the `weekdays()` function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
rawdata$day <- weekdays(as.Date(rawdata$date))
rawdata$dayType <- c("weekday")

for (i in 1:nrow(rawdata)){
    if (rawdata$day[i] == "Saturday" || rawdata$day[i] == "Sunday"){
      rawdata$dayType[i] <- "weekend"
    }
}
```

2. Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.



```r
stepInterval <- aggregate(steps ~ interval + dayType, data=rawdata, FUN=mean)
names(stepInterval) <- c("interval", "dayType", "steps")

library(lattice)
```

```
## Warning: package 'lattice' was built under R version 3.1.3
```

```r
xyplot(steps ~ interval | dayType, stepInterval, type = "l", layout = c(1, 2))
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 

