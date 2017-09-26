---
title: "Bad weather damages on health and economy in the USA"
author: "Alessandro Puiatti"
date: "9/22/2017"
output: html_document
---
[1]: https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf  "Storm Data Documentation"


This documet summarizes the data analysis performed over the NOAA Storm Database 
in order to understand which types of events are most harmful with respect to the
population health and whcih have the greatest economic consequences. 
A detailed descritption of the event types and how the harmful and economic
damage are computed can be found at the National Weather Service 
[Storm Data Documentation][1]



## Data Processing
The amount of data reach almost 0.5GB, so it would be better not to load in memory 
the entire table but only the interested ones. For undertsanding which ones they are
we first read 10 lines (the header an some data), just to have an idea of the data
types and to recognize which are the cols that we have to load. In order to
perform this operation we need to load two libraries: *data.table* and *R.utils*.
For further computation other libraries are needed that are: *plyr* and *dplyr*.


```r
library(data.table)
library(R.utils,warn.conflicts = FALSE, quietly = TRUE, verbose = FALSE)
library(plyr) # for useful tools
library(dplyr, warn.conflicts = FALSE, quietly = TRUE, verbose = FALSE)
```

Therefore the first opeartions are: unzip the data file (already downloaded), read 
the first 10 lines on a variable (*firstLines*), and then look at them.


```r
bunzip2("repdata-data-StormData.csv.bz2", remove = F)
```

```
## Error in decompressFile.default(filename = filename, ..., ext = ext, FUN = FUN): File already exists: repdata-data-StormData.csv
```

```r
firstLines <- read.csv("repdata-data-StormData.csv", nrows = 10)
firstLines
```

```
##    STATE__           BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1        1  4/18/1950 0:00:00      130       CST     97     MOBILE    AL
## 2        1  4/18/1950 0:00:00      145       CST      3    BALDWIN    AL
## 3        1  2/20/1951 0:00:00     1600       CST     57    FAYETTE    AL
## 4        1   6/8/1951 0:00:00      900       CST     89    MADISON    AL
## 5        1 11/15/1951 0:00:00     1500       CST     43    CULLMAN    AL
## 6        1 11/15/1951 0:00:00     2000       CST     77 LAUDERDALE    AL
## 7        1 11/16/1951 0:00:00      100       CST      9     BLOUNT    AL
## 8        1  1/22/1952 0:00:00      900       CST    123 TALLAPOOSA    AL
## 9        1  2/13/1952 0:00:00     2000       CST    125 TUSCALOOSA    AL
## 10       1  2/13/1952 0:00:00     2000       CST     57    FAYETTE    AL
##     EVTYPE BGN_RANGE BGN_AZI BGN_LOCATI END_DATE END_TIME COUNTY_END
## 1  TORNADO         0      NA         NA       NA       NA          0
## 2  TORNADO         0      NA         NA       NA       NA          0
## 3  TORNADO         0      NA         NA       NA       NA          0
## 4  TORNADO         0      NA         NA       NA       NA          0
## 5  TORNADO         0      NA         NA       NA       NA          0
## 6  TORNADO         0      NA         NA       NA       NA          0
## 7  TORNADO         0      NA         NA       NA       NA          0
## 8  TORNADO         0      NA         NA       NA       NA          0
## 9  TORNADO         0      NA         NA       NA       NA          0
## 10 TORNADO         0      NA         NA       NA       NA          0
##    COUNTYENDN END_RANGE END_AZI END_LOCATI LENGTH WIDTH F MAG FATALITIES
## 1          NA         0      NA         NA   14.0   100 3   0          0
## 2          NA         0      NA         NA    2.0   150 2   0          0
## 3          NA         0      NA         NA    0.1   123 2   0          0
## 4          NA         0      NA         NA    0.0   100 2   0          0
## 5          NA         0      NA         NA    0.0   150 2   0          0
## 6          NA         0      NA         NA    1.5   177 2   0          0
## 7          NA         0      NA         NA    1.5    33 2   0          0
## 8          NA         0      NA         NA    0.0    33 1   0          0
## 9          NA         0      NA         NA    3.3   100 3   0          1
## 10         NA         0      NA         NA    2.3   100 3   0          0
##    INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP WFO STATEOFFIC ZONENAMES
## 1        15    25.0          K       0         NA  NA         NA        NA
## 2         0     2.5          K       0         NA  NA         NA        NA
## 3         2    25.0          K       0         NA  NA         NA        NA
## 4         2     2.5          K       0         NA  NA         NA        NA
## 5         2     2.5          K       0         NA  NA         NA        NA
## 6         6     2.5          K       0         NA  NA         NA        NA
## 7         1     2.5          K       0         NA  NA         NA        NA
## 8         0     2.5          K       0         NA  NA         NA        NA
## 9        14    25.0          K       0         NA  NA         NA        NA
## 10        0    25.0          K       0         NA  NA         NA        NA
##    LATITUDE LONGITUDE LATITUDE_E LONGITUDE_ REMARKS REFNUM
## 1      3040      8812       3051       8806      NA      1
## 2      3042      8755          0          0      NA      2
## 3      3340      8742          0          0      NA      3
## 4      3458      8626          0          0      NA      4
## 5      3412      8642          0          0      NA      5
## 6      3450      8748          0          0      NA      6
## 7      3405      8631          0          0      NA      7
## 8      3255      8558          0          0      NA      8
## 9      3334      8740       3336       8738      NA      9
## 10     3336      8738       3337       8737      NA     10
```

