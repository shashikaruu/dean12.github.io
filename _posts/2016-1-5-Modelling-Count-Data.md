---
layout: post
title: Defects in a wave soldering process
---


### Modelling defects in a wave soldering process using a poisson distribution


Components are attached to an electronic circuit card assembly by a wave-soldering process. The soldering process involves baking and preheating the circuit card and then passing it through a solder wave by conveyor. Defect arise during the process. Design is 2^{7-3} with 3 replicates.





### Part1: Exploring the data

First we load the data and plot it to see if we can observe any trends.


```R
library(faraway)
data(wavesolder)
```


```R
head(wavesolder)
```




<table>
<thead><tr><th></th><th scope=col>y1</th><th scope=col>y2</th><th scope=col>y3</th><th scope=col>prebake</th><th scope=col>flux</th><th scope=col>speed</th><th scope=col>preheat</th><th scope=col>cooling</th><th scope=col>agitator</th><th scope=col>temp</th></tr></thead>
<tbody>
	<tr><th scope=row>1</th><td>13</td><td>30</td><td>26</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr>
	<tr><th scope=row>2</th><td>4</td><td>16</td><td>11</td><td>1</td><td>1</td><td>1</td><td>2</td><td>2</td><td>2</td><td>2</td></tr>
	<tr><th scope=row>3</th><td>20</td><td>15</td><td>20</td><td>1</td><td>1</td><td>2</td><td>1</td><td>1</td><td>2</td><td>2</td></tr>
	<tr><th scope=row>4</th><td>42</td><td>43</td><td>46</td><td>1</td><td>1</td><td>2</td><td>2</td><td>2</td><td>1</td><td>1</td></tr>
	<tr><th scope=row>5</th><td>14</td><td>15</td><td>17</td><td>1</td><td>2</td><td>1</td><td>1</td><td>2</td><td>1</td><td>2</td></tr>
	<tr><th scope=row>6</th><td>10</td><td>17</td><td>16</td><td>1</td><td>2</td><td>1</td><td>2</td><td>1</td><td>2</td><td>1</td></tr>
</tbody>
</table>




The response variable is count data and appears to be poisson distributed. We also know that y1, y2, and y3 are independent trials. This means that we can stack columns y1, y2, and y3 on top of each other with their corresponding X values.

First we stack the response variables:


```R
response.vars <- stack(wavesolder,c(y1,y2,y3))
```

Now we stack the data frame on its self three times:


```R
wavesolder.combined <- rbind(wavesolder,wavesolder,wavesolder)
```

Now we merge the two and name the columns appropriately:


```R
final.wavesolder <- cbind(response.vars$values, wavesolder.combined$prebake, wavesolder.combined$flux,
                          wavesolder.combined$speed, wavesolder.combined$preheat, wavesolder.combined$cooling,
                          wavesolder.combined$agitator, wavesolder.combined$temp)
final.wavesolder <- as.data.frame(final.wavesolder)


colnames(final.wavesolder) <- c("response", "prebake", "flux", "speed",
                                "preheat", "cooling", "agitator", "temp")
```


```R
head(final.wavesolder)
```




<table>
<thead><tr><th></th><th scope=col>response</th><th scope=col>prebake</th><th scope=col>flux</th><th scope=col>speed</th><th scope=col>preheat</th><th scope=col>cooling</th><th scope=col>agitator</th><th scope=col>temp</th></tr></thead>
<tbody>
	<tr><th scope=row>1</th><td>13</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr>
	<tr><th scope=row>2</th><td>4</td><td>1</td><td>1</td><td>1</td><td>2</td><td>2</td><td>2</td><td>2</td></tr>
	<tr><th scope=row>3</th><td>20</td><td>1</td><td>1</td><td>2</td><td>1</td><td>1</td><td>2</td><td>2</td></tr>
	<tr><th scope=row>4</th><td>42</td><td>1</td><td>1</td><td>2</td><td>2</td><td>2</td><td>1</td><td>1</td></tr>
	<tr><th scope=row>5</th><td>14</td><td>1</td><td>2</td><td>1</td><td>1</td><td>2</td><td>1</td><td>2</td></tr>
	<tr><th scope=row>6</th><td>10</td><td>1</td><td>2</td><td>1</td><td>2</td><td>1</td><td>2</td><td>1</td></tr>
