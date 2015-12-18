# Reproducible Research: Peer Assessment 1

##Loading and preprocessing the data

###Show any code that is needed to

1. Load the data (i.e. read.csv())

```r
data <- read.csv("activity.csv")
```

2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
#Nothing here at the moment.
```

##What is mean total number of steps taken per day?

###For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day

```r
#Aggregating steps by date
date_total <- aggregate(data$steps, by = list(data$date), FUN = sum)
```

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
hist(date_total$x)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day

```r
mean(date_total$x, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(date_total$x, na.rm = TRUE)
```

```
## [1] 10765
```

##What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
#Aggregating by interval
interval_mean <- aggregate(data$steps, by = list(data$interval), FUN = mean, na.rm = TRUE, na.action = NULL)

#making the plot
plot(interval_mean$Group.1, interval_mean$x, type = "l")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
interval_mean[interval_mean$x == max(interval_mean$x), 1]
```

```
## [1] 835
```

##Imputing missing values

###Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
nas <- nrow(data[is.na(data), ])
nas
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

We are going to create a new dataset, add a new column, and then loop over all the rows in the dataset. If the steps
column in that row is NA, we will fill it in with the mean value for that interval calculated above.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
new_data <- data
new_data$steps2 <- new_data$steps
for (i in 1:nrow(new_data)) {
    if (is.na(new_data[i, 1])) {
        interval <- new_data[i, 3]
        new_data[i, 5] <- interval_mean[interval_mean$Group.1 == interval, "x"]
    }
}
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
#Aggregating steps by date, with NAs filled in
new_date_total <- aggregate(new_data$steps2, by = list(new_data$date), FUN = sum)

#Making the histogram
hist(new_date_total$x)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 

```r
#Mean
mean(new_date_total$x, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
#Median
median(new_date_total$x, na.rm = TRUE)
```

```
## [1] 10765
```

These values arew very similar to the original values. It seems that by entering mean values we did not change the overall mean very much.

##Are there differences in activity patterns between weekdays and weekends?

###For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
new_data$day <- weekdays(as.Date(new_data$date, format = "%Y-%m-%d"))
new_data$weekday <- new_data$day
weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
weekends <- c("Saturday", "Sunday")
new_data$weekday[new_data$day %in% weekdays] <- "weekday"
new_data$weekday[new_data$day %in% weekends] <- "weekends"
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
library(lattice)

#Aggregating by interval and weekend/weekday
new_interval_mean <- aggregate(new_data$steps2, 
                               by = list(new_data$interval, new_data$weekday), 
                               FUN = mean, 
                               na.rm = TRUE, 
                               na.action = NULL)
xyplot(x ~ Group.1 | Group.2, data = new_interval_mean, type = "o", layout = c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png) 
