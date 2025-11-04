## Comparing more than two groups in SPSS  

Bhanji - Statistical Methods  
*last edited Nov 3 2025*  

------------------------------------------------------------------------

## Goals for today  

Learn how to compare means of a continuous outcome from several independent groups to address the effect group on the dependent variable (modeling a continuous DV with a categorical IV). Extend this model to groups that differ on two factors (2x2 design).  

- between subjects ANOVA (1 factor, 3 levels): conduct a one-way analysis of variance    
    - Understand SS<sub>M</sub> (SS between), SS<sub>R</sub> (SS within), F-stat  
    - Effect size: R-squared for full model, partial	η<sup>2</sup> (eta-squared)  
- ANCOVA: add 1 continuous covariate  
    - understand independence of treatment and covariate  
    - understand assumption of homogenous regression slopes  
- Factorial ANOVA (2 factors, 3X2 levels)  



## Step 1 - Get organized, import data
- make a folder for today's activity  

- make a "data" folder (inside the project folder)  

- **Download this data file (save in the data folder):**  
    - [oveis-2020-recoded-data.csv](../data/oveis-2020-recoded-data.csv)  
    
- make an "spss" folder (inside the project folder) for your SPSS files  

- import the data: Open SPSS and use File -\> Import Data-\> CSV or Text Data  - now check the variable types and add labels if you wish.  

------------------------------------------------------------------------

![Decision process for comparing several independent groups (from Field textbook Chapter 12)](../images/anova-process.png)    

