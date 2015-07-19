# Reproducible Research: Peer Assessment 1



## Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. read.csv())


```r
activity <- read.csv(unz("activity.zip", "activity.csv"), colClasses = c( "integer", "character", "integer"))
```

2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
activity$date <- as.Date(activity$date)
```

## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day

```r
activity_per_day <- ddply(activity, .(date), summarize, steps=sum(steps, na.RM=TRUE))
```

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
ggplot(activity_per_day, aes(steps)) + geom_histogram(position="dodge", binwidth=1000) + theme(text = element_text(size=10), axis.text.x = element_text(angle=90, vjust=1))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day

Mean steps per day:

```r
mean(activity_per_day$steps, na.rm=TRUE)
```

```
## [1] 10767.19
```

Median steps per day:

```r
median(activity_per_day$steps, na.rm=TRUE)
```

```
## [1] 10766
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
activity_per_interval <- ddply(activity, .(interval), summarize, steps=mean(steps, na.rm=TRUE))

ggplot(activity_per_interval, aes(interval, steps)) + geom_line(stat="identity", position="dodge") + coord_cartesian(xlim=c(0,max(activity_per_interval$interval)))
```

```
## ymax not defined: adjusting position using y instead
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
activity_per_interval[which.max(activity_per_interval$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```


## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

We will replace all missing values with the mean for the interval.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
missing <- activity
means <- group_by(missing, interval) %>% summarise(mean = mean(steps, na.rm = TRUE))
missing[is.na(missing$steps),]$steps <- left_join(missing,means)[is.na(missing$steps),]$mean
```

```
## Joining by: "interval"
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
missing_per_day <- ddply(missing, .(date), summarize, steps=sum(steps, na.RM=TRUE))
ggplot(missing_per_day, aes(steps)) + geom_histogram(position="dodge", binwidth=1000) + theme(text = element_text(size=10), axis.text.x = element_text(angle=90, vjust=1))
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png) 

Mean with missing values replaced:

```r
mean(missing_per_day$steps, na.rm=TRUE)
```

```
## [1] 10767.19
```

Median with missing values replaced:

```r
median(missing_per_day$steps, na.rm=TRUE)
```

```
## [1] 10767.19
```

The mean remains unchanged since the substituted values were each the mean. The median now matches the mean, due to the large number of entries of the mean value in the midst of the data set. 

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels, "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
missing$isweekend <- as.factor(ifelse(weekdays(missing$date) %in% c('Saturday','Sunday'), "weekend", "weekday"))
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
weekday_per_interval <- ddply(missing, .(isweekend, interval), summarize, steps=mean(steps, na.rm=TRUE))
ggplot(weekday_per_interval, aes(interval, steps)) + geom_line(stat="identity", position="dodge") + coord_cartesian(xlim=c(0,max(weekday_per_interval$interval))) + facet_grid(. ~ isweekend)
```

```
## ymax not defined: adjusting position using y instead
## ymax not defined: adjusting position using y instead
```

![](PA1_template_files/figure-html/unnamed-chunk-16-1.png) 
