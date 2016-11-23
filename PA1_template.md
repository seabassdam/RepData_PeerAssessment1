## Introduction

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a
[Fitbit](http://www.fitbit.com), [Nike
Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or
[Jawbone Up](https://jawbone.com/up). These type of devices are part of
the "quantified self" movement -- a group of enthusiasts who take
measurements about themselves regularly to improve their health, to
find patterns in their behavior, or because they are tech geeks. But
these data remain under-utilized both because the raw data are hard to
obtain and there is a lack of statistical methods and software for
processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012
and include the number of steps taken in 5 minute intervals each day.

## Data

The data for this assignment can be downloaded from the course web
site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken




The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this
dataset.


## Assignment

This assignment will be described in multiple parts. You will need to
write a report that answers the questions detailed below. Ultimately,
you will need to complete the entire assignment in a **single R
markdown** document that can be processed by **knitr** and be
transformed into an HTML file.

Throughout your report make sure you always include the code that you
used to generate the output you present. When writing code chunks in
the R markdown document, always use `echo = TRUE` so that someone else
will be able to read the code. **This assignment will be evaluated via
peer assessment so it is essential that your peer evaluators be able
to review the code for your analysis**.

For the plotting aspects of this assignment, feel free to use any
plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the [GitHub repository created for this
assignment](http://github.com/rdpeng/RepData_PeerAssessment1). You
will submit this assignment by pushing your completed files into your
forked repository on GitHub. The assignment submission will consist of
the URL to your GitHub repository and the SHA-1 commit ID for your
repository state.

NOTE: The GitHub repository also contains the dataset for the
assignment so you do not have to download the data separately.



### Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. `read.csv()`)

```r
# Check if file is unzipped
filename <- "activity.zip"

if (!file.exists("activity.csv")) { 
        unzip(filename) 
}
```

2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
#Load raw data
data.raw <- read.csv("activity.csv",stringsAsFactors = FALSE)

#Transform to date format
data.raw$date <- as.POSIXct(data.raw$date, format="%Y-%m-%d")

#compute weekend or workday
data.raw <- data.frame(date=data.raw$date, 
                       daytype=(ifelse(
                               tolower(weekdays(data.raw$date)) == "saturday" | 
                               tolower(weekdays(data.raw$date)) == "sunday",
                               "weekend", 
                               "weekday")), 
                       steps=data.raw$steps, 
                       interval=data.raw$interval)
```

### What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in
the dataset.

1. Make a histogram of the total number of steps taken each day

```r
#Aggregate number of steps by day  
steps <- aggregate(data.raw$steps, by=list(data.raw$date),FUN = sum, na.rm=TRUE)

#Historgram of total steps each day
hist(steps$x, 
     breaks=seq(from=0, to=25000, by=2500),
     xlab="Total number of steps", 
     ylim=c(0, 20), 
     main="Histogram of the total number of steps taken each day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)

2. Calculate and report the **mean** and **median** total number of steps taken per day


```r
#Calculations of mean and median
mean(steps$x)
```

```
## [1] 9354.23
```

```r
median(steps$x)
```

```
## [1] 10395
```
The mean is 9354.2295082 and the median is 10395.  

### What is the average daily activity pattern?

1. Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)  

```r
#Compute the mean steps across all days for each interval
data.mean <- aggregate(data.raw$steps, by=list(data.raw$interval), FUN = mean, na.rm=TRUE)
plot(data.mean$Group.1, 
     data.mean$x, 
     type="l", 
     xlab="Interval", 
     ylab="Average number of steps", 
     main="Time-series of the average number of steps per intervals")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?  

```r
#Find the position with the max interval  
max <- which(data.mean$x == max(data.mean$x))  
#Find which 5-minute interval 
max_int <-data.mean[max,1]
```
The 5-minute interval that contains the maximum steps, across all days, is: 835.  

### Imputing missing values

Note that there are a number of days/intervals where there are missing
values (coded as `NA`). The presence of missing days may introduce
bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s)  

```r
#Finding the number of missing values  
na <- sum(is.na(data.raw$steps))
```
The number of missing values is 2304.  

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.  

```r
#Find NA positions  
na.pos <- which(is.na(data.raw$steps))
#Vector of means
na.mean <- rep(mean(data.raw$steps,na.rm=TRUE), times=length(na.pos))
```
Replace all NAs with the mean of the Steps.  

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
#Replace all NAs with the mean 
data.new <- data.raw
data.new[na.pos,"steps"] <- na.mean
```
4. Make a histogram of the total number of steps taken each day and Calculate and report the **mean** and **median** total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?  

```r
#Histogram with NAs replaced
#Aggregate number of steps by day  
steps.new <- aggregate(data.new$steps, by=list(data.new$date),FUN = sum, na.rm=TRUE)

#Historgram of total steps each day
hist(steps.new$x, 
     breaks=seq(from=0, to=25000, by=2500),
     xlab="Total number of steps", 
     ylim=c(0, 35), 
     main="Histogram of the total number of steps taken each day")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)
  
The mean and median are computed using: 

```r
#Calculations of mean and median
mean(steps.new$x)
```

```
## [1] 10766.19
```

```r
median(steps.new$x)
```

```
## [1] 10766.19
```
The mean is 1.0766189 &times; 10<sup>4</sup> and the median is 1.0766189 &times; 10<sup>4</sup>.  
  
The values differ from the estimates from the first part of the assignment.  The replacement of the missing data with the mean increased the data values.  Therefore the histogram had a higher frequency and the mean/median are larger.  

### Are there differences in activity patterns between weekdays and weekends?

For this part the `weekdays()` function may be of some help here. Use
the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.  
*"daytype" already processed*

1. Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).  



```r
#Load lattice
library(lattice)

#Compute average steps across daytype variable
week <- aggregate(data.raw$steps, by=list(data.raw$daytype,data.raw$interval),FUN=mean, na.rm=TRUE)
names(week) <- c("daytype","interval", "mean")

#Compute time series plot
xyplot(mean ~ interval | daytype, week, 
       type="l", 
       lwd=1, 
       xlab="Interval", 
       ylab="Number of steps", 
       layout=c(1,2))
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)


## Submitting the Assignment

To submit the assignment:

1. Commit your completed `PA1_template.Rmd` file to the `master` branch of your git repository (you should already be on the `master` branch unless you created new ones)

2. Commit your `PA1_template.md` and `PA1_template.html` files produced by processing your R markdown file with the `knit2html()` function in R (from the **knitr** package)

3. If your document has figures included (it should) then they should have been placed in the `figure/` directory by default (unless you overrode the default). Add and commit the `figure/` directory to your git repository.

4. Push your `master` branch to GitHub.

5. Submit the URL to your GitHub repository for this assignment on the course web site.

In addition to submitting the URL for your GitHub repository, you will
need to submit the 40 character SHA-1 hash (as string of numbers from
0-9 and letters from a-f) that identifies the repository commit that
contains the version of the files you want to submit. You can do this
in GitHub by doing the following:

1. Go into your GitHub repository web page for this assignment

2. Click on the "?? commits" link where ?? is the number of commits you have in the repository. For example, if you made a total of 10 commits to this repository, the link should say "10 commits".

3. You will see a list of commits that you have made to this repository. The most recent commit is at the very top. If this represents the version of the files you want to submit, then just click the "copy to clipboard" button on the right hand side that should appear when you hover over the SHA-1 hash. Paste this SHA-1 hash into the course web site when you submit your assignment. If you don't want to use the most recent commit, then go down and find the commit you want and copy the SHA-1 hash.

A valid submission will look something like (this is just an **example**!)

```r
https://github.com/rdpeng/RepData_PeerAssessment1

7c376cc5447f11537f8740af8e07d6facc3d9645
```
