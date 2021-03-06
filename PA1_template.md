# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
setwd("./activity")
raw_data <- read.csv("activity.csv")
head(raw_data)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```
## What is mean total number of steps taken per day?

- Calculate the total number of steps taken per day

```r
dates<-unique(raw_data$date)
sum_steps<-seq(0,0,length=61)
for (i in 1:61){
  sum_steps[i]<-sum(raw_data[raw_data$date == dates[i], "steps"])
}
```


- Make a histogram of the total number of steps taken each day

```r
hist(sum_steps, main="Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 


- Calculate and report the mean and median of the total number of steps taken per day

```r
mean_steps<-mean(sum_steps[!is.na(sum_steps)])
mean_steps
```

```
## [1] 10766.19
```

```r
median_steps<-median(sum_steps[!is.na(sum_steps)])
median_steps
```

```
## [1] 10765
```
The mean is 1.0766189\times 10^{4} and the median is 1.0765\times 10^{4}

## What is the average daily activity pattern?
- Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).

```r
clean_data<-raw_data[!is.na(raw_data$steps),]
minutes<-unique(clean_data$interval)
avg_steps<-seq(0,0,length=length(minutes))
for (i in 1:length(minutes)){
  avg_steps[i]<-mean(clean_data[clean_data$interval == minutes[i], "steps"])
}
plot(1:length(minutes), avg_steps, type="l", xlab="5-Minute Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

- Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_minute<-minutes[avg_steps==max(avg_steps)]
max_interval<-which(minutes==max_minute)
```
On average, the maximum number of steps is on the 104th 5-minute interval of the day, when the time-interval value equals 835. 

## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

- Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
na_id <- raw_data[is.na(raw_data$steps), "steps"]
number_na <- length(na_id)
number_na
```

```
## [1] 2304
```
There are 2304 missing values in the dataset.

- Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

- Create a new dataset that is equal to the original dataset but with the missing data filled in


```r
# Identify which index have NA values
na_index <- which(is.na(raw_data[,"steps"]))
pre_data <- raw_data

# Change the NA values by the average value in that 5-minute interval
for (i in na_index){
     pre_data[i,"steps"]<-avg_steps[which(minutes == pre_data[i,"interval"])]
}
```
- Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
dates_pre<-unique(pre_data$date)
sum_steps_pre<-seq(0,0,length=length(dates_pre))
for (i in 1:length(dates_pre)){
  sum_steps_pre[i]<-sum(pre_data[pre_data$date == dates_pre[i], "steps"])
}
hist(sum_steps_pre, main="Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

```r
mean_steps_pre<-mean(sum_steps_pre[!is.na(sum_steps_pre)])
mean_steps_pre
```

```
## [1] 10766.19
```

```r
median_steps_pre<-median(sum_steps_pre[!is.na(sum_steps_pre)])
median_steps_pre
```

```
## [1] 10766.19
```
The mean is 1.0766189\times 10^{4} and the median is 1.0766189\times 10^{4}.
Imputing missing data has a very small impact on the mean and the median (bigger in the median that in the mean).

## Are there differences in activity patterns between weekdays and weekends?
For this part the `weekdays()` function may be of some help here. Use
the dataset with the filled-in missing values for this part.

- Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
wday<-weekdays(as.Date(pre_data$date), abbreviate="False")
dayc<-wday
dayc[wday!="dissabte" & wday!="diumenge"]<-"weekday" # Dissabte stands for Saturday and Diumenge stands for Sunday.
dayc[wday=="dissabte" | wday=="diumenge"]<-"weekend"
day<-as.factor(dayc)
post_data <- cbind(pre_data, day)
head(post_data)
```

```
##       steps       date interval     day
## 1 1.7169811 2012-10-01        0 weekday
## 2 0.3396226 2012-10-01        5 weekday
## 3 0.1320755 2012-10-01       10 weekday
## 4 0.1509434 2012-10-01       15 weekday
## 5 0.0754717 2012-10-01       20 weekday
## 6 2.0943396 2012-10-01       25 weekday
```

- Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
minutes<-unique(post_data$interval)
avg_steps_weekday<-seq(0,0,length=length(minutes))
avg_steps_weekend<-seq(0,0,length=length(minutes))
for (i in 1:length(minutes)){
  avg_steps_weekday[i]<-mean(post_data[(post_data$interval == minutes[i])|(post_data$day=="weekday"), "steps"])
  avg_steps_weekend[i]<-mean(post_data[(post_data$interval == minutes[i])|(post_data$day=="weekend"), "steps"])
}
par(mfrow=c(2,1))
plot(minutes, avg_steps_weekday, type="l", xlab="5-Minute Interval", main="Average steps during the weekdays")
plot(minutes, avg_steps_weekend, type="l", xlab="5-Minute Interval", main="Average steps during the weekends")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 
