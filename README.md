# PanelData
---
title: "PanelData"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

Here
```{r}
library(foreign)
Panel <-read.dta("http://dss.princeton.edu/training/Panel101.dta")
head(Panel)

coplot(y ~ year|country, type="l", data=Panel)

install.packages("car")
library(car)

Panel$country

scatterplot(y~year|country, boxplots=FALSE, smooth=TRUE, reg.line=FALSE, data=Panel)

# Here is an example of fixed regression with dummies for the countries, because those are fixed across the years

head(Panel)
fixed.dum <-lm(y ~ x1 + factor(country) -1, data=Panel)
summary(fixed.dum)
```

