MATH1318 Time Series Analysis - Assignment 1
================
Adam Pirsl S3815427





# Introduction

The ozone layer is a generic term used to describe the high
concentration of ozone that surrounds the earth. The ozone sits within
the stratosphere at a height of 15-30 km above the earths crust. The
ozone’s purpose is to absorb ultraviolet radiation from the sun that in
turn makes life of earth habitable by humans and other living organisms.

Over the last 100 years, humans have contributed to the destruction of
the ozone layer, largely through the release of chlorofluorocarbons that
are found in aerosol cans and refrigerants. Studies carried out by
United Nations Environmental Program indicate that current rates of
ozone depletion is unprecedented in the last 600 years and that
continual depletion of the ozone is contributing to global warming. This
has led scientists to question the impacts and damage caused by future
levels of ozone depletion.

# Aims and objectives

Ozone thickness is measured in Dobson units and has been recorded on a
yearly basis since 1927 to 2016. This analysis aims to build time series
models that will forecast the future state of the ozone layer over the
next 5 years from 2017 to 2021.

The data set will undergo a high level analysis of each time series
characteristic to better understand how it should be handled. Multiple
forecasting models will be built and assessed, including polynomial
models and ARIMA models.

# Methodology

## Import libraries and load the data set

``` r
# import libraries
library(readr)
library(dplyr)
library(tidyr)
library(magrittr)
library(TSA)
library(readxl)
library(zoo)
library(writexl)
library(tseries)
library(knitr)
library(latex2exp)
# opts_knit$set(eval.after = 'fig.cap')
# set wd
setwd("~/Documents/RMIT/TimeSeriesAnalysis/Assignment1")

# import data
ozone <- read_csv("data1.csv", col_names = FALSE)
```

## Fundamental Analysis

The data first needs to be converted to a time series object. This is
completed using the TSA package in R. The time series is object is
plotted in Figure @ref(fig:Fundamental-Concepts) below.

``` r
# convert to time series
ozone_1927_2016 <- ts(as.vector(ozone), start=1927, end=2016) 

# plot the time series
plot(ozone_1927_2016,
     type='o', 
     main ="Yearly Changes in Thickness of Ozone Layer",
     ylab = "Change in Dobson units",
     xlab = "Years (1927 - 2016)")
```

![Ozone changes in Dobson
units](Assignment1_files/figure-gfm/Fundamental-Concepts-1.png)

**Behavior:** There is both autoregressive (AR) and moving average (MA)
characteristics withing the time series. There are multiple data points
contributing to the upward and downward shift of the time series (AR)
such as at approximately 1937 to 1940. There are also characteristics of
random movement (MA), however it is not as dominant as the AR behaviour.

**Changing variance:** There are no obvious signs of changing variance.

**Trend:**There is a downward trend.

**Seasonality:** There is no distinct seasonality between equal sets of
years.

**Change point:** There is a small change point at approximately 1995.
This relatively extreme movement is out of proportion compared to other
year on year movements.

## Polynomial Regression

### Linear Regression

``` r
# Fit the linear model
t = time(ozone_1927_2016)
model1 = lm(ozone_1927_2016~t)
summary(model1)
```

    ## 
    ## Call:
    ## lm(formula = ozone_1927_2016 ~ t)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -4.7165 -1.6687  0.0275  1.4726  4.7940 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 213.720155  16.257158   13.15   <2e-16 ***
    ## t            -0.110029   0.008245  -13.34   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 2.032 on 88 degrees of freedom
    ## Multiple R-squared:  0.6693, Adjusted R-squared:  0.6655 
    ## F-statistic: 178.1 on 1 and 88 DF,  p-value: < 2.2e-16

``` r
model1
```

    ## 
    ## Call:
    ## lm(formula = ozone_1927_2016 ~ t)
    ## 
    ## Coefficients:
    ## (Intercept)            t  
    ##      213.72        -0.11

``` r
# fit the linear model
plot(ozone_1927_2016,type='o',
     ylab='Change in Dobson units', 
     main = "Fitted Linear Model to Ozone Series",
     xlab = "Years (1927 - 2016)")
abline(model1,type="l",lty=2,col="red")
```

![Fitted linear model](Assignment1_files/figure-gfm/linear-model-1.png)

