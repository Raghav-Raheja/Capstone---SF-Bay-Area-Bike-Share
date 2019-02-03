---
title: "Capstone Project"
author: "Raghav Raheja"
date: "February 2, 2019"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
knitr::opts_chunk$set(fig.width=14, fig.height=8)
```

## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

```{r}
library(igraph)
file <- file.choose()
Station <- read.csv(file)
View(Station)

file2 <- file.choose()
Trips <- read.csv(file2)
View(Trips)

```
```{r}
length(unique(Trips$end_station_name))
Trips <- Trips[!is.na(Trips$start_station_name),]
Trips <- Trips[!is.na(Trips$end_station_name),]

# Network Between Start to End Station

edgelist <- as.matrix(Trips[c("start_station_name", "end_station_name")])
g <- graph_from_edgelist(edgelist, directed = TRUE)
g <- simplify(g)
plot.igraph(g, 
            edge.arrow.size=0,
            edge.color="black",
            edge.curved=TRUE,
            edge.width=1,
            vertex.size=2,
            vertex.color=NA, 
            vertex.frame.color=NA, 
            vertex.label=V(g)$name,
            vertex.label.cex=1,
            layout=layout.fruchterman.reingold,
            main = "Network of Trips from Start Station to End Station"
)

# The Network below shows that there are two clear and two closely interlinked interlinked clusters
```
```{r}
# Docks Per City
library(ggplot2)
library(maptools)

no_of_docks <- data.frame(Average_Dock = tapply(Station$dock_count,Station$city, mean))
par(mar = c(2,2,2,2))
barplot(no_of_docks$Average_Dock, names.arg = rownames(no_of_docks),
                                          main = "Average Docks Per City", xlab = "City", ylab = "Docks", col = "lightsalmon"
        , ylim = c(0,20))


#  + ylim(25,50) + xlim(-125,-67)

```
```{r}
colnames(Trips) <- c("Trip_id","duration","start_date","start_station_name","id","end_date","end_station_name","end_station_id","bike_id",            "subscription_type","zip_code")
Station_trip <- merge(Trips,Station, By = "id")

# Removing the Outliers by removing the points from 2 deviations from the mean
md <- mean(Station_trip$duration)              
std <- sd(Station_trip$duration)
lower.l <- md - 2*std
higher.l <- md + 2*std
Station_trip <- Station_trip[-(which(Station_trip$duration<lower.l | Station_trip$duration >higher.l)),]

# Plotting Boxplot for Trip Duration in Each City
options(scipen = 999)
boxplot(Station_trip$duration~Station_trip$city
        , main = "Trip Duration for Different Cities")
```

```{r}
barplot(data.frame(trips_per_city = tapply(Station_trip$id,Station_trip$city,length))$trips_per_city
        , names.arg = rownames(data.frame(trips_per_city = tapply(Station_trip$id,Station_trip$city,length)))
        , main = "Number of Trips made in Each City")
```

```{r}
no_of_stations <- data.frame(number = tapply(Station$id,Station$city,length))
no_of_stations$percentage <- no_of_stations$number/sum(no_of_stations$number)
pie(no_of_stations$percentage, labels = rownames(no_of_stations)
    , main = "Percentage of Stations in Each City")


```