</tbody>
</table>





```R

```


```R
tail(final.wavesolder)
```




<table>
<thead><tr><th></th><th scope=col>response</th><th scope=col>prebake</th><th scope=col>flux</th><th scope=col>speed</th><th scope=col>preheat</th><th scope=col>cooling</th><th scope=col>agitator</th><th scope=col>temp</th></tr></thead>
<tbody>
	<tr><th scope=row>43</th><td>19</td><td>2</td><td>1</td><td>2</td><td>1</td><td>2</td><td>1</td><td>2</td></tr>
	<tr><th scope=row>44</th><td>151</td><td>2</td><td>1</td><td>2</td><td>2</td><td>1</td><td>2</td><td>1</td></tr>
	<tr><th scope=row>45</th><td>11</td><td>2</td><td>2</td><td>1</td><td>1</td><td>1</td><td>2</td><td>2</td></tr>
	<tr><th scope=row>46</th><td>17</td><td>2</td><td>2</td><td>1</td><td>2</td><td>2</td><td>1</td><td>1</td></tr>
	<tr><th scope=row>47</th><td>89</td><td>2</td><td>2</td><td>2</td><td>1</td><td>1</td><td>1</td><td>1</td></tr>
	<tr><th scope=row>48</th><td>7</td><td>2</td><td>2</td><td>2</td><td>2</td><td>2</td><td>2</td><td>2</td></tr>
</tbody>
</table>




Now lets look at a plot of our response variable.


```R
options(repr.plot.width = 5)
options(repr.plot.height = 5)
plot(density(final.wavesolder$response))
```


![svg](ass3redo-Copy1_files/ass3redo-Copy1_15_0.svg)


It still appears that a poisson regression will model the count data well. The observations greater than 100 may pose a problem in the model and we must be wary of this.

### Part 2: Fitting a model


```R
wavesolder.poisson1 <- glm(response ~ ., data=final.wavesolder, family=poisson)
summary(wavesolder.poisson1)
```





    Call:
    glm(formula = response ~ ., family = poisson, data = final.wavesolder)

    Deviance Residuals:
        Min       1Q   Median       3Q      Max  
    -7.9074  -2.1202  -0.4121   1.5501  12.1486  

    Coefficients:
                Estimate Std. Error z value Pr(>|z|)    
    (Intercept)  2.85572    0.23670  12.065  < 2e-16 ***
    prebake      0.65124    0.05442  11.968  < 2e-16 ***
    flux        -0.57667    0.05438 -10.604  < 2e-16 ***
    speed        1.22875    0.06113  20.100  < 2e-16 ***
    preheat     -0.16645    0.05343  -3.115  0.00184 **
    cooling     -0.16187    0.05313  -3.047  0.00231 **
    agitator    -0.08974    0.05263  -1.705  0.08819 .  
    temp        -0.69816    0.05537 -12.609  < 2e-16 ***
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

    (Dispersion parameter for poisson family taken to be 1)

        Null deviance: 1450.52  on 47  degrees of freedom
    Residual deviance:  491.78  on 40  degrees of freedom
    AIC: 738.52

    Number of Fisher Scoring iterations: 5






Almost all the explanatory variables are significant to some degree. It would be worthwile to examine the dispersion parameter.


### Part 3: Model Diagnostics and Overdispersion

A key assumption of the poisson distribution is that the variance equals the mean. This requires assuming a dispersion parameter of 1.
Let's estimate the dispersion parameter from the data.


```R
# estimate phi (phi-hat)
n <- 47   
p <- 7               
(phihat <- sum(residuals(wavesolder.poisson1,type="pearson")^2)/(n-p))
```




