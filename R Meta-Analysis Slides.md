R for Meta-Analysis
========================================================
author: Cort W. Rudolph, Ph.D.
date: March 28, 2018  
autosize: true
font-family: 'Helvetica'


Three Facts About Me
========================================================
- Ph.D. in Industrial/Organizational Psychology
> My research focuses on various issues related to the aging workforce, including the application of lifespan development theories, well‐being and work‐longevity, successful aging, and ageism

- I have been a meta-analyst for about 10 years
- I have been using R as a my exclusive statistics platform for about 5 years

Meta-Analysis (MA)
========================================================
Extremely Abbreviated History of MA:
<small>
* 1952: Hans J. Eysenck publishes a narrative review concluding that there were no favorable effects of psychotherapy.
  + This starts a raging debate!
* 20 years of evaluation research and hundreds of studies failed to resolve the debate 
* 1976: To prove Eysenck wrong, Gene V. Glass statistically aggregates the findings of 375 psychotherapy outcome studies 
  + Glass & Smith concluded that psychotherapy did indeed work!
  + In fact, it generalized across modalities…
* Glass called his method “meta-analysis.”
</small>

Meta-Analysis (MA)
========================================================
Classical Definition: 
> The statistical analysis of a large collection of analysis results for the purpose of integrating the findings (Glass, 1976)

Contemporary Definition: 
> Meta-analysis is a procedure for statistically synthesizing the results from a series of primary studies (e.g., Borenstein et al., 2009). 

Meta-Analysis (MA)
========================================================
* Meta-analyses are generally centered on the relationship between one explanatory (X) and one response variable (Y) that is found within a population of primary studies. 
  + This relationship, “the relationship between X and Y,” defines the analysis.
  + The inputs to our meta-analyis are standardized indices (i.e., effect sizes) of this X-Y relationship (e.g., $r_{xy}$)

Meta-Analysis (MA)
========================================================
Seven steps in MA:
<small>
* Step 1: Formulating the Problem (i.e., Define the theoretical relationship of interest) 
* Step 2: Searching the Literature (i.e., Collect the population of studies that provide data on the relationship) 
* Step 3: Gathering Information from Studies (i.e., Code the studies and compute effect sizes) 
* Step 4: Evaluating the Quality of Studies (i.e., Consider methodological artifacts that impact valid inferences)
* <b> Step 5: Analyzing and Integrating the Outcomes of Studies (i.e., Examine the distribution of effect sizes and analyze the impact of moderating variables)
* Step 6: Interpreting the Evidence (i.e., Hypothesis testing) </b>
* Step 7: Presenting the Results (Write the meta-analytic report)
</small>

Meta-Analysis (MA)
========================================================
Today, we will be focusing on some basic MA models for correlations. 
* We will work out of a simulated dataset that I have put together specifically for this demonstration.

Meta-Analysis (MA)
========================================================
First, we need to read in this data and take a look at it:

```r
# install.packages(readxl)
library(readxl)
RMetaData <- read_excel("R Meta-Analysis Data.xlsx")
```

Meta-Analysis (MA)
========================================================
This dataset is a 25x6 matrix, containing the following variables:
<small>
* [,1] 'studyNumber' = An index the study number in our database (<i>K</i> = 25)
* [,2] 'Citation' = A short citation for the study being considered
* [,3] 'rxy' = An effect size estimate (i.e., a correlation, $r_{xy}$)
* [,4] 'N' = The sample size that was used to estimate 'rxy'          
* [,5] 'mod' = A categorical study-level moderator (e.g., 0 = self vs. 1 = other reports of Y)
</small>

```
# A tibble: 3 x 5
  studyNumber            Citation        rxy     N   mod
        <dbl>               <chr>      <dbl> <dbl> <dbl>
1           1 Cohen et al. (1998) 0.15374804   377     0
2           5  Nash et al. (2012) 0.08175671    77     1
3          15 Munoz et al. (2014) 0.04320237   108     1
```

Meta-Analysis (MA)
========================================================
To begin, let's consider the average correlation: 
* This represents the unweighted average of $r_{xy}$ across the <i>K</i> = 25 studies considered here

```r
mean(RMetaData$rxy)
```

```
[1] 0.1216309
```

Meta-Analysis (MA)
========================================================
We will be working with the 'metafor' package today, so let's install & load it now.
* You can learn more about this package, here: http://www.metafor-project.org/doku.php

We will also use a function in the 'MAc' packages, so let's get that installed & loaded, too:

```r
# install.packages("metafor")
library(metafor)

# install.packages("MAc")
library(MAc)
```

