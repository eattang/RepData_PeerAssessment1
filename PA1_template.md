# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data







```r

data <- read.csv("/Users/eddy/Documents/R/data/activity.csv")

# convert to data.table for ease of manipulation
data <- data.table(data)

hist_data <- data[, sum(steps), by = "date"]
names(hist_data) <- c("date", "sum_per_day")
```

```
## Warning: The names(x)<-value syntax copies the whole table. This is due to
## <- in R itself. Please change to setnames(x,old,new) which does not copy
## and is faster. See help('setnames'). You can safely ignore this warning if
## it is inconvenient to change right now. Setting options(warn=2) turns this
## warning into an error, so you can then use traceback() to find and change
## your names<- calls.
```

```r
setnames(hist_data, names(hist_data)[2], "sum_per_day")


hist(hist_data$sum_per_day)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-21.png) 

```r


# obtaining the mean and median
mean_ <- mean(hist_data$sum_per_day, na.rm = T)
print(mean_)
```

```
## [1] 10766
```

```r

# obtain the median
median <- median(hist_data$sum_per_day, na.rm = T)
print(median)
```

```
## [1] 10765
```

```r


median_data <- data[, median(steps), by = "date"]
```

```
## Error: Column 1 of result for group 2 is type 'double' but expecting type
## 'integer'. Column types must be consistent for each group.
```

```r


# time series calculation
time_series <- tapply(data$steps, data$interval, mean, na.rm = T)

plot(row.names(time_series), time_series, type = "l", xlab = "5-min interval", 
    ylab = "Average across all the days", main = "Average number of steps taken", 
    col = "red")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-22.png) 

```r

# to obtain the maximum time interval

names(which.max(time_series))
```

```
## [1] "835"
```

```r

# finding the total number of missing data

data_missing_data <- sum(is.na(data))




StepsAverage <- aggregate(steps ~ interval, data = data, FUN = "mean")
fillNA <- numeric()
for (i in 1:nrow(data)) {
    obs <- data[i, ]
    if (is.na(obs$steps)) {
        steps <- subset(StepsAverage, interval == obs$interval)$steps
    } else {
        steps <- obs$steps
    }
    fillNA <- c(fillNA, steps)
}

new_data <- data
new_data$steps <- fillNA

# Now we have updated the data replacing the NA points with the mean #of the
# data for each level

# use previous approach of the data.table

hist_data2 <- new_data[, sum(steps), by = "date"]
names(hist_data2) <- c("steps", "sum")
```

```
## Warning: The names(x)<-value syntax copies the whole table. This is due to
## <- in R itself. Please change to setnames(x,old,new) which does not copy
## and is faster. See help('setnames'). You can safely ignore this warning if
## it is inconvenient to change right now. Setting options(warn=2) turns this
## warning into an error, so you can then use traceback() to find and change
## your names<- calls.
```

```r
# finally the histogram
hist(hist_data2$sum, breaks = 5)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-23.png) 

```r

# the date is a factor so tapply comes in handy here
sum_steps_per_day <- tapply(new_data$steps, new_data$date, FUN = "sum")

# to get the mean
net_mean_steps <- mean(sum_steps_per_day)
print(net_mean_steps)
```

```
## [1] 10766
```

```r


# to get the median
net_median_steps <- median(sum_steps_per_day)
print(net_median_steps)
```

```
## [1] 10766
```

```r
# 10766 is computed value of the median

# Replacing the missing data with a fixed data did not change the mean but
# removed the bias associated with the median.

# The date column is formatted to a date object for processing
data$date <- as.Date(data$date)
days <- weekdays(data$date)
# declare an empty vector to store variables of the day levels
day_level <- vector()

# the function iterates through all the rows of the data frame
for (i in 1:nrow(data)) {
    if (days[i] == "Saturday") {
        day_level[i] <- "Weekend"
    } else if (days[i] == "Sunday") {
        day_level[i] <- "Weekend"
    } else {
        day_level[i] <- "Weekday"
    }
}
data$daylevel <- day_level
data$daylevel <- factor(data$daylevel)

stepsByDay <- aggregate(steps ~ interval + daylevel, data = data, FUN = "mean")
names(stepsByDay) <- c("interval", "daylevel", "steps")
xyplot(steps ~ interval | daylevel, stepsByDay, type = "l", layout = c(1, 2), 
    xlab = "Interval", ylab = "Number of steps")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-24.png) 

```r


## Are there differences in activity patterns between weekdays and weekends?

# For the weekend, we observe that there are significant and more frequent
# spikes of high activity indicating that there is more time to make such
# measurements.
```