13.4166619584806



That is a very high phi-hat. We will rescale the variances using this estimated phi-hat.


```R
summary(wavesolder.poisson1, dispersion=phihat)
```





    Call:
    glm(formula = response ~ ., family = poisson, data = final.wavesolder)

    Deviance Residuals:
        Min       1Q   Median       3Q      Max  
    -7.9074  -2.1202  -0.4121   1.5501  12.1486  

    Coefficients:
                Estimate Std. Error z value Pr(>|z|)    
    (Intercept)  2.85572    0.86701   3.294 0.000989 ***
    prebake      0.65124    0.19932   3.267 0.001086 **
    flux        -0.57667    0.19920  -2.895 0.003792 **
    speed        1.22875    0.22392   5.487 4.08e-08 ***
    preheat     -0.16645    0.19572  -0.850 0.395060    
    cooling     -0.16187    0.19461  -0.832 0.405537    
    agitator    -0.08974    0.19279  -0.465 0.641586    
    temp        -0.69816    0.20282  -3.442 0.000577 ***
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

    (Dispersion parameter for poisson family taken to be 13.41666)

        Null deviance: 1450.52  on 47  degrees of freedom
    Residual deviance:  491.78  on 40  degrees of freedom
    AIC: 738.52

    Number of Fisher Scoring iterations: 5




We can see that 'preheat', 'cooling' and 'agitator' do not appear to be significant anymore while the other variables maintain their significance.

We now perform an F-test on the model.


```R
anova(wavesolder.poisson1,dispersion=phihat,test='Chisq')
```




<table>
<thead><tr><th></th><th scope=col>Df</th><th scope=col>Deviance</th><th scope=col>Resid. Df</th><th scope=col>Resid. Dev</th><th scope=col>Pr(>Chi)</th></tr></thead>
<tbody>
	<tr><th scope=row>NULL</th><td>NA</td><td>NA</td><td>47</td><td>1450.516</td><td>NA</td></tr>
	<tr><th scope=row>prebake</th><td>1</td><td>165.7739</td><td>46</td><td>1284.742</td><td>0.0004396137</td></tr>
	<tr><th scope=row>flux</th><td>1</td><td>104.5412</td><td>45</td><td>1180.2</td><td>0.005248107</td></tr>
	<tr><th scope=row>speed</th><td>1</td><td>491.1141</td><td>44</td><td>689.0864</td><td>1.446756e-09</td></tr>
	<tr><th scope=row>preheat</th><td>1</td><td>1.747099</td><td>43</td><td>687.3393</td><td>0.7182055</td></tr>
	<tr><th scope=row>cooling</th><td>1</td><td>25.66865</td><td>42</td><td>661.6706</td><td>0.1666091</td></tr>
	<tr><th scope=row>agitator</th><td>1</td><td>1.45499</td><td>41</td><td>660.2156</td><td>0.7419198</td></tr>
	<tr><th scope=row>temp</th><td>1</td><td>168.4397</td><td>40</td><td>491.7759</td><td>0.0003952499</td></tr>
</tbody>
</table>




We will try building another model and excluding 'preheat', 'cooling' and 'agitator'.


```R
wavesolder.poisson2 <- glm(response ~ prebake + flux + speed + temp, data=final.wavesolder, family=poisson)
summary(wavesolder.poisson2)
```





    Call:
    glm(formula = response ~ prebake + flux + speed + temp, family = poisson,
        data = final.wavesolder)

    Deviance Residuals:
        Min       1Q   Median       3Q      Max  
    -8.0503  -1.9044  -0.5489   1.8995  12.5918  

    Coefficients:
                Estimate Std. Error z value Pr(>|z|)    
    (Intercept)  2.12400    0.17496   12.14   <2e-16 ***
    prebake      0.67287    0.05374   12.52   <2e-16 ***
    flux        -0.52878    0.05262  -10.05   <2e-16 ***
    speed        1.23048    0.06076   20.25   <2e-16 ***
    temp        -0.69315    0.05392  -12.86   <2e-16 ***
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

    (Dispersion parameter for poisson family taken to be 1)

        Null deviance: 1450.52  on 47  degrees of freedom
    Residual deviance:  513.75  on 43  degrees of freedom
    AIC: 754.49

    Number of Fisher Scoring iterations: 5





