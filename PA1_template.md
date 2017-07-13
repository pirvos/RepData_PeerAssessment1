# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
The data to be proceesed is originally in file "activity.csv", which is assumed to be part of the current working directory. The data is read into a dataframe named "activity". For that, the following R code is used: 


```r
activity <- read.csv("activity.csv")
```

The following are some initial exploratory operations to better understand the data. 
1. Dimensions of the data are: 

```r
dimsactivity <- dim(activity)
print(paste("Number of rows =", dimsactivity[1]))
```

```
## [1] "Number of rows = 17568"
```

```r
print(paste("Number of columns =", dimsactivity[2]))
```

```
## [1] "Number of columns = 3"
```
## What is mean total number of steps taken per day?



## What is the average daily activity pattern?



## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
Now let's add a new column to the activity data frame. This is a column of type 
factor and two levels: weekend and weekday. It classifies each of the 
observations in data frame activity depending to the day of the week that its
date corresponds to. 

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```r
weekendDays <- day(activity$date) == 1 | day(activity$date) == 7
activity$dayType <- "weekday"
activity[weekendDays,]$dayType = "weekend"
activity$dayType <- as.factor(activity$dayType)
```
And verifying how many of the observations belongs to each of the two categories, 
we get the following: 

```
## 
## weekday weekend 
##   16416    1152
```
