Introduction
------------

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a Fitbit, Nike
Fuelband, or Jawbone Up. These type of devices are part of the
“quantified self” movement – a group of enthusiasts who take
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

The data for this assignment was downloaded from the course web site:

-   Dataset: [Activity monitoring
    data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)
    \[52K\]

The variables included in this dataset are:

-   steps: Number of steps taking in a 5-minute interval (missing values
    are coded as `NA`)

-   date: The date on which the measurement was taken in YYYY-MM-DD
    format

-   interval: Identifier for the 5-minute interval in which measurement
    was taken

Loading and preprocessing the data
----------------------------------

Load and preprocessing data. Ignore missing values.

    setwd("~/Desktop/Reproducible-Research")
    dat <- read.csv("activity.csv",header = TRUE)
    dat$date <- as.Date(dat$date) 
    dat.ig.na <- na.omit(dat) 

What is mean total number of steps taken per day?
-------------------------------------------------

Sum steps by day, create Histogram, and calculate mean and median.

    dailysteps <- rowsum(dat.ig.na$steps, format(dat.ig.na$dat, '%Y-%m-%d')) 
    dailysteps <- data.frame(dailysteps) 
    names(dailysteps) <- ("steps") 

    hist(dailysteps$steps, main = "Total number of steps taken per day ", xlab="Number of Steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

    rmean <- mean(dailysteps$steps)
    rmedian <- median(dailysteps$steps)

The `mean` total number of steps taken per day is 1.076618910^{4} and
the `median` is 10765.

What is the average daily activity pattern?
-------------------------------------------

    steps_by_interval <- aggregate(steps ~ interval, dat, mean)

    plot(steps_by_interval$interval,steps_by_interval$steps, type="l", xlab="Interval", ylab="Number of Steps",main="Average Number of Steps per Day by Interval")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

    max_interval <- steps_by_interval[which.max(steps_by_interval$steps),1]

The 5-minute interval, on average across all the days in the data set,
containing the maximum number of steps is 835.

Impute missing values. Compare imputed to non-imputed data.
-----------------------------------------------------------

Note that there are a number of days/intervals where there are missing
values (coded as 𝙽𝙰). The presence of missing days may introduce bias
into some calculations or summaries of the data.

Our statage is replace the missing value by inserting the average for
each interval.

    incomplete <- sum(!complete.cases(dat))
    imputed_data <- transform(dat, steps = ifelse(is.na(dat$steps), steps_by_interval$steps[match(dat$interval, steps_by_interval$interval)], dat$steps))

    imputed_data[as.character(imputed_data$date) == "2012-10-01", 1] <- 0

Recount total steps by day and create Histogram.

    new_steps_by_day <- aggregate(steps ~ date, imputed_data, sum)
    hist(new_steps_by_day$steps,main = "Total number of steps taken per day", col="blue", xlab="Number of Steps")

    #Create Histogram to show difference. 
    hist(dailysteps$steps, main = "Total number of steps taken per day", col="red", xlab="Number of Steps", add=T)
    legend("topright", c("Imputed", "Non-imputed"), col=c("blue", "red"), lwd=10)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

Calculate new mean and median for imputed data.

    rmean.i <- mean(new_steps_by_day$steps, na.rm=TRUE)
    rmedian.i <- median(new_steps_by_day$steps, na.rm=TRUE)

Calculate difference between imputed and non-imputed data.

    mean_diff <- rmean.i - rmean
    med_diff <- rmedian.i - rmedian

Calculate total difference.

    total_diff <- sum(new_steps_by_day$steps) - sum(dailysteps$steps)

-   The imputed data mean is 1.058969410^{4}
-   The imputed data median is 1.076618910^{4}
-   The difference between the non-imputed mean and imputed mean is
    -176.4948964
-   The difference between the non-imputed mean and imputed mean is
    1.1886792
-   The difference between total number of steps between imputed and
    non-imputed data is 7.536332110^{4}. Thus, there were
    7.536332110^{4} more steps in the imputed data.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Created a plot to compare and contrast number of steps between the week
and weekend. There is a higher peak earlier on weekdays, and more
overall activity on weekends.

    weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", 
                  "Friday")
    imputed_data$dow = as.factor(ifelse(is.element(weekdays(as.Date(imputed_data$date)),weekdays), "Weekday", "Weekend"))

    steps_by_interval_i <- aggregate(steps ~ interval + dow, imputed_data, mean)

    library(lattice)

    xyplot(steps_by_interval_i$steps ~ steps_by_interval_i$interval|steps_by_interval_i$dow, main="Average Steps per Day by Interval",xlab="Interval", ylab="Steps",layout=c(1,2), type="l")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)