```R
# estimate phi (phi-hat)
n <- 47   
p <- 4               
(phihat <- sum(residuals(wavesolder.poisson2,type="pearson")^2)/(n-p))
```




13.1737625979895



Again the estimated phi value is quite high indicating overdispersion. We scale the variances to account for this.


```R
summary(wavesolder.poisson2, dispersion=phihat)
```





    Call:
    glm(formula = response ~ prebake + flux + speed + temp, family = poisson,
        data = final.wavesolder)

    Deviance Residuals:
        Min       1Q   Median       3Q      Max  
    -8.0503  -1.9044  -0.5489   1.8995  12.5918  

    Coefficients:
                Estimate Std. Error z value Pr(>|z|)    
    (Intercept)   2.1240     0.6350   3.345 0.000824 ***
    prebake       0.6729     0.1950   3.450 0.000561 ***
    flux         -0.5288     0.1910  -2.769 0.005629 **
    speed         1.2305     0.2205   5.579 2.41e-08 ***
    temp         -0.6931     0.1957  -3.542 0.000397 ***
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

    (Dispersion parameter for poisson family taken to be 13.17376)

        Null deviance: 1450.52  on 47  degrees of freedom
    Residual deviance:  513.75  on 43  degrees of freedom
    AIC: 754.49

    Number of Fisher Scoring iterations: 5





```R
anova(wavesolder.poisson2,dispersion=phihat,test='Chisq')
```




<table>
<thead><tr><th></th><th scope=col>Df</th><th scope=col>Deviance</th><th scope=col>Resid. Df</th><th scope=col>Resid. Dev</th><th scope=col>Pr(>Chi)</th></tr></thead>
<tbody>
	<tr><th scope=row>NULL</th><td>NA</td><td>NA</td><td>47</td><td>1450.516</td><td>NA</td></tr>
	<tr><th scope=row>prebake</th><td>1</td><td>165.7739</td><td>46</td><td>1284.742</td><td>0.0006230859</td></tr>
	<tr><th scope=row>flux</th><td>1</td><td>104.5412</td><td>45</td><td>1180.2</td><td>0.006588277</td></tr>
	<tr><th scope=row>speed</th><td>1</td><td>491.1141</td><td>44</td><td>689.0864</td><td>3.888421e-09</td></tr>
	<tr><th scope=row>temp</th><td>1</td><td>175.3358</td><td>43</td><td>513.7506</td><td>0.0004337474</td></tr>
</tbody>
</table>




##### We now take a look at observations that are having a large effect on the results.


```R
plot(wavesolder.poisson2)
```


![svg](ass3redo-Copy1_files/ass3redo-Copy1_34_0.svg)



![svg](ass3redo-Copy1_files/ass3redo-Copy1_34_1.svg)



![svg](ass3redo-Copy1_files/ass3redo-Copy1_34_2.svg)



![svg](ass3redo-Copy1_files/ass3redo-Copy1_34_3.svg)


We now remove rows 25, 27 and 43 and re-estimate the model


```R
outlierrem.wavesolder <- final.wavesolder[-c(25, 27, 43), ]
```


