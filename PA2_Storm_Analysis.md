# Reproducible Research: <br><small><b>Peer Assessment 2</b></small>
<a href='mailto:svicente99@yahoo.com' title='twitter:@svicente99'>Sergio Vicente</a>  
February 20th, 2015  

<br><br>
You may get source code of this at <https://github.com/svicente99/RepData_PeerAssessment2> 

And html version is available at RPubs: <http://rpubs.com/svicente99/ReprodResearch_Peer_Assesment2>

* * * *

### Synopsis

This analysis aims to reveal most harmful storm events that cause problems for population health in U.S.A. As well, it points out greatest economic losses in consequence of this kind of events.
It's based on a database provided for the NOAA - National Oceanic and Atmospheric Administration's - which tracks major storms and weather events in the whole country; data are collected from 1950 thru 2011.

All values obtained from a compressed file are summarized at specific data frames, basically a subset of event types and features of interest: frequency of cases and total damage cost.

After having developed tables and plots to this report, we may conclude that **wind storms** are the worst event types that cause injuries, fatalities and economic damages. They always appear among top ten situations ranked in descending order. _'TORNADO'_ is coincidentally the 1st one cause of severe problems with storms in the United States, as demonstrated by the figures you can see above.

------------

#### Introduction

Storms and other severe weather events can cause both public health and economic problems for communities and municipalities. Many severe events can result in fatalities, injuries, and property damage, and preventing such outcomes to the extent possible is a key concern.

This project involves exploring the U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database. This database tracks the characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property damage.

---------

#### Data Sourcing

The data for this assignment can be downloaded from the course web site:

* Dataset: <a href="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2">Storm Data</a> [47 MBytes]

The file has comma-separated-value format, compressed via the bzip2 algorithm to reduce its size. 

There is also some documentation of the database available. See above links to know how some of the variables are constructed/defined.

* [National Weather Service Storm Data Documentation]
    <https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf>

* [National Climatic Data Center Storm Events FAQ]
    <https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2/_doc%2FNCDC%20Storm%20Events-FAQ%20Page.pdf>

The events in the database start in the year 1950 and end in November 2011. In the earlier years of the database there are generally fewer events recorded, most likely due to a lack of good records. More recent years should be considered more complete.

---------

### Data Processing

Setting parameters and files to be processed:


```r
# MAIN PARAMETERS
DATA_FOLDER <- "./data"  # subdirectory of current named 'data'
URL_DATA <- "http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2F"
ZIP_FILE <- "StormData.csv.bz2"
CSV_DATA <- "StormData.csv"
# -----------------------------------------------------------------

# cache file used in this job
cached_file = paste( DATA_FOLDER, CSV_DATA, sep = "/" );
bzip_file = paste( DATA_FOLDER, ZIP_FILE, sep = "/" );
```

Getting data file and assembling the main data frame (df):


```r
# check if the zip_file is already saved ("cached") onto disk

if( !file.exists(cached_file) ) {
    url = paste(URL_DATA, ZIP_FILE, sep="")
    if( !file.exists(DATA_FOLDER) )  dir.create(DATA_FOLDER)
	  if( !file.exists(bzip_file) ) { 
		  # if it is not available --> Download it !
		  downloadStatus = download.file(url, method="internal", destfile=bzip_file)
	      if(downloadStatus != 0)  stop("Download failed from URL: ", url)
	  }
	  # read a bzip file and set to a data frame
	  df <- read.csv(bzfile(bzip_file))
	  write.csv(df, file = cached_file, row.names = TRUE)
} else {
    # read data from zip file already writen on disk
	  df <- read.csv(cached_file)
}

# above, we have a extract of ten lines of main data available to analyse
head(df)
```

