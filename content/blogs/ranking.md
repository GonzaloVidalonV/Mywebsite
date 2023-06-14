---
categories:  
- ""    #the front matter should be like the one found in, e.g., blog2.md. It cannot be like the normal Rmd we used
- ""
date: "2021-09-30"
description: Risk-Return of DJIA stocks # the title that will show up once someone gets to this page
draft: false
image: spices.jpg # save picture in \static\img\blogs. Acceptable formats= jpg, jpeg, or png . Your iPhone pics wont work

keywords: ""
slug: ranking # slug is the shorthand URL address... no spaces plz
title: Ranking of Master's in Finance - Financial Times
---
  

```r
library(readxl)
library(tidyverse)
library(wbstats)
library(tictoc)
library(skimr)
library(countrycode)
library(here)
library(DBI)
library(dbplyr)
library(arrow)
library(rvest)
library(robotstxt) # check if we're allowed to scrape the data
library(scales)
library(sf)
library(readxl)
library(shiny)
library(gapminder)
library(GGally)
library(janitor)

```

## Open the database

```r
ranking <- read_excel("C:/Users/gonza/Desktop/Personal Projects/Master Rankings/ranking-data-1685405201165.xlsx") %>% 
  clean_names()
```

```r
#glimpse at variables
glimpse(ranking)
```

```r
head(ranking)
```

```r

reduced_names <- c("HEC", "ESCP", "Skema", "Essec", "Edhec", "Oxford", "Tsinghua", "IE", "LBS", "MIT", "Nova",
                   "Warwick", "Bocconi", "St Gallen", "WHU", "Imperial", "Católica", "Esade", "Peking", "Stockholm",
                   "Shanghai", "Bayes", "Rennes", "Grenoble", "McGill", "Trinity", "Frankfurt", "HEC Lausanne",
                   "Carlos III", "Kozminski", "Rotterdam", "Eada", "CUHK", "ISEG", "Vlerick", "UCD", "USI", "Henley",
                   "Lund", "SMU", "Iscte", "Edinburgh", "Rochester", "Cranfield", "Texas", "Lancaster", "Tilburg",
                   "BI Norwegian", "Porto", "Exeter", "Lucerne", "Amsterdam", "Durham", "Maryland", "Liverpool")

ranking <- ranking %>% mutate(short_name = reduced_names)
```

```r
#Plotting the Salary today in USD against Unis

p1 <-ranking %>% 
  mutate(number=as.integer(number),
         salary_today_us = as.integer(str_replace(salary_today_us, ",", "")),
         international_students_percent=as.integer(international_students_percent),
         internships_percent = as.integer(internships_percent)) %>% 
  filter(number<=20) %>% 
  ggplot(aes(y=fct_reorder(short_name, -number), x=salary_today_us)) +
  geom_col() +
  geom_text(aes(label = salary_today_us),
            hjust = 1.5,
             vjust = 0.5,
             colour = "white", 
             size = 3) +
  theme(axis.text.x = element_text(angle = 90)) +
  theme_bw() +
  labs(y="", x="Salary Today in USD")


p1
```

<img src="/blogs/Ranking_files/figure-html/unnamed-chunk-5-1.png" width="648" style="display: block; margin: auto;" />


```r

ranking %>% 
  mutate(number=as.integer(number),
         salary_today_us = as.integer(str_replace(salary_today_us, ",", "")),
         international_students_percent=as.integer(international_students_percent),
         internships_percent = as.integer(internships_percent),
         location_main_campus = as.factor(location_main_campus),
         salary_percentage_increase=as.integer(salary_percentage_increase)) %>% 
  
  filter(number<=15) %>% 
  ggplot(aes(x=international_students_percent, 
             y=salary_today_us, 
             size=internships_percent, 
             color=location_main_campus)) +
  geom_point() +
  geom_text(aes(label = short_name),
            hjust = 0.5,
             vjust = 1.5,
             colour = "black",
             size = 3.5) +
  #scale_x_continuous(limits = c(0, 100)) +
  #scale_y_continuous(limits = c(50000, 200000)) +
  theme_bw() +
  labs(x="International Students (%)", y="Salary Today in USD") +
  ggtitle("Top Universities: Salary and international students") + 
  guides(size=guide_legend(title="Internships Achieved (%)"),
         color=guide_legend(title="Main Campus Location"))

```
<img src="/blogs/Ranking_files/figure-html/unnamed-chunk-6-1.png" width="648" style="display: block; margin: auto;" />



```r
ranking %>% 
  mutate(number=as.integer(number),
         salary_today_us = as.integer(str_replace(salary_today_us, ",", "")),
         international_students_percent=as.integer(international_students_percent),
         internships_percent = as.integer(internships_percent)) %>% 
  select(number, short_name, internships_percent)
```

```r
ranking %>% 
  mutate(number=as.integer(number),
         salary_today_us = as.integer(str_replace(salary_today_us, ",", "")),
         international_students_percent=as.integer(international_students_percent),
         internships_percent = as.integer(internships_percent),
         location_main_campus = as.factor(location_main_campus),
         salary_percentage_increase=as.integer(salary_percentage_increase)) %>% 
  group_by(location_main_campus) %>% 
  summarise(mean_salary=mean(salary_today_us), count=n()) %>% 
  #slice_max(order_by = mean_salary, n=10) %>% 
  
  ggplot(aes(y=fct_reorder(location_main_campus, mean_salary), x=mean_salary)) +
  geom_col() +
  geom_text(aes(label = round(mean_salary,0)),
            hjust = 1.5,
             vjust = 0.5,
             colour = "white", 
             size = 3) +
  theme(axis.text.x = element_text(angle = 90)) +
  theme_bw() +
  labs(y="Main Campus Location", x="Mean Salary in USD")

  
```

<img src="/blogs/Ranking_files/figure-html/unnamed-chunk-8-1.png" width="648" style="display: block; margin: auto;" />



