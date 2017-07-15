# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
The data to be proceesed is originally in file "activity.csv", which is assumed 
to be part of the current working directory. The data is read into a dataframe 
named "activity". For that, the following R code is used: 


```r
activity <- read.csv("activity.csv")
```

The following are some initial exploratory operations to better understand the data. 

### Dimensions of the data are: 

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
Determine the number of observations per day. 

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
gactivityDate <- group_by(activity, date)
dataPerDay <- dplyr::summarise(gactivityDate, count = n())
nobs <- unique(dataPerDay$count)
if(length(nobs) == 1) {
      print(paste("There are ", nobs[1], " observations for each day in the data set."))
      n = 24*60/5
      if(nobs[1]==n)
            print("This is exactly the number of 5-minute intervals in 24 hours.")
      else 
            print(paste("The number of 5-minute intervals in 24 hours is", n, "."))
} else {
      print("Observations per day differ. Some of those numbers are:")
      print(nobs)
}
```

```
## [1] "There are  288  observations for each day in the data set."
## [1] "This is exactly the number of 5-minute intervals in 24 hours."
```
This number correspo
2. A new column is added with values 1..7, idendifying the day of the week 
that each registered date corresponds to. The name of the column is _wdayNumber_
and is of type _integer_.

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
activity$wdayNumber <- day(activity$date)
```

## What is mean total number of steps taken per day?

```r
library(dplyr)
gactivityDate <- group_by(activity, date)
sactivityDate <- dplyr::summarise(gactivityDate, totalSteps = sum(steps, na.rm = TRUE))
hist(sactivityDate$totalSteps, breaks = 25, col = "gray", 
     main="Distribution of Total Steps Taken on Each Day", 
     ylab = "Frequency (Number of days)", xlab = "Total Number of Steps Per Day")
md <- round(median(sactivityDate$totalSteps), 2)
mn <- round(mean(sactivityDate$totalSteps), 2)
abline(v=md, col = "red", lwd=2)
abline(v=mn, col="blue", lwd=2)
#title(main="Total Number of Steps Per Day Recorded")
legend("topright", lty=1, lwd=2, legend=c(paste("median = ", md), 
                                     paste("mean = ", mn)), 
                                     col = c("red", "blue"))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

The following might alse be useful for this analysis. 

```r
library(dplyr)
gactivityDate <- group_by(activity, date)
sactivityDate <- dplyr::summarise(gactivityDate, totalSteps = sum(steps, na.rm = TRUE))
sactivityDate$n <- 1:nrow(sactivityDate)

barplot(sactivityDate$totalSteps, names.arg = sactivityDate$n, 
        xlab = "Day Recorded", ylab = "Total Number of Steps")
md <- round(median(sactivityDate$totalSteps), 2)
mn <- round(mean(sactivityDate$totalSteps), 2)
abline(h=md, col = "red")
abline(h=mn, col="blue")
title(main="Total Number of Steps Per Day Recorded")
legend("top", lty=1, lwd=2, legend=c(paste("median = ", md), 
                                     paste("mean = ", mn)), 
                                     col = c("red", "blue"))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->
... add beef describing what is shown...


## What is the average daily activity pattern?
To explore the average daily activity, we want to determine the average number 
of steps taken on each 5-minute interval. This can be done as follows. 

```r
library(dplyr)
gactivityInterval <- group_by(activity, interval)
sactivityInterval <- 
      dplyr::summarise(gactivityInterval, 
                       averageSteps = mean(steps, na.rm = TRUE))
sactivityInterval$intervalNumber <- 1:nrow(sactivityInterval)
plot(sactivityInterval$intervalNumber, sactivityInterval$averageSteps, type = "l",
     xlab = "5-minute Interval in a Day", ylab = "Average Number of Steps in Interval")
maxSteps <- max(sactivityInterval$averageSteps)
maxStepsPeriod <- subset(sactivityInterval, averageSteps == maxSteps)
print(paste("The 5-minute period with maximum average number of steps is: interval", 
            maxStepsPeriod$intervalNumber))
```

```
## [1] "The 5-minute period with maximum average number of steps is: interval 104"
```

