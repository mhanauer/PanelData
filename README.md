# PanelData
---
title: "PanelData"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

Fixed effects example.  Remember that fixed effects asorb invariant trends from stable variables like schools and maybe year need to look into using year as an example of fixed effects.
```{r}
library(foreign)
Panel <-read.dta("http://dss.princeton.edu/training/Panel101.dta")
head(Panel)

coplot(y ~ year|country, type="l", data=Panel)

install.packages("car")
library(car)

Panel$country

scatterplot(y~year|country, boxplots=FALSE, smooth=TRUE, reg.line=FALSE, data=Panel)

# Here is an example of fixed regression with dummies for the countries, because those are fixed across the years.  Controlling for differences in countries, because those are included as dummies

head(Panel)
fixed.dum <-lm(y ~ x1 + factor(country) -1, data=Panel)
summary(fixed.dum)
```
Using the plm package

X1 = Average change across countries 
```{r}
install.packages("plm")
library(plm)
# For index need to have the fixed effects and the time variable identified and within means fixed effects
fixed = plm(y ~ x1, data = Panel, index = c("country", "year"), model = "within")
summary(fixed)
# Get the parameter estimates for the fixed effects
fixef(fixed)
```
Random effects model.  When the error term is related to the included fixed effects.  If differences across the time invariant variables are influecening the dependent variable then likely need a random effects model.  Whether indiviudal characitistic will influence the predictor variables.

X1 = Is the average effect of x on y when x changes across the unit in this case country over time.

So if countries do not have the same effect (cannot assume they are effecting the unit of change the predictor on y) then we need to have random effects, because in fixed effects we are assuming the effect of x on y is the same across units countries in this case.
```{r}
random = plm(y ~ x1, data = Panel, index = c("country", "year"), model = "random")

```
When to use fixed or random effects?

Use the Hausman Test, where the null hypothesis that the model is random effects and the alternative is that fixed, then we reject the null hypothesis.
```{r}
phtest(random, fixed)
```
Latent Growth Modeling.  Wanting to model growth over time.  
```{r}
library(lavaan)
head(Demo.growth)

# Specifcy two components random intercept (everyone get their own) and a random slope.  Slope starts at zero, because there is no slope at time one

i =~ 1*t1 + 1*t2 + 1*t3 + 1*t4
s =~ 0*t1 + 1*t2 + 2*t3 + 3*t4

model = "i =~ 1*t1 + 1*t2 + 1*t3 + 1*t4
s =~ 0*t1 + 1*t2 + 2*t3 + 3*t4"

fit = growth(model, data = Demo.growth)
summary(fit)

head(Demo.growth)
```


