---
title: "Group_06"
output: html_document
date: '2022-03-11'
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE,comment = NA, message = FALSE, warning = FALSE)
```

```{r libraries}
library(ggplot2)
library(utils)
library(tidyverse)
library(skimr)
library(GGally)
library(janitor)
library(sjPlot)
```

```{r import and wrangle data}
# import data
films <- read.csv("dataset6.csv")
# create column with "good" for ratings > 7 and "bad" for ratings <= 7 
films <- films %>% 
  mutate(rank = cut(rating,breaks=c(0,7,10),labels=c("bad","good")))
# view data
glimpse(films)
# numerical summaries
films %>% 
  skim()
# turn genre into a categorical variable
films <- films %>% 
  mutate(genre = as.factor(genre))
# There's some missing data for length. Remove those points? Replace them with median length? Replace with median length for that genre?
# check baseline categories for each categorical variable
levels(films$genre)  # baseline = action
levels(films$rank)   # baseline = bad
```


```{r explore data}
# check for collinearity (nothing obvious)
films %>% 
  select(year,length,budget,votes) %>% 
  ggpairs(films)

# look at rank with year (nothing super obvious)
ggplot(films,aes(y=year,fill=rank))+
  geom_boxplot()
# look at rank with length (lots of outliers for bad and some overlap of IQR, but looks like shorter may be better)
ggplot(films,aes(y=length,fill=rank))+
  geom_boxplot()
# look at rank with budget (overlap of IQR, but bigger budget seems to be good)
ggplot(films,aes(y=budget,fill=rank))+
  geom_boxplot()
# look at rank with votes (lots of outliers- would need to remove to see how this affects the response)
ggplot(films,aes(y=votes,fill=rank))+
  geom_boxplot()
# look at rank with genre (seems like animation, comedy, documentary and short films are more likely to be good, whereas action, drama and romance more likely to be bad)
films  %>% 
  tabyl(genre, rank) %>% 
  adorn_percentages() %>% 
  adorn_pct_formatting() %>% 
  adorn_ns() # To show original counts 
ggplot(films, aes(x= rank,  y = ..prop.., group=genre, fill=genre)) + 
    geom_bar(position="dodge", stat="count") +
    labs(y = "Proportion")
# not a usual plot but interesting to see the categories against each other a little easier
ggplot(films,aes(x=genre,y=rank,col=rank))+
  geom_jitter()
```



```{r}
head(films)
##fit model with all variables
mod.films<-glm(rank~year+length+budget+votes+genre,data=films, family = binomial(link = "logit"))

mod.films %>%
  summary()
```

```{r}
##because the p-value of votes is not significant, so remove votes
mod.films2<-glm(rank~year+length+budget+genre,data=films, family = binomial(link = "logit"))

mod.films2 %>%
  summary()

```


```{r}
## calculate the odds scale the regression coefficients
plot_model(mod.films2, show.values = TRUE,
title = "", show.p = TRUE,value.offset = 0.25)
```


```{r}
## calculate the estimated probabilities
plot_model(mod.films2, type = "pred", title = "",
axis.title = c("", "Probability of good film"))
```
