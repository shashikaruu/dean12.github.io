---
layout: post
title: Logistic Regressions and Breast Cancer
---

Using R to train a classifier to separate between malignant and benign tumors.

#### Examining the data

Data come from a study of breast cancer in Wisconsin. There are 681 cases of potentially cancerous tumors of which 238 are actually malignant. Determining whether a tumor is really malignant is traditionally determined by an invasive surgical procedure. The purpose of this study was to determine whether a new procedure called fine needle aspiration which draws only a small sample of tissue could be effective in determining tumor status.  


---
There are in total 10 variables:

- Class: 0 if malignant, 1 if benign

- Adhes: marginal adhesion

- BNucl: bare nuclei

- Chrom: bland chromatin

- Epith: epithelial cell size

- Mitos: mitoses

- NNucl: normal nucleoli

- Thick: clump thickness

- UShap: cell shape uniformity

- USize: cell size uniformity

The predictor values are determined by a doctor observing the cells and rating them on a scale from 1 (normal) to 10 (most abnormal) with respect to the particular characteristic.

```R
> head(wbca)
  Class Adhes BNucl Chrom Epith Mitos NNucl Thick UShap USize
1     1     1     1     3     2     1     1     5     1     1
2     1     5    10     3     7     1     2     5     4     4
3     1     1     2     3     2     1     1     3     1     1
4     1     1     4     3     3     1     7     6     8     8
5     1     3     1     3     2     1     1     4     1     1
6     0     8    10     9     7     1     7     8    10    10
```
---
Lets run a basic logistic regression with all explanatory variables using the glm package in R.

```R
> allvar.logistic <-  glm(Class ~ Adhes + BNucl + Chrom + Epith + Mitos + NNucl + Thick + UShap + USize, family = binomial(link = 'logit'),wbca)
> summary(allvar.logistic)

Call:
glm(formula = Class ~ Adhes + BNucl + Chrom + Epith + Mitos +
    NNucl + Thick + UShap + USize, family = binomial(link = "logit"),
    data = wbca)

Deviance Residuals:
     Min        1Q    Median        3Q       Max  
-2.48282  -0.01179   0.04739   0.09678   3.06425  

Coefficients:
            Estimate Std. Error z value Pr(>|z|)
(Intercept) 11.16678    1.41491   7.892 2.97e-15 ***
Adhes       -0.39681    0.13384  -2.965  0.00303 **
BNucl       -0.41478    0.10230  -4.055 5.02e-05 ***
Chrom       -0.56456    0.18728  -3.014  0.00257 **
Epith       -0.06440    0.16595  -0.388  0.69795
Mitos       -0.65713    0.36764  -1.787  0.07387 .  
NNucl       -0.28659    0.12620  -2.271  0.02315 *  
Thick       -0.62675    0.15890  -3.944 8.01e-05 ***
UShap       -0.28011    0.25235  -1.110  0.26699
USize        0.05718    0.23271   0.246  0.80589
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 881.388  on 680  degrees of freedom
Residual deviance:  89.464  on 671  degrees of freedom
AIC: 109.46

Number of Fisher Scoring iterations: 8
```
---
####Can we build a better model by using fewer variables?

The step function in R lets us choose the variables that maximize AIC.

```R
> minaic.logistic <- step(allvar.logistic, scope = ~.)
Start:  AIC=109.46
Class ~ Adhes + BNucl + Chrom + Epith + Mitos + NNucl + Thick +
    UShap + USize

        Df Deviance    AIC
- USize  1   89.523 107.52
- Epith  1   89.613 107.61
- UShap  1   90.627 108.63
<none>       89.464 109.46
- Mitos  1   93.551 111.55
- NNucl  1   95.204 113.20
- Adhes  1   98.844 116.84
- Chrom  1   99.841 117.84
- BNucl  1  109.000 127.00
- Thick  1  110.239 128.24

Step:  AIC=107.52
Class ~ Adhes + BNucl + Chrom + Epith + Mitos + NNucl + Thick +
    UShap

        Df Deviance    AIC
- Epith  1   89.662 105.66
- UShap  1   91.355 107.36
<none>       89.523 107.52
+ USize  1   89.464 109.46
- Mitos  1   93.552 109.55
- NNucl  1   95.231 111.23
- Adhes  1   99.042 115.04
- Chrom  1  100.153 116.15
- BNucl  1  109.064 125.06
- Thick  1  110.465 126.47

Step:  AIC=105.66
Class ~ Adhes + BNucl + Chrom + Mitos + NNucl + Thick + UShap

        Df Deviance    AIC
<none>       89.662 105.66
- UShap  1   91.884 105.88
+ Epith  1   89.523 107.52
+ USize  1   89.613 107.61
- Mitos  1   93.714 107.71
- NNucl  1   95.853 109.85
- Adhes  1  100.126 114.13
- Chrom  1  100.844 114.84
- BNucl  1  109.762 123.76
- Thick  1  110.632 124.63
```
We can see that the final model better fits the data according to AIC (lower is better).

---
#### Making Predictions
```R
newdata <- list(Adhes=1, BNucl=1, Chrom=3,Mitos=1, NNucl=1, Thick=4, UShap=1)
> predict(minaic.logistic,newdata, type="response")
        1
0.9921115
```

Things get a little trickier when trying to determine a confidence interval.
We need to get a point estimate as well as standard errors on a linear scale before
we convert those back into probabilities.

```R
> linear.bounds <- predict(minaic.logistic , newdata = list(Adhes=1, BNucl=1, Chrom=3, Mitos=1, NNucl=1, Thick=4, UShap=1), type="link", se.fit=TRUE)
> linear.bounds
$fit
       1
4.834428

$se.fit
[1] 0.5815185

$residual.scale
[1] 1

```
Using a 95% confidence interval around the point estimate we get:

```R
> ilogit(c(linear.bounds$fit-1.96*x$se.fit, linear.bounds$fit, linear.bounds$fit+1.96*x$se.fit))
        1         1         1
0.9757467 0.9921115 0.9974629
```
