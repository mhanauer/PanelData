---
title: "Panel Data Examples using R"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(
	echo = TRUE,
	message = FALSE,
	warning = FALSE
)
```
Here is an example of three different methods (fixed effects, random effects, and latent growth modeling) for analyzing longitudinal data using artificial data sets.  First, we will demonstrate how to create a longitudinal data set.  We create an artificial data set with two variables a1 and a2.  Let us assume that a1 and a2 are scores on some measurement across time and we need to create a longitudinal data set that stacks a1 on top of a2 into a single variable with a time variable that identifies the time point for each observation.  To do this we can use the reshape package in R.  

There are five components that we need to complete to create the longitudinal data set.  First, we identify the dataset that we are using, dataLong in this case.  Then select the variables that we want to stack using the varying command and identifying the a1 and a2 variables.  Then we create a name for the new stacked variable, which we called dependent.  Then we select the time points that will be associated with each of the variables in a variable called time, which is one and two in this example.  Finally, we select the direction, which is long, because we want a longitudinal data set.
    
```{r, message=FALSE, warning=FALSE}
library(reshape)

dataLong = data.frame(a1 = c(rnorm(10)), a2 = c(rnorm(10)))
dataLongLong = reshape(dataLong, varying = list(c("a1", "a2")), v.names = "dependent", times = c(1,2), direction = "long")

head(dataLongLong)
```
The fixed model includes time variant variables that suck out variation associated with steady fixtures such as schools or a time variable that captures variation in the dependent variable associated with these fixtures.  In this example, below we look a data set of countries over time in years across an artificial dependent variable y and artificial x variable x1.  To create the fixed effects model, we use the plm function in R.  To run the PLM function in R, we create the model, which is y regressed on x1, create the index variables and set the model.  The index variables are the fixed effects that we want to create, so in this case we have two sets of fixed effects countries and years.  The model function lets us tell R whether to use fixed or random effects, where in this case within means fixed effects.  Then we can get the parameter estimate for x1, which indicates that average amount of change per country over time given a unit increase in x.  
```{r, message=FALSE, warning=FALSE}
library(foreign)
library(plm)
Panel <-read.dta("http://dss.princeton.edu/training/Panel101.dta")
head(Panel)

fixed = plm(y ~ x1, data = Panel, index = c("country", "year"), model = "within")
summary(fixed)
```
In a random effects model, we do not assume that the effects of the time invariant variables, such as school and country, are the same and allow them to have their own starting values (i.e. intercepts).  In this model, the R code is almost identical, except that in the model section we change within to random.  Then we get the value of the parameter estimate, which is the average change in y over time between countries.  
```{r, message=FALSE, warning=FALSE}
random = plm(y ~ x1, data = Panel, index = c("country", "year"), model = "random")
summary(random)
```
How do we know when to use fixed or random effects?  We can use the Hausman Test, where the null hypothesis is that the model is random effects and the alternative is that fixed effects are better fit.  In this example, the p-value is above .05 so a random effects model is the better fit for this data.
```{r, message=FALSE, warning=FALSE}
phtest(random, fixed)
```
Another way to analyze longitudinal data is through Latent Growth Modeling (LGM). LGM models time as a latent variable similar to a multi-level modeling approach.  To analyze LGM in R we use the lavaan package.  To analyze LGM in lavaan we use an artificial data set provided by lavaan.  The first step is to define the model, which is enclosed in quotations and sent as a string.  In this case, we are fixing parameter estimates for time to be one to ensure the model is identified.  Then we set each of the slope parameter estimates to their respective time point with one change.  Since we cannot estimate the slope of time point one, we set that to zero and use n-1 time points, in this case three to identify the slopes.  Variances for the time points, intercept, slope, and covariance between the slope and variance are freely estimated.    

Then we can use the growth function in lavaan to correctly analyze the data as a LGM.  In this example I am focusing on the interpretation of parameters not the model fit, so I will skip the first section regarding model fit.  The first section is the parameter estimates, we fixed in the model.  The next section is the covariance between the intercept and the slope demonstrating the relationship between the starting values and the slope or change over time.  To fully interpret this value, we need to analyze the slope value found in the intercept section of the output.  If the slope is positive and the covariance is positive, as is the case in our example, then we can say higher starting values (i.e. intercepts) leads to larger slopes or increases in y relative to lower starting values.  Then we can move the intercepts, which is what we are interested in.  "i" is the mean of the starting value at time one and "s" is the average change in y over time.  We can see that the slope is statistically significantly different from zero meaning that there is a significant change in y over time.
```{r, message=FALSE, warning=FALSE}
library(lavaan)
head(Demo.growth)

model = "i =~ 1*t1 + 1*t2 + 1*t3 + 1*t4
s =~ 0*t1 + 1*t2 + 2*t3 + 3*t4"

fit = growth(model, data = Demo.growth)
summary(fit)
```