### Key Linear Model Statistics

1.  R squared value is 0.6655, meaning that the model is not overly
    fitted and there is a some degree of variability in the dependent
    variable within the model.
2.  P-value is less than 0.05 at &lt; 2.2e-16, meaning the linear trend
    is statistically significant.

### Residual Analysis

``` r
# Residual analysis
res.model1 = rstudent(model1)
par(mfrow=c(3,2),mar=c(7,7,7,7))

res.model1 = rstudent(model1)
plot(y = res.model1, 
     x = as.vector(time(ozone_1927_2016)),
     xlab = 'Time', 
     ylab='Standardized Residuals',type='l',
     main = "Standardised Residuals from Linear Model."
     )

hist(res.model1,
     xlab='Standardised Residuals', 
     main = "Histogram of Standardised Residuals.")


qqnorm(y=res.model1, 
       main = "QQ plot of Standardised Residuals.")

qqline(y=res.model1, col = 2, lwd = 1, lty = 2)

acf(res.model1, main = "ACF of Standardized Residuals.")

pacf(res.model1, main = "PACF of Standardized Residuals.")

par(mfrow=c(1,1))
```

![Residual
analysis](Assignment1_files/figure-gfm/linear-model-residuals-1.png)

``` r
shapiro.test(res.model1)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  res.model1
    ## W = 0.98733, p-value = 0.5372

**1. Standardised residuals**  

There are no residuals that exceed 3 or -3 on the normal standardised
residual graph. The movement does not appear to be random.

**2. Histogram of standardised residuals**  

The histogram appears mostly uniform, although the left side is slopped
more evenly compared to the right tail that drops of suddenly.
Uniformity in the histogram suggests the model fits the linear trend.

**3. QQ Plot**  

The straight line pattern along the reference line supports the
hypothesis of a normally distributed stochastic component in this model.
Higher departure from the reference line would mean the linear model
does not fit the trend. At either end of the plot, the residuals peel
off from the reference line, indicating there is some changing variance
of the standardised residuals.

**4. ACF**  

The ACF plot confirms there is auto correlation as there are correlation
values higher than the confidence bounds at lag 1 as well as 2 which is
slightly significant. This outcome would not be expected from a white
noise process.

**5. PACF**  

PACF is the correlation of the standardised residuals. It is intended to
identify hidden information in the residual which can be modeled by the
next lag. There is significant auto correlation of the standardised
residuals at lags 1 and 2 as well as a slightly significant correlation
at lag 3.

**6. Shapiro test**  
p-value = 0.5372

The p value (normality of residuals) &lt; 0.05 is significant therefore
the residuals follow a normal distribution. A model that has residuals
that does not have a normal distribution will not be a good model to
choose.

### Quadratic Regression

``` r
# Fit the quadratic model
t = time(ozone_1927_2016)
t2 = t^2
model2 = lm(ozone_1927_2016~ t + t2)
summary(model2)
```

    ## 
    ## Call:
    ## lm(formula = ozone_1927_2016 ~ t + t2)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -5.1062 -1.2846 -0.0055  1.3379  4.2325 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -5.733e+03  1.232e+03  -4.654 1.16e-05 ***
    ## t            5.924e+00  1.250e+00   4.739 8.30e-06 ***
    ## t2          -1.530e-03  3.170e-04  -4.827 5.87e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.815 on 87 degrees of freedom
    ## Multiple R-squared:  0.7391, Adjusted R-squared:  0.7331 
    ## F-statistic: 123.3 on 2 and 87 DF,  p-value: < 2.2e-16

``` r
plot(ts(fitted(model2)), ylim = c(min(c(fitted(model2), as.vector(ozone_1927_2016))), max(c(fitted(model2),as.vector(ozone_1927_2016)))),
     ylab='Change in Dobson units' , 
     xlab = "Years (1927 - 2016)",
     main = "Fitted Quadratic Curve to Ozone Series", 
     type="l",lty=2,
     col="red")
lines(as.vector(ozone_1927_2016),type="o")
```

![Fitted quadratic
model](Assignment1_files/figure-gfm/quadratic-model-1.png)

### Key Quadratic Model Statistics

1.  R squared value of 0.7331 indicates the model fits the time series
    without being overly fitted.
2.  P-value statistic of &lt; 2.2e-16 shows the model is statistically
    significant.

### Residual Analysis

``` r
# Residual analysisÍ›
res.model2 = rstudent(model2)
par(mfrow=c(3,2))

