---
layout: post
title: Time Series Inflation Forecasting Pt2
---

## ARMA(p,q) Modelling

Now that we know that inflation is stationary we can begin our modeling.


```R
Acf(infl, lag = 100, type = "correlation", lwd = 5, col = gray(0.6), main=expression(paste("ACF of: ",hat(epsilon)[t] )))
AutocorTest(infl, type="Box-Pierce",lag=100)
AutocorTest(infl, type="Ljung-Box",lag=100)
```





    	Box-Pierce test

    data:  infl
    X-squared = 3044.3, df = 100, p-value < 2.2e-16








    	Box-Ljung test

    data:  infl
    X-squared = 3160.8, df = 100, p-value < 2.2e-16





![svg]({{ site.url }}/images/output_8_2.svg)


The correlogram of inflation shows strong indication of autocorrelation, which is supported by Box-Pierce test and Ljung-Box test (in both tests, the p-value is less than 2.2e-16). This is consistent with our intuition that inflation tends to be a relatively persistent process, which means that current and past values should be helpful in forecasting future inflation.

So we start by building up simple ARMA models given that the inflation data is stationary.

#### The AR(1) Model:


```R
#########################  AR(1) #############################
# 1. Set up the exercise
p           = 1
q           = 0
f.period    = 80
h           = 1
RMSPE       = matrix(NA,1,3)
colnames(RMSPE) = c("ARMA(p,q)", "rw", "wn")
rownames(RMSPE) = "RMSPE"
f.error     = rep(NA, f.period-h)
infl.ar.spec = arfimaspec(mean.model = list(armaOrder = c(p, q), include.mean = TRUE),
                          distribution.model = "norm")
# 2. Iterations
cat("iteration: ")
for (i in 0:(f.period-h)) {
  cat(" ",i)
  infl.ar      = arfimafit(spec=infl.ar.spec, data=infl[1:(nrow(infl)-f.period+h+i),], out.sample=h, solver="nloptr")
  infl.f       = arfimaforecast(fitORspec=infl.ar, n.ahead = h, n.roll = 0, out.sample = h)
  f.error[i+1]  = infl[nrow(infl)-f.period+h+i,] - fitted(infl.f)[h,]
}

# 3. Compute the forecast performance measures
RMSPE[1]    = sqrt(mean(f.error^2))    ###ARMA(p,q)
RMSPE[2]    = sqrt(mean((infl[(nrow(infl)-(f.period-h)+1):nrow(infl),] - infl[(nrow(infl)-f.period+1):(nrow(infl)-h),])^2))
RMSPE[3]    = sqrt(mean(infl[(nrow(infl)-(f.period-h)+1):nrow(infl),]^2))

RMSPE
apply(RMSPE,1,which.min)

jb.test(residuals(infl.ar))  ## test residual normality
plot(density(residuals(infl.ar)))
```

    iteration:   0  1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79




<table>
<thead><tr><th></th><th scope=col>ARMA(p,q)</th><th scope=col>rw</th><th scope=col>wn</th></tr></thead>
<tbody>
	<tr><th scope=row>RMSPE</th><td>0.3084316</td><td>0.3425954</td><td>0.3806655</td></tr>
</tbody>
</table>







<strong>RMSPE:</strong> 1






<table>
<thead><tr><th></th><th scope=col>series 1</th></tr></thead>
<tbody>
	<tr><th scope=row>test stat</th><td>992.3047</td></tr>
	<tr><th scope=row>p-value</th><td>3.340158e-216</td></tr>
</tbody>
</table>





![svg]({{ site.url }}/images/output_10_4.svg)


In the above example we specified the model and then forcasted one period ahead. This process was repeated as the coefficients were recomputed and we performed a rolling forecast. At each repitition we reestimated coefficients and then forcasted one period ahead again.

We can see that this AR(1) model beat a random walk and a white noise process and produced a lower Root Mean Squared Prediction Error (RMSPE). Then we ran diagnostic tests on the model.

