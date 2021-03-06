---
title: "RepData_PeerAssessment1"
author: "Amber Zhang"
date: "2018年10月12日"
output: 
html_document:
        keep_md: true
---


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)

```


## Download and read in activitiy data
First we need to download the target data into the R work directory, then unzip
and read in.


```{r}
unzip(zipfile = "repdata%2Fdata%2Factivity.zip")
activityData <- read.csv(file = "activity.csv")
```


## Calculate the total number of steps taken per day
First we need to calculate the total number of steps taken per day.


```{r}
library(dplyr)
totalPerday <- activityData %>%
        select(steps, date) %>%
        group_by(date) %>%
        summarise(steps = sum(steps, na.rm = TRUE))
head(totalPerday)
```


## Make a histogram of the total number of steps taken each day
The plot is made based on ggplot2 system.


```{r}
library(ggplot2)
ggplot(totalPerday, aes(x = steps)) +
        geom_histogram(fill = "purple", binwidth = 1000) +
        labs(x = "Steps", y = "Frequency",
             title = "Daily Steps")

```


## Calculate the mean and median of the total number of steps taken per day


```{r}
mean_step <- mean(totalPerday$steps, na.rm = TRUE)
median_step <- median(totalPerday$steps, na.rm = TRUE)
mean_step
median_step
```


## The average daily activity pattern
1. Make a time series plot of the 5-minute interval (x-axis) and the average 
number of steps taken, averaged across all days (y-axis)


```{r}
avgpattern <- activityData %>%
        select(steps, interval) %>%
        group_by(interval) %>%
        summarise(avgsteps = mean(steps, na.rm = TRUE))
ggplot(avgpattern, aes(x = interval, y = avgsteps)) +
        geom_line(color = "purple", size = 1) + 
        labs(x = "Interval", y = "Avg. steps per day",
             title = "Avg. daily steps")
```


## The maximum number of steps
2.Which 5-minute interval, on average across all the days in the dataset, 
contains the maximum number of steps?


```{r}
avgpattern %>%
        slice(which.max(avgsteps))
```


## Imputing missing values
1.Calculate and report the total number of missing values in the dataset


```{r}
totalNA <- sum(is.na(activityData))
totalNA
```


## New dataset with missing data filled in 
Create a new dataset that is equal to the original dataset but with the missing
data filled in.


```{r}
filterNA <- activityData
filterNA[is.na(filterNA)] <- 0

```


## Make a new  histogram  
Make a histogram of the total number of steps taken each day and Calculate and 
report the mean and median total number of steps taken per day. The values are 
exactly the same as those of the fisrt step, because I set the is.NA arguement
to TRUE.


```{r}
totalPerday2 <- filterNA %>%
        select(steps, date) %>%
        group_by(date) %>%
        summarise(steps = sum(steps, na.rm = TRUE))
head(totalPerday2)
```


```{r}
ggplot(totalPerday2, aes(x = steps)) +
        geom_histogram(fill = "purple", binwidth = 1000) +
        labs(x = "Steps", y = "Frequency",
             title = "Daily Steps")
```


## Are there differences in activity patterns between weekdays and weekends?


```{r}
library(data.table)
activityData <- fread("activity.csv")
activityData[,date := as.POSIXct(date, format = "%Y-%m-%d")]
activityData[,  `Day of Week`:= weekdays(x = date)]
activityData[grepl(pattern = "星期一|星期二|星期三|星期四|星期五", 
                   x = `Day of Week`), "weekday or weekend"] <- "weekday"
activityData[grepl(pattern = "星期六|星期日", x = `Day of Week`), 
             "weekday or weekend"] <- "weekend"
activityData[,`weekday or weekend` := as.factor(`weekday or weekend`)]
head(activityData)
```


```{r}
activityData[is.na(steps), "steps"] <- activityData[, c(lapply(.SD, median, na.rm = TRUE)), .SDcols = c("steps")]
intervaldata <- activityData[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval, `weekday or weekend`)] 
ggplot(intervaldata, aes(x = interval, y = steps, 
                         color = `weekday or weekend`)) +
        geom_line() +
        labs(title = "Avg. Daily Steps by Weektype", x = "Interval", 
             y = "No. of Steps") +
        facet_wrap(~`weekday or weekend` , ncol = 1, nrow=2)
        
```