plot(y = res.model2, 
     x = as.vector(time(ozone_1927_2016)),
     xlab = 'Time', 
     ylab='Standardized Residuals',
     type='l',
     main = "Standardised Residuals from Quadratic Model.")

hist(res.model2,xlab='Standardized Residuals', 
     main = "Histogram of Standardised Residuals.")

qqnorm(y=res.model2, 
       main = "QQ Plot of Standardised Residuals.")

qqline(y=res.model2, col = 2, lwd = 1, lty = 2)

acf(res.model2, main = "ACF of Standardized Residuals.")

pacf(res.model2, main = "PACF of Standardised Residuals.")
par(mfrow=c(1,1))
```

![Quadratic Residual
Analysis](Assignment1_files/figure-gfm/quadratic-model-residuals-1.png)

``` r
shapiro.test(res.model2)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  res.model2
    ## W = 0.98889, p-value = 0.6493

**1. Standardised residuals**  

Most standardised residuals fall within the range of 3 and -3 showing
normality of the model. Although, there is one residual at approximately
1995 that has a value of -3. There is little variance and the movement
of the graph is gradually up and down, emphasising the AR traits of the
model.

**2. QQ Plot**  

The mostly straight line pattern supports the assumption of a normally
distributed stochastic components in this model, although the ends do
depart from the normal line.

**3. Histogram of standardised residuals**  

The histogram is not symmetrical and is not evenly distributed. There
are many residuals at -1 compared to the other positions causing the
distribution not to follow a normal curve. Residuals below -1 are very
small and residuals after 0 remain at high frequency before dropping off
at 1.5. The Shapiro Wilk test will help to confirm the normality.

**4. ACF**  

The ACF plot confirms there is auto correlation as there are correlation
values higher than the confidence bounds at lag 1, 3, 4 and also 5 which
is slightly significant. This outcome would not be expected from a white
noise process.

**5. PACF**  

There is significant auto correlation of the standardised residuals at
lags 1, 2 and 3.

**6. Shapiro test**  

6.  Shapiro test p-value = 0.6493

The p value (normality of residuals) is greater that 0.05 therefore we
fail to reject the null hypothesis. The test confirms what is seen in
the histogram of residuals that the residuals quadratic model does not
follow a normal distribution.

## Model evaluation and selection for forecasting purposes

The linear and quadratic model present strengths and weaknesses. The
linear model has a lower r-squared value of 0.6655 vs the quadratic
r-squared value of 0.7331, meaning that is the less fitted model. Both
models have the same p value, showing that they are both statistically
significant.

The quadratic model has less even distribution of residuals (histogram),
however it does capture a more static version of the standardised
residuals compared to the linear model. The ACF and PACF plots of both
models show strong autocorrelation at lag 1.

Given these models are similar, the Shapiro-Wilk (SW) test will confirm
that the linear model is the better option with a lower p-value of less
than 0.05. The SW test, combined with the lower r-squared value further
strengthens the linear models viability as an accurate model (it is not
ideal to have a model that is over fitted).

### Linear Forecast

``` r
h = 5 # 5 year forecast
t = time(ozone_1927_2016)

ahead = data.frame(t = seq(2017,2021, 1))

fcModel1 = predict(model1, newdata  = ahead, interval = "prediction")

print(fcModel1)
```

    ##         fit       lwr       upr
    ## 1 -8.208590 -12.33732 -4.079864
    ## 2 -8.318619 -12.45033 -4.186902
    ## 3 -8.428648 -12.56342 -4.293878
    ## 4 -8.538677 -12.67656 -4.400792
    ## 5 -8.648706 -12.78977 -4.507643

``` r
plot(ozone_1927_2016, xlim = c(1920,2021), ylim = c(-15, 10), ylab = "Changes in Dobson units", main = "Linear Forecast Model of Ozone Data", xlab = "Years (1927 - 2021)")
lines(ts(as.vector(fcModel1[,1]), start = 2017), col="green", type="l") 
lines(ts(as.vector(fcModel1[,2]), start = 2017), col="orange", type="l") 
lines(ts(as.vector(fcModel1[,3]), start = 2017), col="orange", type="l") 
legend("topleft", lty=1, pch=1, col=c("orange","green","orange"), text.width = 15, c("Upper Confidence Int.","Forecast", "Lower Confidence Int."))
```