```r
h <- maxStepsPeriod$interval
print(paste("It corresponds to the hour: ", 
            hm(paste(trunc(h/100), h%%100, sep = ":"))))
```

```
## [1] "It corresponds to the hour:  8H 35M 0S"
```

```r
md <- round(median(sactivityInterval$averageSteps), 2)
mn <- round(mean(sactivityInterval$averageSteps), 2)
abline(h=md, col = "red")
abline(h=mn, col="blue")
title(main="Average Number of Steps Per 5-minute Interval Across All Days")
legend("topright", lty=1, lwd=2, legend=c(paste("median = ", md), 
                                     paste("mean = ", mn)), 
                                     col = c("red", "blue"))
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->
... add beef describing what is shown...


## Imputing missing values

What I will do here is, to each 5-minute period with na value, assign the current
average number
of steps taken across all days in the data set in which the particular interval 
is not na. If a particular interval is na for all the days in the data set, then
it will be assigned a value of 0. .

## Are there differences in activity patterns between weekdays and weekends?
Now let's add a new column to the activity data frame. This is a column of type 
factor and two levels: weekend and weekday. It classifies each of the 
observations in data frame activity depending to the day of the week that its
date corresponds to. 

```r
library(lubridate)
weekendDays <- activity$wdayNumber == 1 | activity$wdayNumber == 7
activity$dayType <- "weekday"
activity[weekendDays,]$dayType = "weekend"
activity$dayType <- as.factor(activity$dayType)
```
And verifying how many of the observations belongs to each of the two categories, 
we get the following: 

```r
table(activity$dayType)
```

```
## 
## weekday weekend 
##   16416    1152
```

Now, let's plot to compare patterns in weekdays vs weekends. 


```r
#library(lattice)
gactivityDaytypeInterval <- group_by(activity, dayType, interval)
sactivityDaytypeInterval <- dplyr::summarise(gactivityDaytypeInterval, average=mean(steps, na.rm = TRUE))
sactivityDaytypeInterval$intervalNumber <- rep(1:(nrow(sactivityDaytypeInterval)/2), 2)
rangey <- range(sactivityDaytypeInterval$average)
#with(sactivityDaytypeInterval, xyplot(average~intervalNumber|dayType, type = "l"), #layout=c(1,2))
with(subset(sactivityDaytypeInterval, dayType == "weekday"), plot(intervalNumber, average, type = "l", ylim = rangey))
mnwd <- round(mean(subset(sactivityDaytypeInterval, dayType=="weekday")$average),2)
mdwd <- round(median(subset(sactivityDaytypeInterval, dayType=="weekday")$average),2)
abline(h = mdwd, col="red")
abline(h = mnwd, col="blue")
legend("toplef", lty=1, lwd=2, legend=c(paste("median = ", mdwd), 
                                     paste("mean = ", mnwd)), 
                                     col = c("red", "blue"))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


```r
with(subset(sactivityDaytypeInterval, dayType == "weekend"), plot(intervalNumber, average, type = "l", ylim = rangey))
mnwe <- round(mean(subset(sactivityDaytypeInterval, dayType=="weekend")$average), 2)
mdwe <- round(median(subset(sactivityDaytypeInterval, dayType=="weekend")$average), 2)
abline(h = mdwe, col="red")
abline(h = mnwe, col="blue")
legend("topleft", lty=1, lwd=2, legend=c(paste("median = ", mdwe), 
                                     paste("mean = ", mnwe)), 
                                     col = c("red", "blue"))
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

## The previous might not be correct... see readme from specifications...
## it should be something like the following: 

```r
library(lattice)
gactivityDaytypeInterval <- group_by(activity, dayType, interval)
sactivityDaytypeInterval <- dplyr::summarise(gactivityDaytypeInterval, average=mean(steps, na.rm = TRUE))
sactivityDaytypeInterval$intervalNumber <- rep(1:(nrow(sactivityDaytypeInterval)/2), 2)
#rangey <- range(sactivityDaytypeInterval$average)
with(sactivityDaytypeInterval, xyplot(average~intervalNumber|dayType, type = "l"), layout=c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->
