Load the data
=============

    library(knitr)
    library(dplyr)

    ## Warning: package 'dplyr' was built under R version 3.2.5

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(lubridate)

    ## Warning: package 'lubridate' was built under R version 3.2.5

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.2.5

    library(Hmisc)

    ## Warning: package 'Hmisc' was built under R version 3.2.5

    ## Loading required package: lattice

    ## Loading required package: survival

    ## Loading required package: Formula

    ## 
    ## Attaching package: 'Hmisc'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     combine, src, summarize

    ## The following objects are masked from 'package:base':
    ## 
    ##     format.pval, round.POSIXt, trunc.POSIXt, units

    knitr::opts_chunk$set(echo = TRUE)
    knitr::opts_chunk$set(cache = TRUE)

    if(!file.exists('activity.csv')){
        unzip('activity.zip')
    }
    aData <- read.csv('activity.csv')

What is mean total number of steps taken per day?
=================================================

1.  Calculate the total number of steps taken per day

<!-- -->

    steps.date <- aggregate(steps ~ date, data = aData, FUN = sum)

1.  Make a histogram of the total number of steps taken each day

<!-- -->

    steps.date <- aggregate(steps ~ date, data = aData, FUN = sum)
    barplot(steps.date$steps, names.arg = steps.date$date, xlab = "date", ylab = "steps")

![](PA_1_files/figure-markdown_strict/unnamed-chunk-3-1.png)

1.  Calculate and report the mean and median total number of steps taken
    per day

<!-- -->

    mean(steps.date$steps)

    ## [1] 10766.19

    median(steps.date$steps)

    ## [1] 10765

What is the average daily activity pattern?
===========================================

    averageStepsPerTimeBlock <- aggregate(x=list(meanSteps=aData$steps), by=list(interval=aData$interval), FUN=mean, na.rm=TRUE)

1.  Make a time series plot

<!-- -->

    steps.interval <- aggregate(steps ~ interval, data = aData, FUN = mean)
    plot(steps.interval, type = "l")

![](PA_1_files/figure-markdown_strict/unnamed-chunk-6-1.png)

1.  Which 5-minute interval, on average across all the days in the
    dataset, contains the maximum number of steps?

<!-- -->

    steps.interval$interval[which.max(steps.interval$steps)]

    ## [1] 835

Imputing missing values
=======================

1.  Calculate and report the total number of missing values in the
    dataset

<!-- -->

    sum(is.na(aData))

    ## [1] 2304

1.  Devise a strategy for filling in all of the missing values in
    the dataset.

<!-- -->

    "I will use the means for the 5-minute intervals as fillers for missing values"

    ## [1] "I will use the means for the 5-minute intervals as fillers for missing values"

1.  Create a new dataset that is equal to the original dataset but with
    the missing data filled in.

<!-- -->

    newData <- merge(aData, steps.interval, by = "interval", suffixes = c("", ".y"))
    nas <- is.na(newData$steps)
    newData$steps[nas] <- newData$steps.y[nas]
    newData <- newData[, c(1:3)]

1.  Make a histogram of the total number of steps taken each day and
    Calculate and report the mean and median total number of steps taken
    per day.

<!-- -->

    steps.date <- aggregate(steps ~ date, data = newData, FUN = sum)
    barplot(steps.date$steps, names.arg = steps.date$date, xlab = "date", ylab = "steps")

![](PA_1_files/figure-markdown_strict/unnamed-chunk-11-1.png)

    mean(steps.date$steps)

    ## [1] 10766.19

    median(steps.date$steps)

    ## [1] 10766.19

Are there differences in activity patterns between weekdays and weekends?
=========================================================================

1.  Create a new factor variable in the dataset with two levels --
    "weekday" and "weekend" indicating whether a given date is a weekday
    or weekend day.

<!-- -->

    activityDataImputed <- aData
    activityDataImputed$steps <- impute(aData$steps, fun=mean)
    stepsByDayImputed <- tapply(activityDataImputed$steps, activityDataImputed$date, sum)
    activityDataImputed$dateType <-  ifelse(as.POSIXlt(activityDataImputed$date)$wday %in% c(0,6), 'weekend', 'weekday')

1.  Make a panel plot containing a time series plot

<!-- -->

    averagedActivityDataImputed <- aggregate(steps ~ interval + dateType, data=activityDataImputed, mean)
    ggplot(averagedActivityDataImputed, aes(interval, steps)) + 
        geom_line() + 
        facet_grid(dateType ~ .) +
        xlab("5-minute interval") + 
        ylab("avarage number of steps")

![](PA_1_files/figure-markdown_strict/unnamed-chunk-13-1.png)

End
===