Meta-Analysis (MA)
========================================================
As a first step in our analyses, we need to compute the sampling variances for each $r_{xy}$
* Its common to weight each $r_{xy}$ by the inverse of its sampling variance:
$$\frac{1}{\sigma^2}$$
* The logic to this is fairly straightforward:
  + Studies with larger Ns are more precise estimates of the parameter. 
  + Studies with larger Ns also have smaller sampling variances. 
  + Therefore, by weighting each study by the inverse of its sampling variance, larger N studies get more "credit" in our model.


Meta-Analysis (MA)
========================================================
The 'var_r' function in the MAc computes the sampling varance for us simply:

```r
#Variance 
RMetaData$var_rxy <- var_r(RMetaData$rxy, RMetaData$N)
```

Meta-Analysis (MA)
========================================================
With our sampling variances computed, let's start by specifying a simple meta-analysis model.
* The basic function for a random-effects meta-analysis in 'metafor' is 'rma'

* We need to supply several arguments to this function:
  + 'yi' = the effect to be meta-analyzed, in our case 'rxy'
  + 'vi' = the sampling variance of the effect to be analyzed, in our case 'var_rxy'
  + 'data' = our dataframe, 'RMetaData'
  + 'method' = the estimation method, here 'REML'

```r
meta_mod1 = rma(yi = rxy, vi = var_rxy, data=RMetaData, method="REML")
```

Meta-Analysis (MA)
========================================================
Taking a look at this output, we learn several important things:

* The average inverse-variance weighted effect, $\overline{r}_{xy}\$$\approx$.15 
  + How does this compare to our original estimate of the average?

```r
meta_mod1$beta
```

```
             [,1]
intrcpt 0.1494709
```

Meta-Analysis (MA)
========================================================
* The 95% CI around this estimate does not include zero, so we are comfortable saying that it is statistically significantly different from zero (<i>p</i> < .05)

```r
rbind(c(meta_mod1$ci.lb,meta_mod1$ci.ub))
```

```
          [,1]     [,2]
[1,] 0.1209358 0.178006
```

Meta-Analysis (MA)
========================================================
Moreover, from our ouptut we learn that are $I^2$ is relatively high:

```r
meta_mod1$I2
```

```
[1] 24.8278
```
* $I^2$ = Proportion of total variation in effect sizes that is due to systematic differences between effect sizes rather than by chance (see Shadish & Haddock, 2009; pp. 263)
  + This suggests heterogeneity of effect sizes, and hence to presence of moderators


Meta-Analysis (MA)
========================================================
We can also get a "forest plot" to visualize this effect against the <i>K</i> = 25 studies considered in our model

<img src="R Meta-Analysis Slides-figure/unnamed-chunk-10-1.png" title="plot of chunk unnamed-chunk-10" alt="plot of chunk unnamed-chunk-10" style="display: block; margin: auto;" />

Meta-Analysis (MA)
========================================================
So, with this in mind, let's consider tests of moderation.
* Specifically, let's test whether there are differences based upon the source of the Y
  + 0 = self report, 1 = other report
  

```r
meta_mod2 = rma(yi = rxy, vi = var_rxy, mods= ~ factor(mod)-1, data=RMetaData, method="REML")
```

Meta-Analysis (MA)
========================================================
From this output, we learn several additional things:
* $\overline{r}_{xy}\$ appears to be different for self vs. other reports

```r
meta_mod2$beta
```

```
                   [,1]
factor(mod)0 0.18071046
factor(mod)1 0.07319895
```


Meta-Analysis (MA)
========================================================
Non-overlapping confidence intervals suggest "source" is a moderator:
* 'mod' = 0 (Self)

```r
rbind(c(meta_mod2$ci.lb[1],meta_mod2$ci.ub[1]))
```

```
          [,1]      [,2]
[1,] 0.1543327 0.2070883
```
* As compared to when 'mod' = 1 (Other)

```r
rbind(c(meta_mod2$ci.lb[2],meta_mod2$ci.ub[2]))
```

```
           [,1]      [,2]
[1,] 0.02231076 0.1240871
```


Meta-Analysis (MA)
========================================================
Additionally...

* $I^2$ is reduced to zero

```r
meta_mod2$I2
```

```
[1] 0
```

* In other words, modeling this moderator accounts for all of the original heterogeneity that we observed among effect sizes

Meta-Analysis (MA)
========================================================
Concluding thoughts:
* MA is very <b>flexible</b> 
  + Multiple type of effects can be considered: (e.g., $d_{Cohen}$, $OR$, $\overline{x}_{raw}\$)
* MA is very <b>powerful</b>
  + Best estimates of population parameters
  + Effects as inputs to more complex models (e.g., MASEM; Viswesvaran & Ones, 1995)
* MA is very <b>extensible</b>
  + 'Validity Generalization' - psychometric meta-analysis that corrects for statistcal artifacts (e.g., $r_{yy}\$, $U_{x}\$) to arrive at "true score" estimates of population parameters, $\rho_{xy}$