```R
wavesolder.poisson3 <- glm(response ~ prebake + flux + speed + temp, data=outlierrem.wavesolder, family=poisson)
summary(wavesolder.poisson3)
```





    Call:
    glm(formula = response ~ prebake + flux + speed + temp, family = poisson,
        data = outlierrem.wavesolder)

    Deviance Residuals:
        Min       1Q   Median       3Q      Max  
    -5.7692  -1.4484  -0.3082   1.2061   4.4073  

    Coefficients:
                Estimate Std. Error z value Pr(>|z|)    
    (Intercept)  2.81267    0.18415  15.274  < 2e-16 ***
    prebake      0.59401    0.05600  10.608  < 2e-16 ***
    flux        -0.44562    0.05518  -8.075 6.74e-16 ***
    speed        1.02997    0.06229  16.534  < 2e-16 ***
    temp        -0.97919    0.06422 -15.249  < 2e-16 ***
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

    (Dispersion parameter for poisson family taken to be 1)

        Null deviance: 1073.53  on 44  degrees of freedom
    Residual deviance:  225.63  on 40  degrees of freedom
    AIC: 454.59

    Number of Fisher Scoring iterations: 5





```R
# estimate phi (phi-hat)
n <- 44   
p <- 4               
(phihat <- sum(residuals(wavesolder.poisson2,type="pearson")^2)/(n-p))
```




14.1617947928387




```R
summary(wavesolder.poisson3, dispersion=phihat)
```





    Call:
    glm(formula = response ~ prebake + flux + speed + temp, family = poisson,
        data = outlierrem.wavesolder)

    Deviance Residuals:
        Min       1Q   Median       3Q      Max  
    -5.7692  -1.4484  -0.3082   1.2061   4.4073  

    Coefficients:
                Estimate Std. Error z value Pr(>|z|)    
    (Intercept)   2.8127     0.6930   4.059 4.94e-05 ***
    prebake       0.5940     0.2107   2.819  0.00482 **
    flux         -0.4456     0.2077  -2.146  0.03189 *  
    speed         1.0300     0.2344   4.394 1.12e-05 ***
    temp         -0.9792     0.2417  -4.052 5.08e-05 ***
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

    (Dispersion parameter for poisson family taken to be 14.16179)

        Null deviance: 1073.53  on 44  degrees of freedom
    Residual deviance:  225.63  on 40  degrees of freedom
    AIC: 454.59

    Number of Fisher Scoring iterations: 5





```R
anova(wavesolder.poisson3,dispersion=phihat,test='Chisq')
```




<table>
<thead><tr><th></th><th scope=col>Df</th><th scope=col>Deviance</th><th scope=col>Resid. Df</th><th scope=col>Resid. Dev</th><th scope=col>Pr(>Chi)</th></tr></thead>
<tbody>
	<tr><th scope=row>NULL</th><td>NA</td><td>NA</td><td>44</td><td>1073.526</td><td>NA</td></tr>
	<tr><th scope=row>prebake</th><td>1</td><td>118.9356</td><td>43</td><td>954.5903</td><td>0.003755627</td></tr>
	<tr><th scope=row>flux</th><td>1</td><td>79.50418</td><td>42</td><td>875.0861</td><td>0.01781764</td></tr>
	<tr><th scope=row>speed</th><td>1</td><td>383.0064</td><td>41</td><td>492.0798</td><td>1.987691e-07</td></tr>
	<tr><th scope=row>temp</th><td>1</td><td>266.4534</td><td>40</td><td>225.6264</td><td>1.440341e-05</td></tr>
</tbody>
</table>




Residuals were checked again but none were found to significantly impact the model.

### Part 4: Conclusion

It appears that 'prebake', 'flux', 'speed', and 'temp' are statistically significant. Overdispersion in the data was found and as a result variances were re-estimated using the higher dispersion parameter. Afterward observations that were having a significant and determental effect on the model were excluded and another model was built with the remaining data.


Both prebake and speed are positivily related to the number of defects caused. This would suggest that a lower value (or factor) of prebake and speed will result in fewer defects. The opposite applies to flux and temterature as both are negatively related to defects. Choosing a higher setting on these variables is likely to result in fewer defects as well.



##### Extensions

* Using a negative binomial regression instead of a quasi-poisson regression to model the count data. Literature suggests that it may perform better in certain conditions.
