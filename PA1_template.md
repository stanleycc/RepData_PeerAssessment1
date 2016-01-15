# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. 𝚛𝚎𝚊𝚍.𝚌𝚜𝚟())
2. Process/transform the data (if necessary) into a format suitable for your analysis

---

Unzip the activity.zip.

```r
filename <- "activity.zip"
if (file.exists(filename)) {
    unzip(filename)
}
```

Read activity.csv by ``read.csv()`` and transform column 'date' into Date class.

```r
dt <- read.csv("./activity.csv")
dt$date <- as.Date(dt$date)
```

Import dplyr and ggplot2 for following processes.


```r
library(dplyr)
library(ggplot2)
```

---

## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day
2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
3. Calculate and report the mean and median of the total number of steps taken per day

---

With library ``dplyr``, calculate the total steps on each day. 


```r
res <- dt %>% group_by(date) %>% summarise(steps = sum(steps))
```

Make a histogram based on ggplot2.


```r
g <- ggplot(res,aes(steps))
g + geom_histogram(binwidth = 800)
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)\

The mean of the total number of steps taken per day.


```r
mean(res$steps,na.rm=TRUE)
```

```
## [1] 10766.19
```

The median of the total number of steps taken per day.


```r
median(res$steps,na.rm=TRUE)
```

```
## [1] 10765
```

---

## What is the average daily activity pattern?

1. Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

---

Group data table by column ``interval``. Calcuate average steps per group. Make line plot by ggplot2.


```r
res <- dt %>% group_by(interval) %>% summarise(steps = mean(steps,na.rm=TRUE))
g <- ggplot(res,aes(interval,steps))
g + geom_line() + 
    xlab("5-minute Interval") + 
    ylab("Average Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)\

Locate the row with the maximum number of steps.


```r
res[which.max(res$steps),]$interval
```

```
## [1] 835
```

---

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as 𝙽𝙰). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

---

Calculate the number of NAs by ``summary()``.


```r
summary(dt$steps)[7]
```

```
## NA's 
## 2304
```

Choose the mean value for that 5-minute interval to fill the missing values.


```r
res <- dt %>% group_by(interval) %>% summarise(steps = mean(steps,na.rm=TRUE))
dt2 <- dt %>% mutate(avg_steps=rep(res$steps,nrow(dt)/nrow(res)))
dt2[is.na(dt2$steps),]$steps <- dt2[is.na(dt2$steps),]$avg_steps
dt2 <- dt2 %>% select(steps,date,interval)

summary(dt2$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.00    0.00    0.00   37.38   27.00  806.00
```

Make a histogram from new data table and cacluate the mean and median total number of steps taken per day.


```r
res <- dt2 %>% group_by(date) %>% summarise(steps = sum(steps))
g <- ggplot(res,aes(steps))
g <- ggplot(res,aes(steps))
g + geom_histogram(binwidth = 800)
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)\


```r
mean(res$steps,na.rm=TRUE)
```

```
## [1] 10766.19
```


```r
median(res$steps,na.rm=TRUE)
```

```
## [1] 10766.19
```

The impact of imputing missing data looks minor. Especially, the mean total numbers of steps taken per day from each data set are the same.

---

## Are there differences in activity patterns between weekdays and weekends?

For this part the 𝚠𝚎𝚎𝚔𝚍𝚊𝚢𝚜() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
2. Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

---

Use ``weekdays()`` to identify weekdays and weekends in data table. Make a line plot to observe the difference.


```r
tmp <- weekdays(dt2$date) %in% c("Saturday","Sunday")
tmp[tmp == TRUE] <- 'Weekend'
tmp[tmp == FALSE] <- 'Weekday'

res <- dt2 %>% mutate( weekend = as.factor(tmp)) %>% 
    group_by(interval,weekend) %>% summarise(steps = mean(steps))

g <- ggplot(res,aes(interval,steps))
g + geom_line() + 
    facet_grid(weekend ~ .) +
    xlab("5-minute Interval") + 
    ylab("Average Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)\