![Linear Model
Forecast](Assignment1_files/figure-gfm/linear-model-forecast-1.png)

## 3 ARIMA Analysis

The Ozone time series plot displays a downward trend. This means the
time series is likely non-stationary and it will need to undergo
differencing to make it stationary. This process will use ARIMA.

ARIMA(p,d,q)

-P is read from PACF plot (MA\[p\]) -Q is read from ACF plot (AR\[q\])

### Testing for Changeing Variance

The time series graph does not show obvious changing variance, although
variance can still be tested for using a box cox transformation and QQ
plot. If there is any variance, it will need to be removed before any
differencing procedures are applied.

The QQ plot below represents the original data set and not the residuals
that were produced in previous sections.

``` r
# qq plot to assess normality
qqnorm(ozone_1927_2016, main = "QQ Plot of Ozone Data")
qqline(ozone_1927_2016, col = 2)
```

![QQ Plot of time
series](Assignment1_files/figure-gfm/testing-variance-qq1-1.png)

``` r
shapiro.test(ozone_1927_2016)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  ozone_1927_2016
    ## W = 0.95605, p-value = 0.004031

The QQ plot will help identify whether a time series plot has a normal
distribution. If the dots move away from the red line, then it means
there could be variance in the time series. The closer to the red line
means less variance in the time series.

The original plot did not appear to have changing variance, although the
QQ plot indicates there could be some variance by the diverging data
points from the reference line. A box cox transformation may remove the
variance if there is any.

``` r
# Change negative values to positive (Shift the series)
BC = BoxCox.ar(ozone_1927_2016 + abs(min(ozone_1927_2016))+1, lambda = seq(0, 2, 0.1))
```

![QQ Plot of time
series](Assignment1_files/figure-gfm/testing-variance-bc1-1.png)

``` r
# Obtain the lambda function
lambda <- BC$lambda[which(max(BC$loglike) == BC$loglike)]
print(paste0("The value of lambda is : ", lambda))
```

    ## [1] "The value of lambda is : 1.2"

For the box cox transformation, a lambda value of 1 is the same as using
the original data. Therefore, if the confidence interval for the optimal
lambda includes 1, then no transformation is needed. In this case, the
value of lambda is 1.2, meaning that the plot will look similar to the
original after transforming the data and we do not need to apply the
transformations as there is only a small variance.

Lamba values: -1. is a reciprocal -.5 is a recriprocal square root 0.0
is a log transformation .5 is a square toot transform and 1.0 is no
transform.

**Comparison of the original time series to the box cox transformed
series:**

``` r
# Compare the box cox transforamtion to the original series
par(mfrow=c(1,2))
ozone_1927_2016_shifted = ozone_1927_2016 + abs(min(ozone_1927_2016))+1
BC.ozone_1927_2016 = (ozone_1927_2016_shifted^lambda-1)/lambda  

plot(BC.ozone_1927_2016,type='o',
     ylab='Change in Dobson units',
     main ="BC Transformed Ozone Series")

plot(ozone_1927_2016,
     type='o',
     ylab='Change in Dobson units',
     main ="Original Ozone Series")
```

![BC Plot of time series
comparison](Assignment1_files/figure-gfm/testing-variance-bc2-1.png)

``` r
par(mfrow=c(1,1)) 
```

**Comparison of the original QQ plot to the box cox transformed QQ
plot:**

``` r
# Test the normality using qq plot. Compare it against the initial qq plot.

shapiro.test(BC.ozone_1927_2016)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  BC.ozone_1927_2016
    ## W = 0.96644, p-value = 0.01995

``` r
shapiro.test(ozone_1927_2016)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  ozone_1927_2016
    ## W = 0.95605, p-value = 0.004031

``` r
par(mfrow=c(1,2))

qqnorm(BC.ozone_1927_2016, main = "QQ Plot of BC Transformed Data")
qqline(BC.ozone_1927_2016, col = 2)