As can be seen from *firstLines* the cols that are of our interest, checking also
their meanings in the [Storm Data Documentation][1], are:

- EVTYPE
- FATALITIES
- INJURIES
- PROPDMG
- PROPDMGEXP
- CROPDMG
- CROPDMGEXP

So the next step si to extract from the database only these 7 cols


```r
cols <- c("EVTYPE", "FATALITIES", "INJURIES", "PROPDMG", "PROPDMGEXP", 
          "CROPDMG", "CROPDMGEXP")
data <- fread("repdata-data-StormData.csv", select = cols, showProgress = F)
```

Now, just to have an idea of the memory allocated for *data*


```r
sprintf("Size of only seven columns in memory: %s MB", floor(utils::object.size(data)/1000000))
```

```
## [1] "Size of only seven columns in memory: 50 MB"
```
that is almost one-tenth of the total memory needed for the entire dataset. 
Now, we check for NAs values

```r
mean(is.na(data))
```

```
## [1] 0
```
Given that there are no NAs we don't need to bother anymore about it in the next steps

The last operations needed for computing the summaries required are then:

1. reorganize the data grouped by *EVTYPE* for answering the question related to the population 
health (*dataHealth*), after havin selcted only the related cols from *data*

2. reorganize the data grouped by *EVTYPE*, for answering the question related to the most 
damageful event (*dataDmg*), after havin selcted only the related cols from *data*


#### Population health

Select the related cols and remove all rows with null values


```r
cols <- c("EVTYPE", "FATALITIES", "INJURIES")
dataHealth <- select(data, cols) %>%
        filter(FATALITIES != 0 & INJURIES != 0)
```

Group the data by *EVTYPE*, summarize, compute the percentages for *FATALITIES* 
and *INJURIES* and finally reordered the data in descending order


```r
sumHealth <- as.data.frame(group_by(dataHealth, EVTYPE) %>%
        summarize(sumFatal = sum(FATALITIES), sumInjury = sum(INJURIES )) %>%
        mutate(EVTYPE, sumFatal, percFatal = round(100*sumFatal/sum(sumFatal),2), 
               sumInjury, percInj = round(100*sumInjury/sum(sumInjury),2)))

sumFatalities <- data.frame(sumHealth$EVTYPE, sumHealth$sumFatal, sumHealth$percFatal)
names(sumFatalities) <- c("EVTYPE", "Fatal", "percFatal")
sumFatalities <- setorder(sumFatalities, -Fatal)

sumInj <- data.frame(sumHealth$EVTYPE,sumHealth$sumInjury,sumHealth$percInj)
names(sumInj) <- c("EVTYPE", "Injury", "percInj")
sumInj <- setorder(sumInj, -Injury)
```

#### Damges

Select the related cols and remove all rows with null values


```r
cols <- c("EVTYPE", "PROPDMG", "PROPDMGEXP", "CROPDMG", "CROPDMGEXP")
dataDmg <- select(data, cols) %>%
        filter(PROPDMG != 0 & CROPDMG != 0)
```

The two cols *PROPDMGEXP* and *CROPDMGEXP* have to be converted in integers with the values
equivalent to the amount of expressed unit (look at [Storm Data Documentation][1]):

- 10^3^ for *K* 
- 10^6^ for *M*
- 10^9^ for *B* 

However, looking at the chars presented in the two mention cols, we have other values than
*K*, *M*, and *B*:

```r
unique(dataDmg$PROPDMGEXP)
```

