---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


# Activity Monitoring Data Analysis


## 0. Load the necessary library

```r
library(utils)
library(ggplot2)
```

```
## Warning: パッケージ 'ggplot2' はバージョン 4.3.2 の R の下で造られました
```
## 1. Loading and preprocessing the data

```r
# Read the CSV file directly from the zipped archive
zipFilePath <- "activity.zip"
csvFileName <- "activity.csv"
data <- read.csv(unzip(zipFilePath, files = csvFileName))

# Aggregate total steps by date
totalStepsByDay <- aggregate(steps ~ date, data, sum, na.rm = TRUE)
```

## 2. What is mean total number of steps taken per day?

```r
# Make a histogram of the total number of steps taken each day
hist(totalStepsByDay$steps, main = "Total Steps Taken Each Day", xlab = "Total Steps", ylab = "Frequency", col = "blue")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
# Calculate and report the mean and median total number of steps taken per day
meanSteps <- mean(totalStepsByDay$steps)
medianSteps <- median(totalStepsByDay$steps)

# Print the results
cat("Mean total number of steps taken per day: ", meanSteps, "\n")
```

```
## Mean total number of steps taken per day:  10766.19
```

```r
cat("Median total number of steps taken per day: ", medianSteps, "\n")
```

```
## Median total number of steps taken per day:  10765
```

## 3. What is the average daily activity pattern?

```r
# Calculate the average number of steps taken in each 5-minute interval
averageStepsByInterval <- aggregate(steps ~ interval, data, mean, na.rm = TRUE)

# Make a time series plot
plot(averageStepsByInterval$interval, averageStepsByInterval$steps, type = "l", xlab = "Interval", ylab = "Average Number of Steps", main = "Average Daily Activity Pattern")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
# Identify the interval with the maximum average number of steps
maxInterval <- averageStepsByInterval[which.max(averageStepsByInterval$steps), ]$interval

# Print the result
cat("The 5-minute interval with the maximum number of steps on average is:", maxInterval)
```

```
## The 5-minute interval with the maximum number of steps on average is: 835
```

## 4. Imputing missing values

```r
# Calculate and report the total number of missing values
totalNAs <- sum(is.na(data$steps))
cat("Total number of missing values: ", totalNAs, "\n")
```

```
## Total number of missing values:  2304
```

```r
# Calculate the mean for each interval
averageStepsByInterval <- aggregate(steps ~ interval, data, mean, na.rm = TRUE)

# Fill in missing values and create a new dataset
filledData <- data
for (i in 1:nrow(filledData)) {
  if (is.na(filledData$steps[i])) {
    filledData$steps[i] <- averageStepsByInterval$steps[which(averageStepsByInterval$interval == filledData$interval[i])]
  }
}

# Aggregate total steps by day for the filled dataset
totalStepsByDayFilled <- aggregate(steps ~ date, filledData, sum)

# Make a histogram
hist(totalStepsByDayFilled$steps, main = "Total Steps Taken Each Day (Filled Data)", xlab = "Total Steps", ylab = "Frequency", col = "green")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
# Calculate mean and median
meanStepsFilled <- mean(totalStepsByDayFilled$steps)
medianStepsFilled <- median(totalStepsByDayFilled$steps)

# Print the results
cat("Mean total number of steps taken per day (Filled Data): ", meanStepsFilled, "\n")
```

```
## Mean total number of steps taken per day (Filled Data):  10766.19
```

```r
cat("Median total number of steps taken per day (Filled Data): ", medianStepsFilled, "\n")
```

```
## Median total number of steps taken per day (Filled Data):  10766.19
```

## 5. Are there differences in activity patterns between weekdays and weekends?

```r
# Convert 'date' to Date type if it's not already
filledData$date <- as.Date(filledData$date)

# Create a new factor variable for weekday/weekend
filledData$dayType <- ifelse(weekdays(filledData$date, abbreviate = TRUE) %in% c("土", "日"), "weekend", "weekday")
filledData$dayType <- factor(filledData$dayType, levels = c("weekday", "weekend"))

# Aggregate average steps by interval and dayType again to reflect the updated dayType
averageStepsByDayType <- aggregate(steps ~ interval + dayType, filledData, mean)

# Create the plot
ggplot(averageStepsByDayType, aes(x = interval, y = steps, color = dayType)) +
  geom_line() +
  xlab("Interval") +
  ylab("Average Number of Steps") +
  ggtitle("Average Number of Steps by Interval: Weekday vs Weekend") +
  scale_color_manual(values = c("weekday" = "blue", "weekend" = "red")) +
  theme(legend.position = "bottom")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