qqnorm(ozone_1927_2016, main = "QQ Plot Original Data")
qqline(ozone_1927_2016, col = 2)
```

![QQ Plot
comparison](Assignment1_files/figure-gfm/testing-variance-qq-1.png)

``` r
par(mfrow=c(1,1)) 
```

Insights gained: - The shape of the transformed series is similar to the
original - The original QQ plot looks similar to the box cox transformed
QQ plot - Shapiro Wilk test show the original and transformed p-value as
less than 0.05 - Lambda function is equal to 1.2,, meaning that a
transformation is not necessary

For these reasons, a box cox transformation will not improve the results
of the model as there is very little changing variance in the original
time series. The final characteristic to test for is stationarity.

### Testing for Stationarity

The time series plot shows a downward trend. We can use some tools to
test the hypothesis (existence of a trend) such as ACF, Dicker-Fuller
Unit Root Test and Phillips-Perron (PP) test. If the trend is
stationary, an ARMA model can be used. If there is a trend, then a ARIMA
model will be used.

``` r
# ACf and PACF
par(mfrow=c(2,1),mar=c(3,3,3,3))
acf(ozone_1927_2016, main ="ACF Plot of Ozone Time Series")
pacf(ozone_1927_2016, main ="PACF Plot of Ozone Time Series")
```

![ACF and PACF
analysis](Assignment1_files/figure-gfm/testing-stationarity-qq-1.png)

``` r
par(mfrow=c(1,1))

