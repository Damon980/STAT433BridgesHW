# STAT433BridgesHW Code
---
title: "Bridges Exploration"
author: "Damon O'Connor"
date: "2/4/2021"
output: html_document
---

```{r Setup, include=FALSE}
library(data.table)
library(tidyverse)
```

```{r}
#Reading the data set from the government website for reproducibility 
data = fread("https://www.fhwa.dot.gov/bridge/nbi/2018/delimited/WI18.txt", fill = T)

#Exploratory data analysis of the bridges data set

#Starting with variable creation for age, traffic, and tolls

data = data %>% mutate(bridgeAge = 2020 - data$YEAR_BUILT_027)

data = data %>% mutate(truckTraffic = PERCENT_ADT_TRUCK_109 / 100)

data = data %>% mutate(tollRoad = (TOLL_020 != 3))

df = data %>%  select(bridgeAge, truckTraffic, tollRoad)

#Testing to see if truck drivers avoid old bridges
ggplot(data = df, aes(bridgeAge, truckTraffic))+
  geom_point()+
  geom_smooth()

#Doesn't appear so.

#Testing to see if newer bridges are tolled more
ggplot(data = df, aes(bridgeAge, tollRoad))+
  geom_point()

tollroads = df %>% filter(tollRoad == T)
nrow(tollroads)

#Turns out there are no recorded bridges on a toll road in Wisconsin in 2018

#Testing to see if older bridges have lower condition ratings

data = data %>% mutate(bridgeCondition = as.numeric(SUPERSTRUCTURE_COND_059))

ggplot(data = data, aes(bridgeAge, bridgeCondition))+
  geom_point()+
  geom_smooth()

#Interesting relationship. Maybe bridges are usually remodeled/repaired after they reach 100+ y/o?
#Could also be survivorship bias because only the good bridges from the early 1900's will have made it this long

#Does bridge length dictate the materials used to build it?

# Code Description for item 43A Structure material
# 1 Concrete
# 2 Concrete continuous
# 3 Steel
# 4 Steel continuous
# 5 Prestressed concrete *
# 6 Prestressed concrete continuous *
# 7 Wood or Timber
# 8 Masonry
# 9 Aluminum, Wrought Iron, or Cast Iron
# 0 Other

bridgeMaterial = case_when(
  data$STRUCTURE_KIND_043A == 1 ~ "Concrete",
  data$STRUCTURE_KIND_043A == 2 ~ "Concrete continuous",
  data$STRUCTURE_KIND_043A == 3 ~ "Steel",
  data$STRUCTURE_KIND_043A == 4 ~ "Steel continuous",
  data$STRUCTURE_KIND_043A == 5 ~ "Prestressed concrete",
  data$STRUCTURE_KIND_043A == 6 ~ "Prestressed concrete continuous",
  data$STRUCTURE_KIND_043A == 7 ~ "Wood or Timber",
  data$STRUCTURE_KIND_043A == 8 ~ "Masonry",
  data$STRUCTURE_KIND_043A == 9 ~ "Aluminum, Wroght Iron, or Cast Iron",
  data$STRUCTURE_KIND_043A == 0 ~ "Other"
)

data = mutate(data, bridgeMaterialDF = bridgeMaterial)

data %>% 
  group_by(bridgeMaterialDF) %>% 
  summarise(meanLength = mean(STRUCTURE_LEN_MT_049)) %>% 
  ggplot(mapping = aes(x = bridgeMaterialDF, y = meanLength))+
    geom_bar(stat = "identity")+
    theme(axis.text.x= element_text(angle = 90))

#The average brdige length (in meters) is the highest for continuous steel and prestressed
#concrete. Maybe those materials are best suited to longer structures that support
#more weight?

#But what about the max length of each material?

data %>% 
  group_by(bridgeMaterialDF) %>% 
  summarise(maxLength = max(STRUCTURE_LEN_MT_049)) %>% 
  ggplot(mapping = aes(x = bridgeMaterialDF, y = maxLength))+
  geom_bar(stat = "identity")+
  theme(axis.text.x= element_text(angle = 90))

#It seems like concrete and steel are the main materials in bridges more than
#a few meters long.
```
