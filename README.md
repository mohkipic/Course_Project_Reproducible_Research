# Course_Project_Reproducible_Research
## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These
type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find
patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of
statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists
of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute 
intervals each day.

## Reading the dataset and processing the data
```
library("data.table")
library(ggplot2)
activityDT <- data.table::fread(input = "activity.csv")
```

## Histogram of the total number of steps taken each day
```
Total_Steps <- activityDT[, c(lapply(.SD, sum, na.rm = TRUE)), .SDcols = c("steps"), by = .(date)] 
head(Total_Steps, 10)
library(ggplot2)
png("hist1.png", width=480, height=480)
ggplot(Total_Steps, aes(x = steps)) + geom_histogram(fill = "blue", binwidth = 1000) + labs(title = "Daily Steps", 
       x = "Steps", y = "Frequency")
dev.off()
```

## Time series plot of the average number of steps taken
```
Total_Steps[, .(Mean_Steps = mean(steps), Median_Steps = median(steps))]
```

## The 5-minute interval that, on average, contains the maximum number of steps
```
IntervalDT <- activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval)] 
ggplot(IntervalDT, aes(x = interval , y = steps)) + geom_line(color="blue", size=1) + labs(title = "Avg. Daily Steps",
       x = "Interval", y = "Avg. Steps per day")
IntervalDT <- activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval)] 
IntervalDT[steps == max(steps), .(max_interval = interval)]
```

## describe and show a strategy for imputing missing data
```
activityDT[is.na(steps), .N ]
activityDT[is.na(steps), "steps"] <- round(activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps")])
data.table::fwrite(x = activityDT, file = "tidyData.csv", quote = FALSE)
```

## Histogram of the total number of steps taken each day after missing values are imputed
```
Total_Steps <- activityDT[, c(lapply(.SD, sum, na.rm = TRUE)), .SDcols = c("steps"), by = .(date)]
Total_Steps[, .(Mean_Steps = mean(steps), Median_Steps = median(steps))]
library(ggplot2)
ggplot(Total_Steps, aes(x = steps)) +  geom_histogram(fill = "blue", binwidth = 1000) + labs(title = "Daily Steps", 
       x = "Steps", y = "Frequency")
```


## Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
```
activityDT <- data.table::fread(input = "activity.csv")
activityDT[, date := as.POSIXct(date, format = "%Y-%m-%d")]
activityDT[, `Day of Week`:= weekdays(x = date)]
activityDT[grepl(pattern = "Monday|Tuesday|Wednesday|Thursday|Friday", x = `Day of Week`), "weekday or weekend"] <- "weekday"
activityDT[grepl(pattern = "Saturday|Sunday", x = `Day of Week`), "weekday or weekend"] <- "weekend"
activityDT[, `weekday or weekend` := as.factor(`weekday or weekend`)]
head(activityDT, 10)
```
```
activityDT[is.na(steps), "steps"] <- activityDT[, c(lapply(.SD, median, na.rm = TRUE)), .SDcols = c("steps")]
IntervalDT <- activityDT[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval, `weekday or weekend`)] 
ggplot(IntervalDT , aes(x = interval , y = steps, color=`weekday or weekend`)) + geom_line() +
       labs(title = "Avg. Daily Steps by Weektype", x = "Interval", y = "No. of Steps") + facet_wrap(~`weekday or weekend` ,
       ncol = 1, nrow=2)
```