We will end our discussion of univariate models here and proceed to multivariate models.

### Vector Autoregressions (VARs)

All the univariate models struggle to forecast the sudden drop in inflation caused by the GFC. Perhaps a leading indicator such as housing starts could help forecast the drop. The GFC was caused by a housing crisis and was propagated through the use of mortgate backed securities.


```R
var.data<-readSeries("vardata.csv",sep=",",header=T,format="%d/%m/%Y")

another.var.data <- cbind(diff(log(var.data[,1])),var.data[,2],var.data[,9])

another.var.data <- another.var.data[complete.cases(another.var.data),]

###Splitting up sample

sample.data.var <- another.var.data[1:311,]
foreval.data.var <- another.var.data[312:nrow(another.var.data),]

###Running adf test on housing starts

ar(sample.data.var[,3], aic=T, order.max=15, method="ols")
adfTest(diff(sample.data.var[,3]), lags=13, type="c")

#Need to difference once to make stationary

#---------------------------
# Building the var


another.var.data.final <- cbind(diff(log(var.data[,1]))*100,var.data[,2],diff(var.data[,9]))

another.var.data.final<-another.var.data.final[complete.cases(another.var.data.final),]

plot(another.var.data.final)

#-------

ppilaglength<-24
all.lags.RMSPE<-matrix(NA,1,ppilaglength)
for (z in 1:ppilaglength){
  optimal.lags <- z
  forecast.error <- matrix(NA,1,80)
  forecast.pred <- matrix(NA,1,80)

  for (i in 1:80){
    y.var = VAR(y=another.var.data.final[(1:310+i),], p = optimal.lags, type = "const")
    y.var.f = predict(y.var, n.ahead=1)
    forecast.pred[,(i)] <- y.var.f$fcst$CPI...All.urban[1,1]
    forecast.error[,(i)] <- (another.var.data.final[(311+i),1]) - (y.var.f$fcst$CPI...All.urban[1,1])
  }

  RMSPE.err <- sqrt(mean(forecast.error^2))
  all.lags.RMSPE[z]<-RMSPE.err

}
all.lags.RMSPE
min(all.lags.RMSPE)
which.min(all.lags.RMSPE)
```





    Call:
    ar(x = sample.data.var[, 3], aic = T, order.max = 15, method = "ols")

    Coefficients:
          1        2        3        4        5        6        7        8  
     0.4762   0.2294   0.1811   0.1210  -0.0026   0.0990  -0.0102  -0.0646  
          9       10       11       12       13  
     0.0481  -0.0917   0.0083  -0.1302   0.0947  

    Intercept: -2.695 (5.553)

    Order selected 13  sigma^2 estimated as  9103



    Warning message:
    In adfTest(diff(sample.data.var[, 3]), lags = 13, type = "c"): p-value smaller than printed p-value





    Title:
     Augmented Dickey-Fuller Test

    Test Results:
      PARAMETER:
        Lag Order: 13
      STATISTIC:
        Dickey-Fuller: -4.1729
      P VALUE:
        0.01

    Description:
     Mon Jan 11 22:29:49 2016 by user:







<table>
<tbody>
	<tr><td>0.318491094343143</td><td>0.325921755880338</td><td>0.330656024438053</td><td>0.331999448485697</td><td>0.333938146706006</td><td>0.331809235950189</td><td>0.33478988574643 </td><td>0.339053275997145</td><td>0.339783424658625</td><td>0.345178186174702</td><td>⋯                </td><td>0.350373903301265</td><td>0.353896782968087</td><td>0.35658079544454 </td><td>0.358757442596357</td><td>0.35384358882712 </td><td>0.359960603535059</td><td>0.365831158247124</td><td>0.369094701183099</td><td>0.372333759307331</td><td>0.376025110542731</td></tr>
</tbody>
</table>







0.318491094343143






1




![svg]({{ site.url }}/images/output_13_6.svg)


The optimal lag length for the VAR is 1 and this produces a RMSPE of 0.318.