#### data description:  
`oveis-2020-recoded-data.csv` is shared data from the publication [Oveis, C., Gu, Y., Ocampo, J. M., Hangen, E. J., & Jamieson, J. P. (2020). Emotion regulation contagion: Stress reappraisal promotes challenge responses in teammates. Journal of Experimental Psychology: General, 149(11), 2187.](https://doi.org/10.1037/xge0000757) 

  - in this study, participants were randomly assigned to one of six cells in a 3 X 2 factorial design. The explanation here will be brief and we will use simplified labels for the conditions - please refer to the publication for a full study description.
  - DV and covariate:
      1. `card_out_react_prep` - Cardiac output during interaction with a partner (preparing for a marketing pitch about a hypothetical product). Higher cardiac output is thought to represent an interpretation of a stressor as a "challenge" (as opposed to a threat; e.g., [Mendes, et al., 2001](https://doi.org/10.1002/ejsp.80))  
      2. `prepIOS_cent` - closeness rating (mean centered). participants rated how connected they felt to their partner during the interaction (higher values indicate greater connectedness)  
  - IVs: 
      1. Emotion regulation condition (`emoreg_cond`) - participants were randomly assigned to one of three groups: control=no emotion regulation instruction, suppress=instructions to suppress displays of emotion, reappraise=instructions to think about feelings of stress/arousal as a helpful part of their body's response that prepares itself for a challenge  
      2. Instruction Target (`direct_cond`) - participants were randomly assigned to receive the emotion regulation instruction directly ("self" condition) or interact with a partner who received the emotion regulation instruction ("partner" condition - they never receive the instruction directly)  

------------------------------------------------------------------------


## Step 2 - descriptives   
Do the following:  
1. Make sure the columns that contain nominal vals are stored as nominal (`emoreg_cond` and `direct_cond`)   
2. Then get descriptives (and histogram) for the DV `card_out_react_prep`  organized by just the first IV `emoreg_cond` - use Analyze-\>Descriptive Statistics-\>Explore, and put the grouping variable ("emoreg_cond") in the Factor List.  
    - there are no missing cases in this data (this is a subset of the original dataset) but you should be mindful of missing cases when working with your own data   
3. We are going to start by considering only 1 of the categorical IVs, `emoreg_cond`. Check the distribution shape (for a normal-ish shape) and potential outliers for the DV (histogram and boxplot) 

Make note of anything noticeable you see in the data.   

------------------------------------------------------------------------

## Step 3 - ANOVA using GLM univariate    

#### Step 3.1: specify the model  
Let's fit a model with 1 categorical predictor `emoreg_cond` and one continuous DV `card_out_react_prep`.  

It is as simple as:  
1. Analyze-\>General Linear Model-\>Univariate    
2. `card_out_react_prep` as Dependent Variable  
3. `emoreg_cond` as Fixed Factor  
4. click on "Plots" and add a plot with `emoreg_cond` on the Horizontal Axis (be sure to click Add after you select it)  
5. Click on "EM Means" and add `emoreg_cond` to the "Display means for:" box  
6. Click on "options" and check the box for effect size estimates  
7. click OK  


##### Understanding the output :  
1. The "Tests of Between Subjects Effecs" table gives the overall model F-statistic (in the "corrected model" row), which is equal to the mean square between groups divided by the mean square within groups. In our regression examples we referred to this value more generally as the ratio of variance explained by the model to leftover error (residual variance). The F-stat is the same for the single predictor `emoreg_cond` because we only have one predictor    
2. The p-value for the F-stat tells you the probability of an F-statistic at least that large (mean differences that large) under the null hypothesis that there is no difference between emo_reg group means. When you report the F-stat you include the model (between groups) and residual (within groups) degrees of freedom, e.g.: F(2,231) = 9.324, p = .0001  
3. The partial eta-squared effect size can be interpreted as proportion of variance in the DV explained by the model (notice the value for the model is the same as the R-squared value below the table). When we talk about an effect of a factor with many levels this is often called the "omnibus effect" to refer to the effect across all levels of the factor.  
4. The F-stat and effect size do not tell you anything about the direction of the group effect, so we need to look at the "Mean" column in the "Estimated Marginal Means" table to see the pattern.  It is immediately apparent that the mean cardiac output in the "reappraise" group is highest.  
  
#### Step 3.4- check assumptions  
We are using the same modeling framework that we used in linear regression, so the assumptions concerning residuals (normality and homogeneity) are the same, and we can check them the same way. In SPSS when you are specifying the model, you can click the "Save" button and you will have the option to save the model residuals, which will then appear as a new variable in the file. Once it is in the file you can explore it (normality plots) as you like.  
SPSS also provides options to use statistical tests to check for unequal variance (under the Options button - we won't do that now).

------------------------------------------------------------------------

## Step 4 - ANCOVA - independent groups with a continuous covariate  
Now that we have seen how to model a continous DV with a categorical IV, we can extend the model. The variable `prepIOS_cent` is closeness ratings from each participant, rating how connected they felt to the partner they interacted with. By including `prepIOS_cent` as a covariate, we can see what effect a the emotion regulation variable has, adjusting for the effect of the covariate (described in full in Field textbook section 13.2).  
Using the same model building menu (Analyze-\>General Linear Model-\>Univariate), add `prepIOS_cent` in the Covariate box in the model. Keep everything else the same as before.  

##### Understanding the output  
The "Tests of Between Subjects Effects" table gives us an F-statistic for each predictor. The F-stat for the emotion regulation condition now tells us about the difference in group means, adjusting for closeness. The F-stat for closeness (`prepIOS_cent`) tells us about the covariance with the DV. 
Notice that model eta-squared (R-squared) has gone up a little (because the covariate explains some added variance), and the partial eta-squared for `emoreg_cond` has gone down a tiny bit because the covariate explains some of the variance that is shared by the main IV and DV.  
Notice that the "estimated marginal means" are a little bit different now becuase they are adjusted for the covariate.  

##### what do you conclude based on this model?
- does closeness influence cardiac output?

- When we include a covariate in a model like this we make an assumption that the covariate is independent of the main IV (if groups were randomly assigned this should be the case). We can check that assumption by fitting a separate model to predict the covariate by the IV (but we won't do that now). See the Field textbook Chapter 13 section 13.4 for more discussion.    

-----------------------------------------------------------------------------

## Step 5 - Factorial Anova  

In the data description there is actually a second factor we haven't considered yet: `direct_cond` with two levels. This variable indicates whether a participant received instructions to regulate their emotions directly ("self"), or whether they were exposed indirectly ("other") by interacting with a partner who had received emotion regulation instructions. The design is fully crossed, so there are a total of 6 (3*2) groups. Let's examine the full design now.  
We can hypothesize that there is an interaction of `emoreg_cond` and `direct_cond` such that the effect of `emoreg_cond` on cardiac output is greater when the instructions are received directly ("self" condition) compared to indirectly ("other" condition).


#### Step 5.1 - Factorial ANOVA    
Let's run the full model now (DV predicted by IV1 + IV2 + IV1*IV2):
1. Go to the same Analyze-\>General Linear Model-\>Univariate menu  
2. remove the covariate from the covariate box  
3. add `direct_cond` to the Fixed Factors list  
4. Click on "Model" and see that "full factorial" is pre-selected by default  
5. Click Plots, and add a plot with `emoreg_cond` on the "horizontal axis", and `direct_cond` on "Separate Lines". Remove the previous plot from the list of plots.  
6. click Options, verify that "effect sizes" is checked  


##### Check out the output      
1. We get an F-stat for each of the three predictors (`emoreg_cond`, `direct_cond` and their interaction `emoreg_cond:direct_cond`). If we had found a significant interaction, we would infer that the effect of emotion regulation depended on whether the instructions were received directly or just by interacting with a partner who received emotion regulation instructions.  
3. We do find a significant main effect of emotion regulation condition (consistent with what we saw earlier). The column "partial eta-squared" gives an effect size for each predictor.  

#### Step 5.3 - Simple effects  
- Although there is no interaction, we might want to confirm that there is a significant emotion regulation effect within each level of the `direct_cond` factor. It is interesting if there is an effect just by interacting with a person who is regulating emotions, so we should confirm that.  
- We can run simple effects tests, which just means testing the effect of one factor at each level of the second factor  
- You can run these tests through the "EM Means" button in the model builder. The output is pretty gnarly so we will look at it together as a class on screen.  
- The Post-hoc button will also allow you to compare pairs of means. In this case one of our factors, `direct_cond` only has two levels, so if we try to include it through this menu we will get a message that the variable is excluded from Post-hoc tests because there are less than three levels. If we put `emoreg_cond` into the post-hoc test then we will get comparisons between means of the emotion regulation conditions, collapsing across the other factor.  


----------------------------------------------------------------------------
#### That's all for this activity!

----------------------------------------------------------------------------

## References

- Chapters 12, 13, 14 of Field textbook: Field, A.P. (2018). Discovering Statistics Using IBM SPSS Statistics. 5th Edition. London: Sage.  