# Dicker Fuller unit root test
adf.test(ozone_1927_2016, alternative = c("stationary"))
```

    ## 
    ##  Augmented Dickey-Fuller Test
    ## 
    ## data:  ozone_1927_2016
    ## Dickey-Fuller = -3.2376, Lag order = 4, p-value = 0.0867
    ## alternative hypothesis: stationary

``` r
adf.test(ozone_1927_2016, k=3,alternative = c("stationary"))  
```

    ## 
    ##  Augmented Dickey-Fuller Test
    ## 
    ## data:  ozone_1927_2016
    ## Dickey-Fuller = -3.9376, Lag order = 3, p-value = 0.01596
    ## alternative hypothesis: stationary

``` r
adf.test(ozone_1927_2016, k=5,alternative = c("stationary"))  
```

    ## 
    ##  Augmented Dickey-Fuller Test
    ## 
    ## data:  ozone_1927_2016
    ## Dickey-Fuller = -2.5704, Lag order = 5, p-value = 0.3413
    ## alternative hypothesis: stationary

``` r
# Phillips-Perron test
pp.test(ozone_1927_2016, alternative = c("stationary"))
```

    ## 
    ##  Phillips-Perron Unit Root Test
    ## 
    ## data:  ozone_1927_2016
    ## Dickey-Fuller Z(alpha) = -35.431, Truncation lag parameter = 3, p-value
    ## = 0.01
    ## alternative hypothesis: stationary

ACF: The ACF plot is displaying a decaying pattern, further highlighting
that the time series has a trend and is non-stationary. The number of
lags between each peak is the same, indicating there may be seasonality
in the series.

Dicker Fuller test: At k = 4, the p-value = 0.0867 which is slightly
above the limit of 0.05. Given the proximity to 0.05, the K lag order of
3 and 5 are tested as well, with k = 3 producing a p-value of 0.01596
and k = 5 producing a p-value of 0.3413. K = 3 is withing the p-value
limits of stationarity.

Phillips-Perron test: The p-value is 0.01 indicating non-stationarity.

Despite the Dicker Fuller test showing some uncertainty around p-value
testing for non-stationarity, we can use the decaying pattern of the ACF
plot and p-value of the Phillips-Perron test to conclude that the plot
is in fact non-stationary and will require differencing.

### 3.3 Differencing (d)

**Difference the time series to remove the trend and analyse the ACF and
PACF:**

``` r
# Difference the data
diff.ozone_1927_2016 = diff(ozone_1927_2016,differences=1)
plot(diff.ozone_1927_2016,type='o',ylab='Ozone metrics', main ="Differenced Ozone Data")
```

![Differenced data set (d =
1)](Assignment1_files/figure-gfm/differencing-1.png)

``` r
# test for stationarity
adf.test(diff.ozone_1927_2016, alternative = c("stationary"))
```

    ## 
    ##  Augmented Dickey-Fuller Test
    ## 
    ## data:  diff.ozone_1927_2016
    ## Dickey-Fuller = -7.1568, Lag order = 4, p-value = 0.01
    ## alternative hypothesis: stationary

``` r
pp.test(diff.ozone_1927_2016, alternative = c("stationary"))
```

    ## 
    ##  Phillips-Perron Unit Root Test
    ## 
    ## data:  diff.ozone_1927_2016
    ## Dickey-Fuller Z(alpha) = -70.244, Truncation lag parameter = 3, p-value
    ## = 0.01
    ## alternative hypothesis: stationary

After differencing the time series once, the p-value of adf and pp test
are less than 0.05. The time series is now stationary (d = 1).

### AR and MA values

Multiple methods are used to find the optimal p and q values for the
model such as ACF/PACF plots, EACF plot and BIC plot.

**Method 1: ACF / PACF**  

``` r
par(mfrow=c(1,2),mar=c(3,3,3,3))
acf(diff.ozone_1927_2016, main="ACF of Differenced (d=1) Ozone Data")
pacf(diff.ozone_1927_2016, main="PACF of Differenced (d=1) Ozone Data")
```

![ACF and PACF analysis](Assignment1_files/figure-gfm/acf-pacf-1.png)

``` r
par(mfrow=c(1,1))
```

ACF(q) = 1,2 There is one strong lag at position 3 and one slightly
significant lag at position 4.

PACF(p) = 1,2 There is one strong lag at position 3 and one significant
lag at position 4.

ARIMA models using ACF/PACF:

ARIMA(p,d,q)

{ARIMA(1,1,1) ARIMA(1,1,2) ARIMA(2,1,1) ARIMA(2,1,2)}

**Method 2: EACF Plots**  

``` r
eacf(diff.ozone_1927_2016)
```

    ## AR/MA
    ##   0 1 2 3 4 5 6 7 8 9 10 11 12 13
    ## 0 o o x o o o x o o x o  o  o  o 
    ## 1 x o x o o o o o o x o  o  o  o 
    ## 2 x o x o o o x o o x o  o  o  o 
    ## 3 x o x o o x o o o o o  o  o  o 
    ## 4 x o o x o x o o o o o  o  o  o 
    ## 5 x x x x o x o o o o o  o  o  o 
    ## 6 o o o x x o o o o o o  o  o  o 
    ## 7 o o o x o o o o o o o  o  o  o

The EACF plot displays values for q across the top and values for p on
the vertical axis. The vertex occurs at position q = 3 and p = 0.

ARIMA models using EACF:

ARIMA(p,d,q)

{ARIMA(0,1,3) ARIMA(0,1,4) ARIMA(1,1,4)}

**Method 3: BIC Plots**  

``` r
res = armasubsets(y=diff.ozone_1927_2016 , nar=5 , nma=5, y.name='Ozone', ar.method='ols')
plot(res)
```

![BIC analysis](Assignment1_files/figure-gfm/BIC-1.png)

p is only the left side and q is on the right side (denoted by ‘error’).

p value is equal to 3. q value is equal to 0 because there are no strong
vertical bars or colors that reach the minimum BIC. Error-lag2 might be
considered as slightly significant if the bar underneath it was more
complete.

ARIMA models using BIC:

ARIMA(p,d,q)

{ARIMA(3,1,0)}

### ARIMA Model Outcomes

There are a total of 8 ARIMA models derived from ACF/PACF, EACF, BIC
methods:  

{ARIMA(1,1,1) ARIMA(1,1,2) ARIMA(2,1,1) ARIMA(2,1,2) ARIMA(0,1,3)
ARIMA(0,1,4) ARIMA(1,1,4) ARIMA(3,1,0)}

There is a tendency toward lower p values (1,2) and mid to high q values
(3,4) for the this time series ARIMA model.

# Conclusion

Time series models contain subjectivity in their approach. By using
statistical tools and processes, it is possible to derive multiple time
series models for a single data set. Within the scope of this
investigation, a simple linear model is able to forecast the next 5
years of Ozone level changes in dobson units. ARIMA models are also
useful, although require more work to remove variance, understand
stationarity and finally to derive inputs for p and q that are likely to
produce good forecasts.
