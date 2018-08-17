
Heart Disease Mortality And Farmer's Market
===========================================

##### <i>Kyoosik Kim<br>July 2018</i>

### Abstract

I explore the data sets, 'Heart Disease Mortality' and 'Farmer's Market In The US', to study the relationship between them. The goal of this project is to capture a possible link of farmer's markets to heart disease mortality.

### Introduction

The Center For Disease Control And Prevention provides the data set 'Heart Disease Mortality Data Among US Adults (35+) by State/County in 2014' which is downloadable at <a href='https://catalog.data.gov/dataset/heart-disease-mortality-data-among-us-adults-35-by-state-territory-and-county-5fb7c'>DATA.GOV</a>. This data set contains the numbers of deaths from <a href='http://www.heart.org/HEARTORG/Conditions/What-is-Cardiovascular-Disease_UCM_301852_Article.jsp#.Wz57p9JKjIU'>Cardiovascular Disease</a>, also simply called heart disease, every 100,000 population by gender and race in state/county level. Another data set 'Farmer's Market In The US' is procured from The United States Department of Agriculture at <a href='https://www.ams.usda.gov/services/local-regional/farmers-markets-and-direct-consumer-marketing'>USDA</a>. 8.7k+ Farmer's markets are listed with information of state, exact locations and items. Even though the farmer's market data is collected between 2011 and 2018, most of the data was updated in 2015 which is almost the same year of that of the heart disease data. Also considering the small variability of the data over time, therefore, these two data sets can be compared without any serious errors.

I will be studying these data to discover trends of which states have more or less heart disease mortality and whether the states have a relationship with the number of farmer's markets. The importance of these questions stem from the fact that the number one cause of death in the US is none other than heart diseases.

It has been talked a lot about what causes heart diseases such as diet habits as interests in organic/fresh foods have been growing fast. Although people now believe that bad diets could cause heart diseases thanks to researches and education, there lacks the reverse thoughts of reducing the risks with good diets.

First step to answer these questions is to explore the data sets individually. I will look into the data, one by one, to see how they are distributed and visualize on the US map to gain some ideas where heart diseases or farmer's markets are found the most and least.

Second, I will join the two data sets to find a pattern between the heart disease mortality and the number of farmer's markets. This will be shown on the map for better understanding. The anticipated result is that the mortality is found to be lower where there are many farmer's markets than where there are less.

------------------------------------------------------------------------

### Preparation

``` r
# load packages
library(ggplot2)
library(dplyr)
library(stringi)
library(readr)
library(maps)
library(mapdata)
```

#### Define Function

``` r
state.name <- tolower(state.name)
state.name[51] <- "district of columbia"
state.abb[51] <- "DC"

process_region <- function(df, state_list) {
  # state
  df$state <- state.name[match(trimws(df$state), state_list)]
  df$state <- tolower(df$state)
  
  # county
  df$county <- tolower(df$county)
  # remove unnecessary string or character
  df$county <- gsub("county", "", df$county)
  df$county <- gsub("city", "", df$county)
  df$county <- gsub("parish", "", df$county)
  df$county <- gsub("\\.", "", df$county)
  df$county <- gsub("\\'", "", df$county)
  # modify names for consistency
  df$county <- gsub("dekalb", "de kalb", df$county)
  df$county <- gsub("desoto", "de soto", df$county)
  df$county <- gsub("dupage", "du page", df$county)
  df$county <- gsub("laporte", "la porte", df$county)
  df$county <- gsub("yellowstone", "yellowstone national", df$county)
  # washington dc county
  df$county[which(df$state == "district of columbia")] <- "washington"

  # final screening
  df$state <- factor(trimws(df$state))
  df$county <- factor(trimws(df$county))
  df <- df[complete.cases(df), ]
  
  return(df)
}
```

#### Load and Process the Data set

First, I need to read the Heart Disease Mortality data set and modify it to fit the format in which common variables are refined and named the same with the Farmer's market data set. Some other process here includes dropping variables such as race and gender. These could be useful for other analysis but not for the subject of this project.

``` r
# load heart disease mortality data
heart_mortality <- read.csv('../Data/Heart_Disease_Mortality_by_County.csv')
# select columns
heart_mortality <- subset(heart_mortality, GeographicLevel == 'County')
heart_mortality <- heart_mortality[, -c(1, 4:7, 9:13, 15, 17:19)]
heart_mortality <- heart_mortality[!is.na(heart_mortality$Data_Value), ]
# change column names
colnames(heart_mortality) <- c('state', 'county', 'mortality', 
                               'gender', 'race')
# process the regional columns
heart_mortality <- process_region(heart_mortality, state.abb)
# extract "Overall" condition only
heart_mortality <- heart_mortality[(heart_mortality$gender == "Overall") 
                & (heart_mortality$race == "Overall"), ]
heart_mortality <- heart_mortality[, -c(4, 5)]
# show structure
str(heart_mortality)
```

    ## 'data.frame':    3136 obs. of  3 variables:
    ##  $ state    : Factor w/ 51 levels "alabama","alaska",..: 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ county   : Factor w/ 1903 levels "abbeville","acadia",..: 23 24 44 158 488 503 579 726 880 898 ...
    ##  $ mortality: num  105 212 258 352 306 ...

