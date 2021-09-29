## Goals for today  

-   Linear regression with one continuous outcome, one or more predictors  
    -   understand model R^2, F-statistic, beta coefficients (standardized, unstandardized)  
    -   check model residuals for potential sources of bias  
        - linearity, heteroscedasticity, independence, normality and unusual cases   
    - F-statistic for model comparisons  
    
-   Dichotomous outcome: logistic regression
    -   why not use regular linear regression?
    -   sigmoid link function (logit)  
    -   interpret coefficients as log-odds  


------------------------------------------------------------------------



## Step 1 - Get organized
- **Earlier you downloaded and unzipped [multi-regression.zip](../../templates/multi-regression.zip)**   
- Now open RStudio and start a new project, select "Existing Directory" and select the new folder you unzipped as the location    
- In RStudio, open the multi-regression.Rmd doc inside and do your work in there  
  - run the setup code chunk (the necessary `library()` statements are in there)  
- In the RStudio console, install the packages you'll need today with the install.packages() command:
  - `install.packages("GGally")`  
  - `install.packages("parameters")`  


------------------------------------------------------------------------

## Step 2 - Import data and check it out  

- data description: lumos_subset1000.csv is the same file we worked with last week. 
This is subset of a public dataset of lumosity (a cognitive training website) user performance data. You can find the publication associated with the full dataset here:  
[Guerra-Carrillo, B., Katovich, K., & Bunge, S. A. (2017). Does higher education hone cognitive functioning and learning efficacy? Findings from a large and diverse sample. PloS one, 12(8), e0182276. https://doi.org/10.1371/journal.pone.0182276](https://doi.org/10.1371/journal.pone.0182276)

  - this data subset includes only the arithmetic reasoning test (AR) score from a post-test at the end of a 100 day training program (`raw_score`)  
  - `pretest_score` (test at start of the training program) has been transformed to have a mean of 100 and standard deviation of 15 (this is a typical transformation for IQ scores)  

- **What to do first:** Make a new code chunk and use readr::read_csv() to read in the data. Make sure that NA values are handled the way you want (click on the tibble in the Environment window pane to take a quick look).   
- **What to do next:** make sure the columns that contain nominal vals are treated as nominal, using forcats::as_factor()  *you can copy your code chunk from last week, or just look at the solution below*

<button class="btn btn-primary" data-toggle="collapse" data-target="#step-2"> Show/Hide Solution </button>  
<div id="step-2" class="collapse">  
```{r Step-2-import, fig.show='hold', results='hold', message=FALSE}
#first import the data
lumos_tib <- readr::read_csv("data/lumos_subset1000plusimaginary.csv", na = "NA")
# now make sure the columns we want as factors are treated that way, using forcats::as_factor()
lumos_tib <- lumos_tib %>% dplyr::mutate(
  test_type = forcats::as_factor(test_type),
  assessment = forcats::as_factor(assessment),
  gender = forcats::as_factor(gender),
  edu_cat = forcats::as_factor(edu_cat),
  english_nativelang = forcats::as_factor(english_nativelang),
  ethnicity = forcats::as_factor(ethnicity)
)
```
</div> 
&nbsp;

------------------------------------------------------------------------

## Step 3 - The General Linear Model (GLM) with one predictor  
![GLM Decision chart](../images/glm_decision.png){width=50%} 

#### Above is the decision process chart from the book.  

- It says we should start by using scatter plots to check for non-linear associations and unusual cases. 
- Last week we looked at scatter plots  of pretest_score, raw_score, and age. The linearity assumption was reasonable and there were no concerning outliers, but let's use a shortcut to recreate those scatter plots all at once. Use the function `GGally::ggscatmat()` like in the code below.  

<button class="btn btn-primary" data-toggle="collapse" data-target="#step-3"> Show/Hide Solution </button>  
<div id="step-3" class="collapse">
```{r Step3a,fig.show='hold', results='hold'}
p1 <- lumos_tib %>% 
  GGally::ggscatmat(columns = c("age","pretest_score","raw_score"))
p1
```
</div>
&nbsp;
- Notice that the `age` variable is not normally distributed, but normality of the variables, especially with a large data set is not a concern. The scatters show that a linear relation between variables is a reasonable assumption    
- So let's dive straight in and use linear regression with `age` as an explanatory variable for `raw_score`  
- We will use the `lm()` function, piping in the data and specifying `age` as a predictor for `raw_score` with the arguments `formula = raw_score ~ age` (an intercept term is automatically included), and `data=.` (the "." means "use the data from the pipe on the left/above"). Save the model in a variable called `score_lm`    
- use `summary(score_lm)` to show the model summary  

<button class="btn btn-primary" data-toggle="collapse" data-target="#step3b"> Show/Hide Solution </button>  
<div id="step3b" class="collapse">
```{r Step3b, fig.show='hold', results='hold'}
score_lm <- lumos_tib %>% drop_na(age,raw_score) %>% 
  lm(formula = raw_score ~ age, data = .)
summary(score_lm)
```
</div>

### Looking at the model summary:  
1. The Multiple R-squared value tells us that `age` explains about 1.03% of the variance in `raw_score` (not much by most standards).  
2. The F-statistic is the ratio of variance explained by the model to the error within the model (use `anova(score_lm)` to view Mean Sum of Squares for the model and residuals), and the p-value tells you the probability of an F-statistic at least that large under the null hypothesis.
3. The (unstandardized) beta coefficient  ("Estimate" column, "age" row) tells you that a change of one unit in the predictor (age) predicts a change of -.04 units in the outcome (raw_score). In other words, the model predicts that two people one year apart in age will be .04 units apart in their performance score (the older person having the lower predicted score).  
  - you can get standardized coefficients by using the command `parameters::modelparameters(score_lm, standardize="refit")` (this function also gives you confidence intervals for parameters)- the standardized coefficient for `age` indicates that a change of +1 standard deviation in age predicts a change of -.10 standard deviations in `raw_score` it is the same as if we first standardized (z-scored) each variable and then ran the regression model.  
4. The Intercept ("Estimate" column, "(Intercept)" row) tells us that if all predictors were at their zero level (i.e., at age zero) the model predicts a score of 17.84. It doesn't make a lot of sense to talk about a score for someone age zero, but this is how linear models work. If you wanted an interpretable intercept you could re-scale `age` so that the zero value was meaningful (e.g., re-center to mean=0, then the intercept would correspond to the predicted score for someone with the mean age)  
5. The "Std. Error" column gives the standard error of each estimate (intercept and age coefficient) -- for a visual explanation of standard error of regression coefficients, see the [Andy Field video on the topic (same as the one in the syllabus)](https://youtu.be/3L9ZMdzJyyI).  
6. The "t value" column gives the t-stat for each estimate (estimate/std err) and the "Pr(>|t|)" column gives the probability of observing a t-stat that far from zero under the null hypothesis.  
  - notice that the p-value for `age` (under "Pr(>|t|)") is the same as the overall p-value - this is because age is the only predictor in the model
  - notice that the t-stat and p-value for `age` are the same as the t-stat and p-value the we got last week for the correlation between `age` and `raw_score` - this is because the correlation is also a test of a linear relation (and `age` is the only predictor)  

  
------------------------------------------------------------------------

## Step 4 - The General Linear Model (GLM) with multiple predictors  

Now let's add `pretest_score` to our model, so that we are predicting `raw_score` as a function of `age` and `pretest_score`. See if you can write the code yourself (copy and edit the code from the previous step) before looking at the solution below. Store the model in a variable named `score_2pred_lm`.   

<button class="btn btn-primary" data-toggle="collapse" data-target="#step4a"> Show/Hide Solution </button>  
<div id="step4a" class="collapse">  
```{r Step4a,fig.show='hold', results='hold'}
score_2pred_lm <- lumos_tib %>% drop_na(age,raw_score,pretest_score) %>% 
  lm(formula = raw_score ~ age + pretest_score, data = .)
summary(score_2pred_lm)
```
</div>
&nbsp;

#### Now, answer the following questions for yourself based on the model summary  
1. What does the "Multiple R-squared" tell you for this model? That is, what percent of the variance in `raw_score` is explained by the model?
2. What does the overall F-statistic and p-value tell you?  
3. What does the beta coefficient for `pretest_score` tell you?  
4. What does it mean that the t-statistic for the `age` variable is not significant in this model? How does it compare to the partial correlation test that we ran last week with the same variables?    

------------------------------------------------------------------------

## Step 5 - Model diagnostics: Examine model residuals for potential sources of bias  

-Aside from the model summary information, when you fit a linear regression model, the `lm()` function produces a predicted outcome value for every set of predictor variables (e.g., if one participant has an age of 15 and a pretest_score of 99.3 then the predicted outcome value will be 17.84 + (-.006\*15) + (.275\*99.3), based on the model we just estimated.  
- The model **residuals** are the difference between each predicted score and the actual outcome value (`raw_score`), thus there are as many model residual values as there are cases. We'll look at these residuals in this step  

Now, look back at the decision process chart. It says we should look at "zpred vs zresid" to check for linearity, heteroscedasticity, and independence, then look at a histogram of residuals to check for normality. 

Fortunately, there is one easy function that will show everything we need: `plot(modelname)` 
- Use `plot(score_2pred_lm)` to show the diagnostic plots for the 2 predictor model from the last step.  

<button class="btn btn-primary" data-toggle="collapse" data-target="#step5a"> Show/Hide Solution </button>  
<div id="step5a" class="collapse">  
```{r Step5a,fig.show='hold', results='hold'}
plot(score_2pred_lm)
```
</div>
&nbsp;

#### Here is a brief description of each plot  

1. "Residuals vs. Fitted" - this is the unstandardized version of "zpred vs zresid" and is just as useful for checking linearity, heteroscedasticity, and independence. We are basically checking to make sure there are no clear patterns in the residuals. 

  - Here are some [examples of patterns](../images/residualpatterns.png) you might see in residuals that indicate sources of bias (from Prof Andy Field's discovr tutorial section 08 - *note that the non-linear example is also an example of non-independence, because the residuals are related to the fitted values.*)  

2. "Normal Q-Q" - this is a Q-Q plot of the residuals, and helps us check for non-normally distributed residuals. We expect points to fall close to the dotted line if the residuals are normal (they do here).  
3. "Scale-Location" - similar to the first plot, but the y-axis is the square root of the absolute value of residuals. Use this for checking linearity, heteroscedasticity, and independence along with the first plot - they contain the similar information but some heteroscedastic patterns may be more apparent in one plot or the other. See [BoostedML](https://boostedml.com/2019/03/linear-regression-plots-scale-location-plot.html) for more examples of heteroscedasticity.  
4. "Residuals vs Leverage" - Points with high leverage are cases that are potentially influential because of their extreme values. Here, we are checking (1) that the spread of residuals does not change across leverage values (approximately horizontal solid red line indicates no heteroscedasticity concern), and (2) that there are no points with very high influence (points outside the dotted red line, which is not visible in this case because there are no high leverage points), which would violate linearity. We saw an example of high leverage last week in [Anscome's Quartet Plot 4](../images/highleverage.png).   

----------------------------------------------------------------------------

## Step 6 - Compare Models  
We can compare two models (w/ the same outcome data) by computing an F-statistic based on the ratio of error (RSS=residual sum of squares) of one model to the other (for example, a full model with all predictors compared to less full model with a subset of predictors). If one model is better at explaining the outcome than the other, than the RSS of the worse model will be greater than the RSS of the better model, and the F-statistic quantifies that for us (and gives a p-value under the null hypothesis). You can also rephrase this F-statistic as a ratio based on R^2 for each model (see Chapter 9 of the Field Textbook)   
- Let's compare the last model (`age` and `pretest_score`) to the first model (`age` only) of `raw_score`. Because we are comparing the residual variance between models, we can use the `anova()` function in R. The arguments we pass are the names of the two models (put the fuller model second). Try it now.  

<button class="btn btn-primary" data-toggle="collapse" data-target="#step6a"> Show/Hide Solution </button>  
<div id="step6a" class="collapse">  
```{r Step6a,fig.show='hold', results='hold'}
anova(score_lm, score_2pred_lm)
```
</div>
&nbsp;

- The p-value tells us that the F-stat is unlikely under the null hypothesis (that the models are equivalent). In other words, the difference in variance explained by `age`+`pretest_score` vs `age` alone is unlikely if the models equally explain variance in `raw_score` (so the fuller model is better.   
- When you report a model comparison this way you report the F-statistic as well as the change in R-squared (R^2 of better model minus R^2 of the worse model). But one important issue is that adding more predictors will always increase R^2.  
- For this reason you may also report other indicators of model fit that account for the number of predictors, such as AIC and BIC (see Chapter 9 of the Field Textbook).  
- You can view the AIC (Akaike Information Criterion) and BIC (Bayesian Information Criterion) for each model by using  `broom::glance(modelname)` in R - lower AIC and BIC values indicate better model fit.  

----------------------------------------------------------------------------

## Step 7 - Quadratic/curvilinear associations  

When we looked just at `age` and `raw_score`, we saw a small association such that older participants scored lower. But is it possible that the association is really inverted U-shaped such that older is better at young ages, but worse at old ages?
We can examine this possibility by including a quadratic (age^2) term in the model. It's as simple as centering the age variable (by subtracting the mean) and squaring it, then including that term in the model. It's always best to include lower order terms when you add higher order terms, so our new formula will be `raw_score ~ age_ctr + age_squared` (first we have to compute `age_ctr` and `age_squared`). Try it now: first compute `age_ctr` and `age_squared`, then estimate the model with the new formula (store the model in a variable named `score_agesq_lm`).  

<button class="btn btn-primary" data-toggle="collapse" data-target="#step7a"> Show/Hide Solution </button>  
<div id="step7a" class="collapse">  
```{r Step7a,fig.show='hold', results='hold'}
lumos_tib <- lumos_tib %>% 
  mutate(
    age_ctr = age-mean(age),
    age_squared = age_ctr^2
  )
score_agesq_lm <- lumos_tib %>%  lm(formula = raw_score ~ age_ctr + age_squared, data = .)
summary(score_agesq_lm)
```
</div>
&nbsp;

#### Examine the model summary:
1. Notice that there are coefficient estimates for `age_ctr`, and `age_squared`.  
2. The "Multiple R-squared" is .015 - we can compare that to the model that had age only (R-squared was .010), so the quadratic term increases the variance explained just a little bit. The p-value for `age_squared` tells us that it is unlikely (p=.029) to find a t-stat this large under the null hypothesis (that the age_squared association is zero).  
3. You can go through the rest of the summary like before, but now let's spare a moment to think about **multi-collinearity**.  

#### Multi-collinearity (textbook Chapter 9, section 9.9.3)  
- when predictors in the model are highly correlated with each other, we have *multi-collinearity* and the estimates of coefficients for correlated terms are unstable (large std err and highly variable across samples)  
- an easy way to check for multi-collinearity is by computing the *VIF* (variance inflation factor) for each predictor. It measures the shared variance between one predictor and the others. The reciprocal of VIF is called *tolerance*. A general rule of thumb is that you should worry if you have VIF > 5 for any predictor in the model.  
- check the VIF for each predictor now, by running `car::vif(score_agesq_lm)` -- looks okay, right? But what if we hadn't centered the age variable before squaring it?  

----------------------------------------------------------------------------

## Step 8 - Dichotomous outcome: logistic regression  

Logistic regression is used when the outcome variable has just two possible values (e.g., true/false, on/off, yes/no, or greater/less than some critical threshold). For the sake of learning, let's imagine that a `raw_score` value of 20 or greater wins a $10 prize, so we want to see if we can predict who wins the prize based only on their years of education (`years_edu`).  
**Why can't we run a regular linear regression?** - Because regular linear regression will give you predicted values that fall outside the possible outcome values and are not interpretable.  
Logistic regression will yield predicted values between 0 and 1 that can be interpreted as the probability of the outcome (e.g., prize received) occurring. These predicted values follow a sigmoid-shaped logit function.  

#### Let's estimate a logistic model now  

- First create a new variable called `prize` that is 1 if `raw_score` is >= 20. Peek at the solution code to see how.  
- Now use the `glm()` function (glm=generalized linear model) to estimate the logistic model. Just like with `lm()` you can specify the formula as `prize ~ years_edu`, but now add the argument `family=binomial` to specify that you are estimating a logistic model.  
- store the model in a variable called `prize_binlm`, and use `summary(prize_binlm)` to view the model summary.  

<button class="btn btn-primary" data-toggle="collapse" data-target="#step8a"> Show/Hide Solution </button>  
<div id="step8a" class="collapse">  
```{r Step8a,fig.show='hold', results='hold'}
lumos_tib <- lumos_tib %>% 
  mutate(prize = ifelse(raw_score>=20,1,0))
prize_binlm <- lumos_tib %>% drop_na(prize,years_edu) %>% 
  glm(formula = prize ~ years_edu, data = ., family = binomial)
summary(prize_binlm)
```
</div>
&nbsp;

#### Interpretation:
- First, notice there is no R^2 value. This is because there is no universally agreed method for computing it, but you can compute something similar called Pseudo-R^2 if you want (e.g., see documentation for the [DescTools Package](https://www.rdocumentation.org/packages/DescTools/versions/0.99.41/topics/PseudoR2))

- Let's focus on the coefficient estimate for `years_edu`. In logistic regression, the coefficients are in units of log(odds) (log = natural logarithm). This means that if we increase a value of `years_edu` by one unit the model would predict an increase of .113 in the log(odds) of receiving a prize.  
- We can try to make that more understandable by converting the coefficient to an odds ratio by *exponentiating* it (e to the power of the coefficient). The function `exp()` does this in R. You can exponentiate the model coefficients with `exp(prize_binlm$coefficients)`. You would find that `exp(.113)` is about 1.12. Meaning that a 1 year increase in `years_edu` would predict a 12% increase in odds of receiving the prize (according to the model).  
- We can plot the model predictions with a few lines of code using `ggplot` (the y-axis is the predicted probability of `prize`):  

<button class="btn btn-primary" data-toggle="collapse" data-target="#step8b"> Show/Hide Code and Plot </button>  
<div id="step8b" class="collapse">  
```{r Step8b,fig.show='hold', results='hold'}
lumos_tib %>% drop_na(prize,years_edu) %>% 
  ggplot(aes(x=years_edu, y=prize)) + 
    geom_point(alpha=.25) +
    stat_smooth(method="glm", se=FALSE, method.args = list(family=binomial))
```
</div>
&nbsp;
- We didn't check the same sources of bias (non-linearity, heteroscedasticity, independence) because logistic regression does not have the same assumptions. Rather than linearity, logistic regression assumes that the relation of predictors to log(odds) of the outcome is linear. We should check for influential values and (if there are more than one predictor) multi-collinearity, however. Refer to the Field textbook (Chapter 20) for a complete overview of logistic regression. [This resource at STHDA is also useful](http://www.sthda.com/english/articles/36-classification-methods-essentials/148-logistic-regression-assumptions-and-diagnostics-in-r/#logistic-regression-diagnostics)  

#### That's all for this activity!

----------------------------------------------------------------------------

## References

- Chapters 9, 20 of textbook  
- Field, A.P. (2021). Discovering Statistics Using R and RStudio. Second. London: Sage.