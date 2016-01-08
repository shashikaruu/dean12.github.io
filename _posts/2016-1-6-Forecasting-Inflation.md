---
layout: post
title: Time Series Inflation Forecasting
---


## Part 1: Stationarity and Structural Breaks

#### Time series forecasting of inflation figures using ARIMA, Vector Error Correction and Vector Autoregression techniques.

We will forecast the U.S. inflation rate, constructed as the log-returns of the U.S. Consumer Price Index (CPI) for All Urban Consumers.
Different forecasting models will be constructed and then we will perform a rolling sample forecasting procedure for a one-month-ahead inflation forecast over the last 80 months of the sample period. To compare and evaluate the accuracy of different specifications, we compute the root mean squared error (RMSE) for the predictions.

In part 1 we test for unit roots and structural breaks.


```R
library(fImport); library(timeDate); library(FinTS); library(ccgarch); library(rugarch);
library(nloptr); library(vars); library(fUnitRoots); library(strucchange);

```


```R

# First we plot the data
cpi=read.csv("uscpi.csv", header=T, dec=",", sep=";")
plot(cpi[,2], type="l",lwd=2, col="gray", main="US cpi")

dd=data.frame(date = seq(as.Date('1947/02/01'), by = 'month', length.out = 811),
                usinflation = 100*diff(log(cpi[,2])))

infl= ts(as.data.frame(dd$usinflation), names="US inflation")
par(mfrow=c(1,1))                        
plot.ts(infl, lwd=3, col="gray", main="US inflation")

```


![svg]({{ site.url }}/_posts/output_2_0.svg)



![svg]({{ site.url }}/_posts/output_2_1.svg)


#### Structural Break Tests
Structural breaks are a possibility in the data on inflation. The 1970’s oil shock as well as the early 1980’s recession caused by the US Federal Reserve choosing to keep inflation under control even when the economy was experiencing downward pressure are taken into consideration in the models built.


We run the test for structural break, and find two breaking points where the intercept of inflation shifts. They are at May 1967, Aug 1982 respectively. Therefore, we might want to include structural breaks in our forecasting models.


```R
fs.infl=Fstats(as.ts(infl[1:729]) ~ 1)
plot(fs.infl)
sctest(fs.infl)
plot.ts(infl)
breakpoints(fs.infl)  ## obs. 244
dd[244,]   ##   [1967-05-01,0]

infl.after=infl[244:811,]
fs.infl.after=Fstats(as.ts(infl.after) ~ 1)
plot(fs.infl.after)
sctest(fs.infl.after)
plot.ts(infl.after)
breakpoints(fs.infl.after)  ## obs. 183
dd[427,]   ##   [1982-08-01,0.2049181]
```





    	supF test

    data:  fs.infl
    sup.F = 61.447, p-value = 2.727e-13





![svg]({{ site.url }}/_posts/output_4_1.svg)






    	 Optimal 2-segment partition:

    Call:
    breakpoints.Fstats(obj = fs.infl)

    Breakpoints at observation number:
    244

    Corresponding to breakdates:
    244






<table>
<thead><tr><th></th><th scope=col>date</th><th scope=col>usinflation</th></tr></thead>
<tbody>
	<tr><th scope=row>244</th><td>1967-05-01</td><td>0</td></tr>
</tbody>
</table>





![svg]({{ site.url }}/_posts/output_4_4.svg)






    	supF test

    data:  fs.infl.after
    sup.F = 206.63, p-value < 2.2e-16





![svg]({{ site.url }}/_posts/output_4_6.svg)






    	 Optimal 2-segment partition:

    Call:
    breakpoints.Fstats(obj = fs.infl.after)

    Breakpoints at observation number:
    183

    Corresponding to breakdates:
    183






<table>
<thead><tr><th></th><th scope=col>date</th><th scope=col>usinflation</th></tr></thead>
<tbody>
	<tr><th scope=row>427</th><td>1982-08-01</td><td>0.2049181</td></tr>
</tbody>
</table>





![svg]({{ site.url }}/_posts/output_4_9.svg)


#### Stationarity and Unit Roots

We also investigate the stationarity of our inflation data. We use Augmented Dickey–Fuller (ADF) unit root test and find the test results depend on the specification of lag length. Shorter maximum length of lags gives small p-value, which suggests no unit root, i.e. stationary, for example, for lag order equals 20, the p-value is 0.01, which is less than the significant level 5%.

While longer maximum length of lags gives large p-value suggesting the series has at least one unit root, for example, for lag order equals 50, the p-value is 0.2758, which is larger than the significant level 5%. This is because if the lag length is too large, the power of the test will suffer. Given the structural break we find, the ADF test results might not be valid. Therefore, we consider Phillips and Perron test for unit roots.

Using the short version of the lag parameter (=6), we get a p-value equal to 0.01, which suggests no unit root at significance level of 5%. So we conclude that the inflation is stationary.


```R
adfTest(cpi[,2], lags=20, type="ct")  # cpi has at least one unit root
adfTest(infl, lags=50, type="c")     # infl has no unit root, stationary

PP.test(infl, lshort = TRUE)  # infl has no unit root, stationary -- this is the preferred test
```





    Title:
     Augmented Dickey-Fuller Test

    Test Results:
      PARAMETER:
        Lag Order: 20
      STATISTIC:
        Dickey-Fuller: -2.5558
      P VALUE:
        0.3431

    Description:
     Fri Jan  8 14:08:40 2016 by user:








    Title:
     Augmented Dickey-Fuller Test

    Test Results:
      PARAMETER:
        Lag Order: 50
      STATISTIC:
        Dickey-Fuller: -2.0997
      P VALUE:
        0.2758

    Description:
     Fri Jan  8 14:08:40 2016 by user:








    	Phillips-Perron Unit Root Test

    data:  infl
    Dickey-Fuller = -15.4, Truncation lag parameter = 6, p-value = 0.01