Next, the Farmer's Market data set is read and processed in the same way. The data set has a number of categorical variables of items that farmer's markets sell. Many of the items like tobu are so limitedly available as to be dropped. At last, the items that are general but can indicate freshness and organicity of a farmer's market are selected.

``` r
# load farmer's market data set
farmers_market <- read.csv('../Data/Farmers_Markets_by_County.csv')
# select columns
farmers_market <- farmers_market[, -c(1:9, 12:20, 23, 59)]
farmers_market <- farmers_market[, c(1:4, 11, 17:18, 24, 32:33)]
farmers_market <- farmers_market[farmers_market$County != "", ]
farmers_market <- farmers_market[!is.na(farmers_market$x), ]
# change column names
colnames(farmers_market)[1:4] <- c('county', 'state', 'long', 'lat')
farmers_market <- farmers_market[c(2, 1, 3:10)]
# process the regional columns
farmers_market$state <- tolower(farmers_market$state)
farmers_market <- process_region(farmers_market, state.name)
# show the structure
str(farmers_market)
```

    ## 'data.frame':    8174 obs. of  10 variables:
    ##  $ state     : Factor w/ 51 levels "alabama","alaska",..: 46 36 26 33 43 33 8 9 9 33 ...
    ##  $ county    : Factor w/ 1397 levels "abbeville","accomack",..: 198 343 92 878 354 878 874 1326 1326 164 ...
    ##  $ long      : num  -72.1 -81.7 -94.3 -73.9 -86.8 ...
    ##  $ lat       : num  44.4 41.4 37.5 40.8 36.1 ...
    ##  $ Bakedgoods: Factor w/ 2 levels "N","Y": 2 2 2 2 2 2 1 2 2 1 ...
    ##  $ Herbs     : Factor w/ 2 levels "N","Y": 2 2 2 2 2 2 2 2 2 1 ...
    ##  $ Vegetables: Factor w/ 2 levels "N","Y": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ Nuts      : Factor w/ 2 levels "N","Y": 1 1 1 2 1 2 1 2 1 1 ...
    ##  $ Beans     : Factor w/ 2 levels "N","Y": 2 1 1 1 1 1 1 2 1 2 ...
    ##  $ Fruits    : Factor w/ 2 levels "N","Y": 2 2 2 1 2 2 2 2 2 2 ...

Lastly, the population data set is brought as a supplement. Because the heart disease mortality is based on 2014, it would make the best sense to remove other years from the population matrix. Also, county and state should be seperated to be aligned with the other data sets.

``` r
# load population data set
population_county <- read.csv('../Data/PEP_2014_PEPANNRES_with_ann.csv')
# 
population_county <- population_county[-1, c(3, 10)]
colnames(population_county) <- c('geo_name', 'population')
#
population_county$geo_name <- tolower(population_county$geo_name)
geo_split <- strsplit(as.character(population_county$geo_name), split = ',')
population_county$county <- sapply(geo_split, '[', 1)
population_county$state <- sapply(geo_split, '[', 2)
population_county <- process_region(population_county, state.name)
#
population_county$population <- with(population_county,
                                     as.numeric(levels(population))[population])
#
population_county <- population_county[, -1]
population_county <- population_county[c(3, 2, 1)]
rm(geo_split)
str(population_county)
```

    ## 'data.frame':    3142 obs. of  3 variables:
    ##  $ state     : Factor w/ 51 levels "alabama","alaska",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ county    : Factor w/ 1830 levels "abbeville","acadia",..: 82 89 99 148 163 222 232 243 289 310 ...
    ##  $ population: num  55395 200111 26887 22506 57719 ...

------------------------------------------------------------------------

### Exploring the Data

In this section, I will look into the two data sets individually to see how they are distributed and what trends they have in general before they are merged for the final analysis.

#### Part I: Histogram

<b>Histogram of Heart Disease Mortality</b>

