---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data


```r
mydata <- read.csv("activity.csv")
mydata$date <- as.Date(mydata$date)
cleanNA <- subset(mydata, !is.na(mydata$steps))
```


## Histogram of the total number of steps each day

```r
totalEachDay <- tapply(cleanNA$steps, cleanNA$date, sum)
hist(totalEachDay, col = "purple", xlab = "Total Steps Each Day", main = "")
```

![](PA1_template_files/figure-html/sumbyday-1.png)<!-- -->

## What is mean total number of steps taken per day?


```r
options(scipen = 999) # disables exponential notation 
meanAcrossDays <- mean(totalEachDay)
```
The mean number of steps taken per day: 10766.1886792


## The median total number of steps taken per day:

```r
medianAcrossDays <- median(totalEachDay)
```
The median number of steps taken per day: 10765

## What is the average daily activity pattern?


```r
avgInterval <-tapply(cleanNA$steps, cleanNA$interval, mean, simplify = TRUE)
df_new <- data.frame(avg = avgInterval, interval = as.integer(names(avgInterval)))
plot(df_new$interval, df_new$avg, type = "l", xlab = "5-minute interval" , ylab = "Average Number Of Steps Across Days")
```

![](PA1_template_files/figure-html/averages-1.png)<!-- -->

Next, we'll find the interval with the highest average number of steps.


```r
maxSteps <- max(df_new$avg)  
dex <- which(df_new$avg == maxSteps, arr.ind = TRUE)
maxInterval <- df_new$interval[dex]
maxSteps <- as.integer(maxSteps)
```
The interval with the highest average number of steps is interval 835 with 206 steps.

## Imputing missing values


```r
totalNA <- sum(is.na(mydata$steps))
```

The total number of missing values is 2304

Since we already have the avereages for each interval calculated in the <code>avgIntervals</code> object, we will use those values for the equivalent intervals in the dataset in which we want to impute the values.


```r
df_imp <- mydata # copy of original dataframe with missing values
cursor <- is.na(df_imp$steps) # stores indices of missing values
# set missing steps to equivalent interval's average steps
df_imp$steps[cursor] <- avgInterval[as.character(df_imp$interval[cursor])]
df_imp$date <- as.Date(df_imp$date) # convert dates from integers
```
Again, we'll make a histogram and calculate the mean and median of steps aross days.


```r
totalImp <- with(df_imp, tapply(steps, date, sum))
hist(totalImp, col = "blue", xlab = "Total Steps per Day", main = "")
```

![](PA1_template_files/figure-html/imphist-1.png)<!-- -->

```r
impMean <- mean(totalImp)
impMed <- median(totalImp)
```

The imputed data mean steps across days calculates to 10766.1886792.

The imputed data median steps across days calculates 
to 10766.1886792


First of all, while the histograms looks very similar in shape and distribution, it is important to notice the difference in scale on the y-axes when comparing the two.  Comparing the imputed data does show a higher frequency of days with more steps toward the middle of the graph, suggesting that there is some sensitivity to the missing data in this respect.  Comparing the mean and median values reveals very little change between the two sets. 

## Are there differences in activity patterns between weekdays and weekends?

**Important to note:** for this last section of the markdown file, we'll be using the imputed dataset, not the original.

First, we must differentiate between weekdays and weekends:


```r
library(lubridate)
    # helper function finds day of week, assigns weekday or weekend
    dow <- function(date){
      
      dayOfWeek <- weekdays(date)
      
      if(dayOfWeek %in% c("Saturday", "Sunday")){
        return("weekend")
      }else{ return ("weekday")} 
      
    }

df_imp$time.of.week <- with(df_imp, sapply(date, FUN = dow))
```

Next, we'll plot the average steps on weekdays and weekends for a comparison of activity levels during these times of the week.


```r
library(ggplot2)
wdayAvg <- aggregate(steps ~ time.of.week + interval, data = df_imp, FUN = mean)
# generate plot
g <- ggplot(wdayAvg, aes(interval, steps, colour = time.of.week))
g <- g + geom_line() +
     facet_grid(time.of.week ~ .) +
     xlab("5-minute Interval") +
     ylab("Average Number of Steps")
print(g)
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

From a comparison of the plots, it looks like there is a spike of activity on weekdays earlier than on weekends, but across the day there is, on average, more activity during the weekend days.
