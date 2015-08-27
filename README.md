---
title: "Hyper-heatwave predictive models"
author: "G. Brooke Anderson, Keith Oleson, Bryan Jones, Roger D. Peng"
date: "August 25, 2015"
output: html_document
---

# Overview

We have developed several predictive models to classify whether a heatwave is a hyper-heatwave, which we define as a heatwave associated with a 20% or higher increase in all-cause mortality risk in the community in which it occurs. The methods used to develop these models are fully described in a paper we are currently preparing to submit ([Reference]). Once the paper is published, we will provide a link to the paper's web address.

This repository includes three main models, all of which use, as predictive variables of whether a heatwave is a hyper-heatwave, a variety of characteristics of the heatwave. These models are: 

Model           | File name
--------------  | ----------
Tree model      | `unpr.tree.rose.RData`
Bagging model   | `bag.tree.rose.RData`
Boosting model  | `boost.tree.rose.RData`

We have also included three example datasets that we use to show how to use these models can be used to explore trends in hyper-heatwaves. Those three datasets are:

Dataset                                           | File name
------------------------------------------------- | ---------
Projected heatwaves in 82 US communities, 2061--2080 | `projected_heatwaves.txt`
Projected populations in 82 US communities, 2061--2080 | `projected_populations.csv`
Land areas for 82 US communities                      | `land_area.csv`