``` r
heart_disease_overall_hist <- ggplot(heart_mortality, aes(mortality)) +
  geom_histogram(binwidth = 20, color = 'gray', fill = 'lightblue') +
  ggtitle("Heart Disease Mortality in the US (2014)") +
  xlab("Number of Deaths per 100,000") + ylab("Counties") +
  scale_x_continuous(limits = c(0, 700), breaks = seq(0, 700, 100))

heart_disease_overall_hist
```

![](heart_disease_mortaltiy_and_farmers_market_files/figure-markdown_github/unnamed-chunk-3-1.png)

<b>Highest to Lowest Heart Disease Mortality by State</b>

``` r
#
NE <- c("CT","ME","MA","NH","RI","VT","NJ","NY","PA")
MW <- c("IN","IL","MI","OH","WI","IA","KS","MN","MO","NE","ND","SD")
SO <- c("DE","DC","FL","GA","MD","NC","SC","VA","WV","AL",
           "KY","MS","TN","AR","LA","OK","TX")
WE <- c("AZ","CO","ID","NM","MT","UT","NV","WY","CA","OR","WA")
PAC <- c("AK","HI")
region_list <- list(Northeast = NE, Midwest = MW, South = SO,
                    West = WE, Pacific = PAC)
```

Classifying the states follows [The US Census Region](https://www2.census.gov/geo/pdfs/maps-data/maps/reference/us_regdiv.pdf). These groups give a more intuitive idea to roughly understand which areas have the highest mortality and which areas do not.

``` r
#
mortality_state <- heart_mortality %>%
  group_by(state) %>%
  summarise(mortality_mean = mean(mortality)) %>%
  arrange(state)
#
mortality_state$state_abb <- state.abb[match(mortality_state$state, state.name)]
mortality_state$region <- sapply(mortality_state$state_abb, function(x) 
  names(region_list)[grep(x, region_list)])

#
heart_disease_state_bar <- ggplot(mortality_state, 
                                  aes(reorder(state_abb, -mortality_mean), 
                                      mortality_mean,
                                      fill = region)) +
  geom_bar(stat = 'identity', width = 0.5) +
  ggtitle("Mean Mortality by State") +
  xlab("State") + ylab("Mortality")
  
# 
heart_disease_state_bar + theme_bw() + 
  theme(axis.text.x = element_text(size = 7, angle = 75),
        legend.title = element_blank(),
        legend.text = element_text(size = 10),
        legend.position = c(0.9,0.8),
        legend.background = element_rect(fill = alpha('white', 0)))
```

![](heart_disease_mortaltiy_and_farmers_market_files/figure-markdown_github/unnamed-chunk-5-1.png)

``` r
mortality_region_rank <- aggregate(mortality_state[, "mortality_mean"], 
                                 list(mortality_state$region), mean)
mortality_region_rank[order(-mortality_region_rank$mortality_mean), ]
```

    ##     Group.1 mortality_mean
    ## 4     South       402.2686
    ## 1   Midwest       335.9101
    ## 2 Northeast       313.6581
    ## 5      West       300.0910
    ## 3   Pacific       283.2477

<b>Most and Least Farmer's Market Counts by State</b>

``` r
# 
farmers_market_state <- farmers_market %>%
  group_by(state) %>%
  summarise(count = n()) %>%
  arrange(state)

farmers_market_state$state_abb <- state.abb[match(farmers_market_state$state,
                                                  state.name)]
farmers_market_state$region <- sapply(farmers_market_state$state_abb,
                            function(x) names(region_list)[grep(x, region_list)])

#
population_state <- population_county %>%
  group_by(state) %>%
  summarise(population = sum(population)) %>%
  arrange(state)

#
farmers_market_popl_adjusted <- merge(farmers_market_state,
                                      population_state,
                                      by = "state")
farmers_market_popl_adjusted$count <- with(farmers_market_popl_adjusted,
                                           count/(population/100000))
```

populatio adjusted setup and absolute number of markets graph below

``` r
#
farmers_market_bar <- ggplot(farmers_market_state, 
                             aes(reorder(state_abb, -count), 
                                 count, fill = region)) +
  geom_bar(stat = 'identity', width = 0.5) +
  ggtitle("Farmer's Markets by State") +
  xlab("State") + ylab("Count")

#
farmers_market_bar + theme_bw() + 
  theme(axis.text.x = element_text(size = 7, angle = 75),
        legend.title = element_blank(),
        legend.text = element_text(size = 10),
        legend.position = c(0.9,0.8),
        legend.background = element_rect(fill = alpha('white', 0)))
```

![](heart_disease_mortaltiy_and_farmers_market_files/figure-markdown_github/unnamed-chunk-8-1.png)

``` r
farmers_market_region <- aggregate(farmers_market_state[, 2], 
                                      list(farmers_market_state$region), mean)
farmers_market_region[order(farmers_market_region$count), ]
```

    ##     Group.1    count
    ## 3   Pacific  60.0000
    ## 4     South 132.1765
    ## 5      West 143.3636
    ## 1   Midwest 199.5833
    ## 2 Northeast 203.8889

population adjusted graph

``` r
#
farmers_market_popl_adj_bar <- ggplot(farmers_market_popl_adjusted, 
                             aes(reorder(state_abb, -count), 
                                 count, fill = region)) +
  geom_bar(stat = 'identity', width = 0.5) +
  ggtitle("Farmer's Markets per 100,000 by State") +
  xlab("State") + ylab("Count")

#
farmers_market_popl_adj_bar + theme_bw() + 
  theme(axis.text.x = element_text(size = 7, angle = 75),
        legend.title = element_blank(),
        legend.text = element_text(size = 10),
        legend.position = c(0.9,0.8),
        legend.background = element_rect(fill = alpha('white', 0)))
```

![](heart_disease_mortaltiy_and_farmers_market_files/figure-markdown_github/unnamed-chunk-10-1.png)

``` r
farmers_market_region_adj <- aggregate(farmers_market_popl_adjusted[, 2], 
                                       list(farmers_market_popl_adjusted$region), mean)
farmers_market_region_adj[order(farmers_market_region_adj$x), ]
```

    ##     Group.1        x
    ## 4     South 2.838131
    ## 5      West 3.183809
    ## 1   Midwest 4.446609
    ## 3   Pacific 5.140722
    ## 2 Northeast 5.225179

#### Part II: Visualization on Map

<b>Heart Disease Mortality by State</b>

``` r
#
state_map <- merge(map_data("state"), mortality_state, 
                   by.x = "region", by.y = "state", all.x = TRUE)

labs <- data.frame(
  long = c(-122.064873, -122.306417),
  lat = c(36.951968, 47.644855),
  names = c("SWFSC-FED", "NWFSC"),
  stringsAsFactors = FALSE
  )  

mortality_state_map <- ggplot(data = state_map) +
  geom_polygon(aes(long, lat, fill = mortality_mean, group = group), 
               color = "black", size = 0.1) +
  coord_fixed(1.3) +
  scale_fill_gradientn(colours = rev(rainbow(2)), guide = "colorbar")

mortality_state_map
```

![](heart_disease_mortaltiy_and_farmers_market_files/figure-markdown_github/unnamed-chunk-12-1.png)

<b>Heart Disease Mortality by County</b>

``` r
# merge data frame with county map
county_map <- merge(map_data("county"), heart_mortality,
                    by.x = c('region', 'subregion'), 
                    by.y = c('state', 'county'),
                    all.x = TRUE)
county_map <- county_map[order(county_map$order), ]

mortality_county_map <- ggplot(data = county_map) +
  geom_polygon(aes(long, lat, fill = mortality, group = group), 
               color = "darkgray", size = 0.1) +
  coord_fixed(1.3) +
  scale_fill_gradientn(colours = rev(rainbow(2)), guide = "colorbar")

mortality_county_map
```

![](heart_disease_mortaltiy_and_farmers_market_files/figure-markdown_github/unnamed-chunk-13-1.png)

<b>Heart Disease Mortality and Farmer's Market</b>

``` r
# farmer's market location points
farmers_martket_loc <- data.frame(
  lat = as.vector(farmers_market$lat),
  long = as.vector(farmers_market$long)
)

mortality_county_map +
  geom_point(data = farmers_martket_loc, aes(x = long, y = lat), 
             color = "yellow", size = 0.01, alpha = 0.1)
```

![](heart_disease_mortaltiy_and_farmers_market_files/figure-markdown_github/unnamed-chunk-14-1.png)

<b>Heart Disease Mortality and Farmer's Market</b>

``` r
farmers_market_pop <- merge(farmers_market, population_county,
                            by = c("state", "county"))

farmers_martket_loc2 <- data.frame(
  lat = as.vector(farmers_market_pop$lat),
  long = as.vector(farmers_market_pop$long),
  size = as.vector(farmers_market_pop$population/100000)
)

mortality_county_map +
  geom_point(data = farmers_martket_loc2, aes(x = long, y = lat), 
             color = "yellow", size = 0.1/farmers_martket_loc2$size, alpha = 0.1)
```

![](heart_disease_mortaltiy_and_farmers_market_files/figure-markdown_github/unnamed-chunk-15-1.png)

------------------------------------------------------------------------

### Analyzing Data

<b>Correlation between Heart Disease Mortality and Farmer's Market</b>

<b>Considering Categorical Data</b>

------------------------------------------------------------------------

### Conclusion