```
##   X STATE__           BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1 1       1  4/18/1950 0:00:00     0130       CST     97     MOBILE    AL
## 2 2       1  4/18/1950 0:00:00     0145       CST      3    BALDWIN    AL
## 3 3       1  2/20/1951 0:00:00     1600       CST     57    FAYETTE    AL
## 4 4       1   6/8/1951 0:00:00     0900       CST     89    MADISON    AL
## 5 5       1 11/15/1951 0:00:00     1500       CST     43    CULLMAN    AL
## 6 6       1 11/15/1951 0:00:00     2000       CST     77 LAUDERDALE    AL
##    EVTYPE BGN_RANGE BGN_AZI BGN_LOCATI END_DATE END_TIME COUNTY_END
## 1 TORNADO         0                                               0
## 2 TORNADO         0                                               0
## 3 TORNADO         0                                               0
## 4 TORNADO         0                                               0
## 5 TORNADO         0                                               0
## 6 TORNADO         0                                               0
##   COUNTYENDN END_RANGE END_AZI END_LOCATI LENGTH WIDTH F MAG FATALITIES
## 1         NA         0                      14.0   100 3   0          0
## 2         NA         0                       2.0   150 2   0          0
## 3         NA         0                       0.1   123 2   0          0
## 4         NA         0                       0.0   100 2   0          0
## 5         NA         0                       0.0   150 2   0          0
## 6         NA         0                       1.5   177 2   0          0
##   INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP WFO STATEOFFIC ZONENAMES
## 1       15    25.0          K       0                                    
## 2        0     2.5          K       0                                    
## 3        2    25.0          K       0                                    
## 4        2     2.5          K       0                                    
## 5        2     2.5          K       0                                    
## 6        6     2.5          K       0                                    
##   LATITUDE LONGITUDE LATITUDE_E LONGITUDE_ REMARKS REFNUM
## 1     3040      8812       3051       8806              1
## 2     3042      8755          0          0              2
## 3     3340      8742          0          0              3
## 4     3458      8626          0          0              4
## 5     3412      8642          0          0              5
## 6     3450      8748          0          0              6
```

Selecting all diferent types of interesting events (fatalities, injuries and damage cost):


```r
df2 <- aggregate(df$INJURIES, by=list(Event=df$EVTYPE), sum)
injuriesByEvent <- df2[order(df2$x),]
names(injuriesByEvent)[names(injuriesByEvent)=="x"] <- "Total_of_Injuries"
# above, ten most severe events that cause more injuries to U.S. population 
tail(injuriesByEvent,10)
```

```
##                 Event Total_of_Injuries
## 238              HAIL              1361
## 759 THUNDERSTORM WIND              1488
## 147       FLASH FLOOD              1777
## 424         ICE STORM              1975
## 269              HEAT              2100
## 452         LIGHTNING              5230
## 123    EXCESSIVE HEAT              6525
## 164             FLOOD              6789
## 854         TSTM WIND              6957
## 830           TORNADO             91346
```

==> There were 985 types of storm events identified by this summary.


```r
df2 <- aggregate(df$FATALITIES, by=list(Event=df$EVTYPE), sum)
fatalitiesByEvent <- df2[order(df2$x),]
names(fatalitiesByEvent)[names(fatalitiesByEvent)=="x"] <- "Total_of_Fatalities"
# above, ten most severe events that cause more fatalities to U.S. population 
tail(fatalitiesByEvent,10)
```

```
##              Event Total_of_Fatalities
## 11       AVALANCHE                 224
## 354      HIGH WIND                 248
## 581    RIP CURRENT                 368
## 164          FLOOD                 470
## 854      TSTM WIND                 504
## 452      LIGHTNING                 816
## 269           HEAT                 937
## 147    FLASH FLOOD                 978
## 123 EXCESSIVE HEAT                1903
## 830        TORNADO                5633
```


```r
# adding both associated costs to damage => PROPDMG + CROPDMG and express values in US$x1,000
df$DAMAGE_COSTS <- round((df$PROPDMG + df$CROPDMG)/1000,3)
df2 <- aggregate(df$DAMAGE_COSTS, by=list(Event=df$EVTYPE), sum)
costDamage <- df2[order(df2$x),]
names(costDamage)[names(costDamage)=="x"] <- "Total_Cost"
# above, ten most severe events that cause more economic damages to U.S. population 
tail(costDamage,10)
```

```
##                  Event Total_Cost
## 972       WINTER STORM    134.701
## 354          HIGH WIND    341.984
## 783 THUNDERSTORM WINDS    464.221
## 452          LIGHTNING    606.809
## 759  THUNDERSTORM WIND    943.056
## 164              FLOOD   1067.962
## 238               HAIL   1268.015
## 854          TSTM WIND   1444.200
## 147        FLASH FLOOD   1599.270
## 830            TORNADO   3308.232
```


```r
# obtaining year of any row in dataframe to summarize costs annually
df$YEAR <- as.integer(format(as.Date(df$BGN_DATE, "%m/%d/%Y 0:00:00"), "%Y"))
costByYear <- aggregate(df$DAMAGE_COSTS, by=list(Year=df$YEAR), sum)
names(costByYear)[names(costByYear)=="x"] <- "Annual_Cost"
tail(costByYear,10)
```

```
##    Year Annual_Cost
## 53 2002     404.889
## 54 2003     490.014
## 55 2004     485.245
## 56 2005     476.697
## 57 2006     515.346
## 58 2007     514.838
## 59 2008     873.942
## 60 2009     612.961
## 61 2010     653.724
## 62 2011     918.385
```

