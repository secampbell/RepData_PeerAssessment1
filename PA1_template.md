### Scott Campbell

### 1-29-17

Introduction
------------

It's now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a Fitbit, Nike
Fuelband, or Jawbone Up. These type of devices are part of the
"quantified self" movement - a group of enthusiasts who take
measurements about themselves regularly to improve their health, to find
patterns in their behavior, or because they are tech geeks. But these
data remain under-utilized both because the raw data are hard to obtain
and there is a lack of statistical methods and software for processing
and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

Data
----

The data for this assignment can be downloaded from the course web site:

Dataset: Activity monitoring data \[52K\] The variables included in this
dataset are:

-   steps: Number of steps taking in a 5-minute interval (missing values
    are coded as NA)

-   date: The date on which the measurement data was taken in YYYY-MM-DD
    format

-   interval: Identifier for the 5-minute interval in which measurement
    was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

Loading the data
----------------

    data <- read.csv("activity.csv")

What is the mean total number of steps taken per day?
-----------------------------------------------------

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.3.2

    total.steps <- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)
    qplot(total.steps, binwidth=1000, xlab="total number of steps taken each day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

    mean(total.steps, na.rm=TRUE)

    ## [1] 9354.23

    median(total.steps, na.rm=TRUE)

    ## [1] 10395

What is the average daily activity pattern?
-------------------------------------------

    library(ggplot2)
    averages <- aggregate(x=list(steps=data$steps), by=list(interval=data$interval),
                          FUN=mean, na.rm=TRUE)
    ggplot(data=averages, aes(x=interval, y=steps)) +
        geom_line() +
        xlab("5-minute interval") +
        ylab("Average number of steps taken")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

The 5-minute interval that contains the maximum number of steps-

    averages[which.max(averages$steps),]

    ##     interval    steps
    ## 104      835 206.1698

Imputing the missing values
---------------------------

We need to remove or nullify the missing values using an average over
time.fs

    missing <- is.na(data$steps)

    # Number of missing values
    table(missing)

    ## missing
    ## FALSE  TRUE 
    ## 15264  2304

The missing values are filled in using a mean for that missing value's
interval-

    # Replacing each missing value with a mean
    fill.value <- function(steps, interval) {
        filled <- NA
        if (!is.na(steps))
            filled <- c(steps)
        else
            filled <- (averages[averages$interval==interval, "steps"])
        return(filled)
    }
    filled.data <- data
    filled.data$steps <- mapply(fill.value, filled.data$steps, filled.data$interval)

Now that the missing values are filled in, making a histogram of the
total number of steps each day-

    total.steps <- tapply(filled.data$steps, filled.data$date, FUN=sum)
    qplot(total.steps, binwidth=1000, xlab="total number of steps taken each day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

    mean(total.steps)

    ## [1] 10766.19

    median(total.steps)

    ## [1] 10766.19

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Assigning a marker (weekday or weekend) to each measurement in the data-
