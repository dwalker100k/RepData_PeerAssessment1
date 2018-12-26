Reproducible Research Course Project 1
======================================

This analysis will study the number of steps taken by an anonymous
individual, in five minute increments, during October-November 2012. The
first step is to download the data:

        if(!file.exists("./data")){dir.create("./data")}
        url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
        download.file(url, destfile = "./data/activity.zip")
        unzip("./data/activity.zip", exdir = "./data")

Now read the file into R:

        activity1 <- read.csv("activity.csv", header = TRUE)

Looking at the data using the function, we see that there are three
variables: steps, date, and interval. Steps and interval are integers
which seem fine, but date is listed as a factor. We change it to a date
variable with:

        activity1$date <- as.Date(activity1$date, format = "%Y-%m-%d")

What is the mean total number of steps taken per day?
-----------------------------------------------------

This portion of the assignment requires calculating the total number of
steps taken per day, plotting a histogram of total number of steps taken
per day and reporting the mean and median of steps taken per day.

Total number of steps taken per day and the histogram is produced by:

        daysteps <- with(activity1, aggregate(steps ~ date, FUN = sum, na.rm = TRUE))
        daystats <- data.frame(Mean = mean(daysteps$steps), Median = median(daysteps$steps))
        hist(daysteps$steps, main = "Total Daily Steps", xlab = "Steps Taken per Day", col = "blue")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

        print(paste0("The average daily steps is ",daystats$Mean))

    ## [1] "The average daily steps is 10766.1886792453"

        print(paste0("The median daily steps is ",daystats$Median))

    ## [1] "The median daily steps is 10765"

What is the average daily activity pattern?
-------------------------------------------

Here we are required to plot a time series of average steps taken during
the five minute intervals and report on which interval has the highest
average activity. Starting with finding the interval with the highest
average activity:

        intavgsteps <- with(activity1, aggregate(steps ~ interval, FUN = mean, na.rm = TRUE))
        maxint <- intavgsteps$interval[which.max(intavgsteps$steps)]
        print(paste0("The interval with the highest average activity is ",maxint))

    ## [1] "The interval with the highest average activity is 835"

The prior code shows that the highest activity is the 835 interval. For
the plot, the following code does the trick:

        with(intavgsteps, plot(interval, steps, type = "l", xlab = "Interval", ylab = "Average Steps per Interval", main = "Average Number of Steps per 5 Minute Interval"))
        abline(h = median(intavgsteps$steps), col = "red", lwd = 2)
        abline(v = maxint, col = "blue", lwd = 2)
        legend("topright", legend = c("Average Steps", "Median", "Max Interval = 835"), col = c("black", "red",     "blue"),
        lty = 1:1, lwd = c(1,2,2))

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-6-1.png) The
horizontal line is the median of the set of interval averages; the
vertical line represents the interval with the highest steps activity.

Imputing Missing Values
-----------------------

First, we have to calculate and report the number of missing values in
the dataset:

        missingsteps <- sum(is.na(activity1$steps))
        percMissStep <- mean(is.na(activity1$steps))
        print(paste0("The number of missing data points for steps is ", missingsteps))

    ## [1] "The number of missing data points for steps is 2304"

        print(paste0("The percentage of missing data is ", round(percMissStep*100, digits = 2)))

    ## [1] "The percentage of missing data is 13.11"

Second, we will create a second identical data set and then replace the
missing values with the overall average steps per interval

        fullactivity <- activity1
        fullactivity$steps[which(is.na(activity1$steps))] <- mean(activity1$steps, na.rm = TRUE)

This last line of code is worth explaining. Calling is.na on the steps
variable of the original data set creates a logical vector with TRUE for
an NA value and FALSE otherwise. Calling which on that logical vector
returns the index number where the values are TRUE. Since this index
vector is enclosed in brackets, it is subsetting the fullactivity
variable steps with the index numbers that are NA and assigning those
indexed positions the mean of the steps variable in the original data
set with the NA's removed.

And now we create the histogram for the full data:

        Fulldaysteps <- with(fullactivity, aggregate(steps ~ date, FUN = sum, na.rm = TRUE))
        hist(Fulldaysteps$steps, main = "Total Daily Steps", xlab = "Steps Taken per Day", col = "red")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-9-1.png)

        fulldaystats <- data.frame(Mean = mean(Fulldaysteps$steps), Median = median(Fulldaysteps$steps))
        print(paste0("The average daily steps is ",fulldaystats$Mean))

    ## [1] "The average daily steps is 10766.1886792453"

        print(paste0("The median daily steps is ",fulldaystats$Median))

    ## [1] "The median daily steps is 10766.1886792453"

As you can see, there is very little difference between the median and
mean of the original data set and the data set with the imputed values.
Since the mean was used as the replacing value, this is not surprising.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Create a new factor variable named weekend with two levels--weekend or
weekday and make a time series panel plot showing average steps per
interval vs interval between Weekends and Weekdays. First, Create the
factor variable weekend:

        fullactivity$day <- weekdays(fullactivity$date, abbreviate = FALSE)
        fullactivity$weekend<- ifelse(fullactivity$day == "Saturday"|fullactivity$day == "Sunday",
           "Weekend", "Weekday")

And now aggregate the steps by interval and weekend and make the panel
plot:

        daytypesteps <- with(fullactivity, aggregate(steps ~ interval+weekend, FUN = mean))
        library(lattice)
        xyplot(steps ~ interval|weekend, data = daytypesteps, type = "l", ylab = "Average Steps",xlab = "Interval", layout = c(1,2))

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-11-1.png)
The plot indicates that, although there is a spike in weekday activity
around 835, the activity level is generally lower during the weekday
versus the weekend.