And lastly, total values of economic losses with storms (2011-2002) listed from 2002 to next ten years.

### Results

Drawing two graphs, each one associated with a **kind of harmful to population**:


```r
## creating a panel layout 1x2, setting its margins
par(mfrow=c(1, 2),mar=c(3, 6, 2, 1), oma=c(1.5, 2, 2, 1))													

cEvents_I <- tail(injuriesByEvent$Event,10)
cEvents_F <- tail(fatalitiesByEvent$Event,10)

barplot(tail(injuriesByEvent$Total_of_Injuries,10), main="Injuries", 
        horiz=TRUE, xlim=c(0,90000), names.arg=cEvents_I, las=1, cex.names=0.7)
barplot(tail(fatalitiesByEvent$Total_of_Fatalities,10), main="Fatalities", 
        horiz=TRUE, xlim=c(0,6000), names.arg=cEvents_F,  las=1, cex.names=0.7)

mtext( "TOP TEN EVENT TYPES THAT CAUSE HARMFUL IN U.S. POPULATION", side=3, outer=TRUE, col="blue", font=2, cex=1.15 )  
mtext( "_number of cases_", side=1, outer=TRUE, col="blue", font=1, cex=0.9 )  
box("outer", col="maroon", lwd=3) 
```

![](PA2_Storm_Analysis_files/figure-html/unnamed-chunk-7-1.png) 


Drawing one graph associated with **cost of damage** caused by storm events:


```r
## creating a panel, setting its margins
par(mfrow=c(1, 1),mar=c(3, 7, 2, 1), oma=c(1.5, 2, 2, 1))  												

cEvents_D <- tail(costDamage$Event,10)

barplot(tail(costDamage$Total_Cost,10), main="Total Cost of Damage", 
        horiz=TRUE, xlim=c(0,4000), names.arg=cEvents_D, las=1, cex.names=0.7)

mtext( "TOP TEN EVENT TYPES THAT CAUSE GREATEST ECONOMIC LOSSES IN U.S. POPULATION", side=3,   outer=TRUE, col="blue", font=2, cex=1.0 )  
mtext( "_ US$ (x 1,000) _", side=1, outer=TRUE, col="blue", font=1, cex=0.9 )  
box("outer", col="maroon", lwd=3) 
```

![](PA2_Storm_Analysis_files/figure-html/unnamed-chunk-8-1.png) 


And the last graph is also related to **cost of damage** but observed along years of the research:


```r
par(mfrow=c(1, 1),mar=c(3, 3, 2, 2), oma=c(1.5, 2, 2, 1))    											

plot(costByYear$Year, costByYear$Annual_Cost, type="o", xlab="Year", ylab="US$ x 1,000")

# Label x-axis with years
nYears = nrow(costByYear)
axis(1, at=c(1:nYears), lab=costByYear$Year, cex.axis=0.8)
title(main="Total Annual Cost of Damage caused by Events Storm", col.main="blue", font.main=4)
mtext( "Year", side=1, outer=TRUE, col="blue", font=1, cex=0.9 )  
mtext( "_ US$ (x 1,000) _", side=2, outer=TRUE, col="blue", font=1, cex=0.8 )  
box("outer", col="maroon", lwd=3) 
```

![](PA2_Storm_Analysis_files/figure-html/unnamed-chunk-9-1.png) 

* * * *

## <u>Conclusion</u>

In relation to harmful events for the U.S. population, it's clear that _'TORNADO'_ is the worst storm weather between them. It's inserted into the group of **wind storms**, like _'THUNDERSTORM WINDS'_ and _'HIGH WINDS'_. Other groups that have importance are **flood** [_'FLOOD'_] and **heat** [_'HEAT'_]. Both, injuries and fatalities, are very affected by them, varying only the rank order.

On the other hand, when we examine economic damages it differs quite nothing; only order of events. However, _'TORNADO'_ still prevails as the first! Its damage value achieves the cypher of more than 3 millions of dollars to families and properties along these years (from 1950). 

And we look toward to historical series of values totalized year by year, we can observe that it's coming up - almost 1 million US$ were lost in damage caused by weather storms. Curiously, in 1992 the level of the series had a big _ramp-up_, which it worths to be investigated, possibly, by statistical inference or time-series analysis, for instance. In 2009 and 2010 it turned back to decrease, but the tendency seems to be ascendent.

The intuitive perception that **Tornado** is one of the most ferocious storms that cities and population can fight has been confirmed by this quick analysis - more than double of worst cases or greatest costs is due to that terrific storm.

* * * *