```
## [1] "B" "M" "m" "K" "5" "0" ""  "3"
```

```r
unique(dataDmg$CROPDMGEXP)
```

```
## [1] "M" "K" "m" "k" "B" "0" ""
```
We can assume that the number "5", "3" and "0" are the power of 10, "m" is equal to "M"
and "k" is equal to "K". The "" element would be considered as NA


```r
dataDmg$PROPDMGEXP <- as.numeric(revalue(dataDmg$PROPDMGEXP, c("K"=10^3, "M"=10^6, "m"=10^6,
                                                    "B"=10^9, "5"=10^5, "0"=1, "3"=10^3)))
dataDmg$CROPDMGEXP <- as.numeric(revalue(dataDmg$CROPDMGEXP, c("K"=10^3, "k"=10^3, "M"=10^6, "m"=10^6,
                                                    "B"=10^9, "0"=1)))
```

Then we can create a new table with the *PROPDMG* and *CROPDMG* computed in dollars


```r
dataDmg <- transmute(dataDmg, EVTYPE, PROPDMG = PROPDMG*PROPDMGEXP, CROPDMG = CROPDMG*CROPDMGEXP)
```

Group the data by *EVTYPE*, summarize and compute also the percentages for *CROPDMG*
and *PROPDMG*


```r
sumDmg <- group_by(dataDmg, EVTYPE) %>%
       summarize( sumCrop = sum(CROPDMG, na.rm = T), sumProp = sum(PROPDMG,na.rm = T)) %>%
        mutate(EVTYPE, sumProp, percProp = round(100*sumProp/sum(sumProp),2),
               sumCrop, percCrop = round(100*sumCrop/sum(sumCrop),2))

sumCrop <- data.frame(sumDmg$EVTYPE, sumDmg$sumCrop, sumDmg$percCrop)
names(sumCrop) <- c("EVTYPE", "Crop", "percCrop")
sumCrop <- setorder(sumCrop, -Crop)

sumProp <- data.frame(sumDmg$EVTYPE,sumDmg$sumProp,sumDmg$percProp)
names(sumProp) <- c("EVTYPE", "Prop", "percProp")
sumProp <- setorder(sumProp, -Prop)
```

Remove unnecessary data


```r
rm(list = c("data","dataDmg","dataHealth","firstLines","cols"))
```



## Results
Having done all the preparatory data processing what is left now is to look at the results following 
the two main questions.


#### Across the United States, which types of events are most harmful with respect to population health?

For answering this question we have to look in the *sumHealth* datase and find which event is more 
harmful considering separately the *FATALITIES* and the *INJURIES*. Looking at the *sumHealth* data
it turns out, and it makes sense, that the *Tornado* is the most harmful for both cases. However, the same progression does not follow equally for the other events, as reported in the following table for the first
5 calamities more harmful that account for the 82.85%, in the case of *FATALITIES*, and the 89.63 for the 
*INJURIES*

```r
library(gridExtra)
temp <- format(data.frame(sumFatalities[1:5,1], sumFatalities[1:5,2:3], sumInj[1:5,2:3], sumInj[1:5,1]), big.mark = "'")

grid.table(rbind(temp,setNames(data.frame("","","","","",""), names(temp)), 
                 setNames(data.frame("TOTAL", format(sum(sumFatalities[1:5,2]), big.mark = "'"),
                 sum(sumFatalities[1:5,3]),format(sum(sumInj[1:5,2]), big.mark = "'"),
                 sum(sumInj[1:5,3]),""),names(temp))), cols = c("EVTYPE", "Fatalities", "%", "Injuries",
                 "%", "EVTYPE"), rows = NULL,  ttheme_default(base_size = 9))
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png)

In the following figures,
indeed, it is possible to see how they are distributed for the first 5 calamities more harmful.


```r
library(RColorBrewer)
colors = brewer.pal(10,"Spectral")

par(oma= c(0,0,3,0), mfrow=c(1,2),cex=0.5, cex.main=2)

pie(sumFatalities$Fatal[1:5],sumFatalities$EVTYPE[1:5], col=colors, main = "Fatalities")
pie(sumInj$Injury[1:5],sumInj$EVTYPE[1:5], col=colors, main = "Injuries")
title(main = "The 5 more armful events: comparison between Fatality and Injury", outer = T)
box("outer")
```

![The 5 weather events that casues Fatalities and Ingiuries](figure/unnamed-chunk-15-1.png)








#### Across the United States, which types of events have the greatest economic consequences?





