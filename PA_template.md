---
---
title: "Week 2 Project 1"
output: html_document
---
### Loading the data

First, the data is loaded in to R and the date column transformed using the following code:

```{r echo=TRUE}
activity <- read.csv("activity.csv")
activity[,2]<-as.Date(activity$date)
```

## Data Analysis

### What is the mean total number of steps taken per day? 

This block of code produces a histogram illustrating total number of steps per day.

```{r echo=TRUE}
steps<-with(activity, tapply(steps, date, sum, na.rm=TRUE))
hist(steps, col="lightblue", xlab="Total Steps", ylab="Frequency", main = "Total Number of Steps Per Day" )
```

```{r echo=TRUE}
mean_steps<-mean(steps)
median_steps<-median(steps)
print(c(mean_steps, median_steps))
```
The average mean total steps taken each day is 9324 and the average median total steps taken each day is 10,395. 

## What is the average daily activity pattern? 

```{r echo=TRUE}
stepsPerInterval<-aggregate(steps~interval, data=activity, mean, na.rm=TRUE)
plot(steps~interval, data=stepsPerInterval,type="l", ylim=c(0, 220), col="blue", xlab="Intervals", ylab="Average Steps", main="Average Steps per Interval")
```
### Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? 

```{r echo=TRUE}
max_intervals<-stepsPerInterval[which.max(stepsPerInterval$steps),]$interval
print(max_intervals)
```

## Imputing missing values

### How many missing values are there in this dataset? 

```{r echo=TRUE}
activity_missing<-sum(is.na(activity$steps))
print(activity_missing)
```

There are 2304 missing values. 

### Strategy for filling in missing values

The following block of code fills in all missing values in the dataset with the mean and saves this dataset as Activity_noNA

```{r echo=TRUE}
MeanPerInterval<-function(interval){
        stepsPerInterval[stepsPerInterval$interval==interval,]$steps
}

activity_noNA<-activity
for(i in 1:nrow(activity_noNA)){
        if(is.na(activity_noNA[i,]$steps)){
                activity_noNA[i,]$steps<-MeanPerInterval(activity_noNA[i,]$interval)
        }
}
```

### Comparing the original dataset with the new dataset with imputed missing data

```{r echo=TRUE}
total_steps_noNA<- aggregate(steps~date, data=activity_noNA, sum)
hist(total_steps_noNA$steps, col="lightblue", xlab="Total Steps", ylab="Frequency", main = "Total Number of Steps Per Day" )

mean_steps_noNA<-mean(total_steps_noNA$steps)
median_steps_noNA<-median(total_steps_noNA$steps)
```
### Do these values differ from the estimates made from the original dataset? 

The above histogram has a very similar shape to the original histogram, but the middle and largest bin has grown in frequency from 26 to 36. The mean and median have also slightly decreased due to the method chosen to impute missing data. 

## Are there differences in activity patterns between weekdays and weekends? 

To answer this, a factor variable ("wk") has been created with two levels - "weekday" and "weekend". 

```{r echo=TRUE}
is_weekday<-function(act_pattern){
        wd<-weekdays(act_pattern)
        ifelse(wd=="Saturday"| wd =="Sunday", "weekend", "weekday")
}

wk2<-sapply(activity_noNA$date, is_weekday)
activity_noNA$wk<-as.factor(wk2)
```
A panel plot will now be created of a timeseries plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days. The lattice package will be used to create this. 

```{r echo=TRUE}
wk_df<-aggregate(steps~wk +interval, data=activity_noNA, FUN=mean)

library(lattice)
xyplot(steps~interval|factor(wk), layout=c(1, 2), xlab="Interval", ylab="Number of Steps", type="l", data=wk_df)
```

Activity begins at earlier intervals on a weekday compared to a weekend. The weekend has generally higher levels of activity throughout the day compared to weekdays. 