We explain these files further [below](#example_data).

Finally, for use in sensitivity analyses, we have included three health-based models developed using an alternative threshold of 19% to define hyper-heatwaves (i.e., a hyper-heatwave was defined as any heatwave associated with a 19% or higher increase in all-cause mortality risk in the community). These models are: 

Model           | File name
--------------  | ----------
Tree model      | `unpr.tree.rose19.RData`
Bagging model   | `bag.tree.rose19.RData`
Boosting model  | `boost.tree.rose19.RData`

These models were developed as a part of a larger project on the Benefits of Reducing Anthropogenic Climate changE ([BRACE](https://chsp.ucar.edu/brace-benefits-reduced-anthropogenic-climate-change); O'Neill and Gettelman, in prep.), which focuses on characterizing the difference in impacts driven by climate outcomes resulting from the forcing associated with RCP 8.5 and 4.5.

# Loading models into R

All models are saved as R objects. Provided they are saved within the working directory for R, they can be loaded using the `load()` command. Once you load the files, these models will be available as the R objects `unpr.tree.rose`, `bag.tree.rose`, and `boost.tree.rose`:


```r
load("unpr.tree.rose.RData")
load("bag.tree.rose.RData")
load("boost.tree.rose.RData")
```

A full description of the development of these models is provided in [Anderson et al., in prep]. 

# <a name="example_data"></a>Example data

We have included three example data files, to show how these models can be used to explore projected trends in hyper-heatwaves. As long as the datasets are saved in R's working directory, they can be read into an R session using the following code:


```r
proj_hws <- read.table("projected_heatwaves.txt", header = TRUE)
proj_pops <- read.csv("projected_populations.csv", header = TRUE)
land_area <- read.csv("land_area.csv", header = TRUE)
```

### Projected heatwaves dataset

The first file, `projected_heatwaves.txt`, is a dataset of all heatwaves projected in the 82 study communities considered in our paper, covering the years 2061--2080. These projections were generated using the National Center for Atmospheric Research's Community Earth System Model (CESM). There are from the second ensemble member of a large-ensemble run of CESM (CESM-LE), under the Representative Concentration Pathway 8.5 emission scenario. 

This dataset includes the following variables:

Column name          | Meaning
-------------------- | ----------------------------------
city                 | Abbreviation for community in which heatwave occurred
mean.temp            | Average of daily mean temperature during the heatwave (&deg;F)
max.temp             | Highest daily mean temperature during the heatwave (&deg;F)
min.temp             | Lowest daily mean temperature during the heatwave (&deg;F)
length               | Length of the heatwave in days
start.doy            | Day of year the heatwave started (e.g., Jan. 1 = 1; Feb. 1 = 32)
start.month          | Month the heatwave started
days.above.80        | Number of days in heatwave with mean temperature above 80&deg;F
days.above.85        | Number of days in heatwave with mean temperature above 85&deg;F
days.above.90        | Number of days in heatwave with mean temperature above 90&deg;F
days.above.95        | Number of days in heatwave with mean temperature above 95&deg;F
days.above.99th      | Number of days in heatwave with mean temperature above the community's 99<sup>th</sup> percentile of mean daily temperature
days.above.99.5th    | Number of days in heatwave with mean temperature above the community's 99.5<sup>th</sup> percentile of mean daily temperature
first.in.season      | Whether the heatwave was the first of the year in its community 
mean.temp.quantile   | Quantile of the average of daily mean temperature during the heatwave compared to the community's year-round temperature distribution
max.temp.quantile    | Quantile of the highest daily mean temperature during the heatwave compared to the community's year-round temperature distribution
min.temp.quantile    | Quantile of the lowest daily mean temperature during the heatwave compared to the community's year-round temperature distribution
mean.temp.1          | Average daily temperature in the community in which the heatwave occurred
mean.summer.temp     | Average warm season (May--September) temperature in the community in which the heatwave occurred

[Something here about adaptation scenario in the example data.]

Full details on the creation of this dataset are available in Oleson et al. 2015. 

### Projected populations dataset

The second file, `projected_populations.csv`, is a dataset that includes the projected populations of all 82 study communities for 2061--2080 under two different population scenarios, Shared Socioeconomic Pathways 3 and 5. 

This dataset has the following variables: 

Column name          | Meaning
-------------------- | ----------------------------------
city                 | Community (abbreviation)
county               | County name
statename            | State name
fips                 | FIPS county code
SSP3                 | Projected population under Shared Socioeconomic Pathway 3
SSP5                 | Projected population under Shared Socioeconomic Pathway 5

*Note: Since some of the study communities cover more than one county, there are multiple rows for some study communities in this dataset.* 

### Land area dataset

The third file, `land_area.csv`, provides the land area of each of the study communities. It has the following variables:

Column name          | Meaning
-------------------- | ----------------------------------
city                 | Community (abbreviation)
arealand             | Land aread of the community

# Formatting a heatwave dataset for the models

To run through the predictive models, data describing heatwaves must be in the format of a dataframe and must include all of the heatwave characteristics in `proj_hws`, plus two characteristics generated from the population data, community population and community population density. 

Here is an example of code to add the variables `pop100`, which gives population size under the SSP5 scenario, and `pop.density`, which gives the population density under the SSP5 scenario. This code uses functions from the `dplyr` function to sum populations across different counties for a community, join the population data to the land area data by community, and then calculate the population density.


```r
library(dplyr)

proj_pops <- group_by(proj_pops, city) %>%
        summarise(pop100 = sum(SSP3)) %>%
        left_join(land_area) %>%
        mutate(pop.density = pop100 / arealand) %>%
        dplyr::select(-arealand)
head(proj_pops)
```

```
## Source: local data frame [6 x 3]
## 
##   city  pop100  pop.density
## 1  akr  670335 6.270976e-04
## 2 albu  671311 2.222872e-04
## 3 atla 1886749 9.141814e-04
## 4 aust 1004847 3.921679e-04
## 5 bake  755850 3.584776e-05
## 6 balt  819231 3.914476e-03
```

Now this data can be merged in with the heatwave characteristics given in `proj_hws`:


```r
proj_hws <- left_join(proj_hws, proj_pops, by = "city")
```

The `proj_hws` dataset now has all the variables needed to run through the health-based models: 


```r
colnames(proj_hws)
```

```
##  [1] "city"               "mean.temp"          "max.temp"          
##  [4] "min.temp"           "length"             "start.doy"         
##  [7] "start.month"        "days.above.80"      "days.above.85"     
## [10] "days.above.90"      "days.above.95"      "days.above.99th"   
## [13] "days.above.99.5th"  "first.in.season"    "mean.temp.quantile"
## [16] "max.temp.quantile"  "min.temp.quantile"  "mean.temp.1"       
## [19] "mean.summer.temp"   "pop100"             "pop.density"
```

To use these health-based models for a dataset of heatwaves, it is important that all the same variables be calculated for the heatwaves and that they be saved using the same column names as those used here. This is how the health-based models will recognize the heatwave characteristics required to predict from the model.

# Using the models

Once a heatwave dataset is in the correct format, the models can be applied to the dataset to predict which heatwaves are likely to be hyper-heatwaves, based on these heatwave characterstics. Since the models generate a lot of false positives, the total number of projected hyper-heatwaves will then need to be adjusted to account for the models' false positive rates (precision). 

### Generating predictions from the models

To have access to prediction methods for the three models, it is necessary to load the following three R packages:


```r
library(tree)
library(randomForest)
library(gbm)
```

For all three models, the `predict` function can be used to use the model to predict outcomes for a new dataset, in this case the heatwave dataset `proj_hws`:


```r
projs_tree <- predict(unpr.tree.rose,
                      newdata = proj_hws, 
                      type = "class")

projs_bag <- predict(bag.tree.rose,
                     newdata = proj_hws)

projs_boost <- predict(boost.tree.rose,
                       newdata = proj_hws, 
                       n.trees = 500)
```

The methods for these three models differ somewhat, and as a result, the prediction calls differ and the predictive output differs in format. For example, while the predictions from the tree and bagging models are a string of predictions for the heatwaves-- either "very" if the heatwave is predicted to be a hyper-heatwave or "other" otherwise-- the predictions from the boosting model are numeric. 


```r
head(projs_tree)
```

```
## [1] other other other other other other
## Levels: other very
```

```r
head(projs_bag)
```

```
##     1     2     3     4     5     6 
## other other other other other other 
## Levels: other very
```

```r
head(projs_boost)
```

```
## [1] -0.8212893 -0.8099903 -0.8212893 -0.8212893 -0.8212893 -0.8189654
```

The outputs from the boosting model can be converted to the same format as the other predictions using the following code:


```r
projs_boost <- factor(ifelse(projs_boost > 0, "very", "other"))
head(projs_boost)
```

```
## [1] other other other other other other
## Levels: other very
```

### Estimating hyper-heatwave frequency from the predictions

The following equations can be used to estimate the total number of hyper-heatwaves projected within a dataset of heatwaves: 

$$ H_{h} = H_{ph} * P + H_{pl} * F $$

where: 

- $H_{h}$ is the estimated number of hyper-heatwaves
- $H_{ph}$ is the number of heatwaves predicted by the model to be hyper-heatwaves
- $P$ is the model's estimated precision
- $H_{pl}$ is the number of heatwaves predicted by the model to be less dangerous heatwaves
- $F$ is the model's estimated false precision rate

This equation takes the total number of heatwaves predicted by the model to belong to each class and adjusts these values by estimates of the model's precision and false omission rates. 

The equation requires estimates of the precision and false omission rates. We estimated these for each of the three models using Monte Carlo cross-validation (for more details, see our paper describing the development of these models). These are our estimates for these values for the three models:

Model          | Precision   | False omission rate
-------------- | ----------- | --------------------
Tree model     | 2.6%        | 0.0%
Bagging model  | 2.6%        | 0.0%
Boosting model | 2.3%        | 0.0%

Here is an example of how to apply this equation to projections from the bagging model. First, a vector is created that has the adjustment rates (false omission rates and precision rates). 


```r
adjustment_rates <- c(0.0, 2.6) / 100

unadj_counts <- table(projs_bag)
unadj_counts
```

```
## projs_bag
## other  very 
## 10719  1836
```

```r
sum(unadj_counts * adjustment_rates)
```

```
## [1] 47.736
```

Based on this calculation, our projection for this set of scenarios is that we would expect around 0 hyper-heatwaves in the 82 study communities for 2061--2080. 

### Estimating hyper-heatwave frequency from the predictions

Exposure to hyper-heatwaves combines both the frequency of such heatwaves with projections of how long they will last and how many people will be living in the affected community. The following equation can be used to project hyper-heatwave exposure using the predictions from our health-based models: 

$$ E_{h} = P\sum_{i}(D_{i}N_{i}) + F\sum_{j}(D_{j}N_{j}) $$

where: 

- $E_{h}$ is the estimated exposure to hyper-heatwaves, in person-days
- $P$ is the model's estimated precision
- $i$ indexes across all heatwaves classified as hyper-heatwaves by the predictive model
- $D_{i}$ is the length of heatwave $i$ in days
- $N_{i}$ is the population of the community in which heatwave $i$ occurs
- $F$ is the model's estimated false omission rate
- $j$ indexes across all heatwaves classified as less dangerous heatwaves by the predictive model
- $D_{j}$ is the length of heatwave $j$ in days
- $N_{j}$ is the population of the community in which heatwave $j$ occurs

The following code gives an example of how exposure to hyper-heatwaves could be estimated in R based on `projs_bag`, the results from applying the predictive bagging model to the projected heatwave data.

First, it's necessary to combine the model predictions with data from the original heatwave dataset on heatwave length and population of the community in which the heatwave occurred:


```r
exp_projs <- data.frame(prediction = projs_bag,
                        length = proj_hws$length,
                        pop = proj_hws$pop100)
exp_projs[sample(1:nrow(exp_projs), 6), ]
```

```
##      prediction length     pop
## 2607       very     13 5271247
## 1852      other      3 6370140
## 5982      other      4 2628754
## 944       other      2  819231
## 5546      other      2  870892
## 2275      other      4 1676978
```

Now this dataframe can be used to calculate the person-days of exposure for each heatwave and sum this exposure by predicted class of heatwave:


```r
exp_projs <- mutate(exp_projs, exposure = length * pop) %>%
        group_by(prediction) %>%
        summarise(exposure = sum(exposure))
exp_projs
```

```
## Source: local data frame [2 x 2]
## 
##   prediction     exposure
## 1      other 106250642249
## 2       very  42858657528
```

Now the estimated exposure can be calculated by adjusting these two values for the model's estimated precision and false omission rates (saved in earlier code as `adjustment_rates`):


```r
sum(exp_projs$exposure * adjustment_rates)
```

```
## [1] 1114325096
```

This number can also be presented in units of millions of person-days per year (the example dataset covers 20 years):


```r
sum(exp_projs$exposure * adjustment_rates) / 10^6 / 20
```

```
## [1] 55.71625
```

For this ensemble member, emission scenario, and population scenario, our estimated projection of exposure to hyper-heatwaves is 56 million person-days per year across the 82 study communities.

## Using the sensitivity analysis models

Projections of trends in hyper-heatwaves could be sensitive to the threshold used to define a hyper-heatwaves. Our main models were developed using the definition that a hyper-heatwave was associated with a 20% or greater increase in all-cause mortality risk.

As a sensitivity analysis, we investigated alternative definitions: we considered using thresholds of 19% and 21% increase in mortality risk to define hyper-heatwaves. Models were unchanged with the 21% threshold definition because no heatwaves in the training dataset fell between 20% and 21% increase in mortality risk. 

However, some heatwaves did fall between the 19% and 20% threshold values, and so we generated different models, to use in sensitivity analyses, based on the 19% threshold definition of a hyper-heatwave. These models are available in the following files:

Model           | File name
--------------  | ----------
Tree model      | `unpr.tree.rose19.RData`
Bagging model   | `bag.tree.rose19.RData`
Boosting model  | `boost.tree.rose19.RData`

They can be loaded and used in the same way as the main models. Here are the estimated precision and false omission rates for these three models:

Model          | Precision   | False omission rate
-------------- | ----------- | --------------------
Tree model     | 0.5%        | 0.1%
Bagging model  | 0.5%        | 0.1%
Boosting model | 0.5%        | 0.1%
