---
title: 'Open-Ended Registration: Modelling the Development of Preschoolers’ Fairness
  Judgements'
author: "Peidong Mei"
date: "15/03/2021"
output:
  pdf_document: default
  html_document: default
---

<style type="text/css">
body, td {font-size: 14px;}
code.r{font-size: 14px;}
pre {font-size: 12px}
</style>



## 1 Study Information

#### 1.1 Title 

The Cost of Fairness: Its Dynamic Influence on Developmental Stage, Competition and Relation in British and Chinese Preschoolers’ Resource Allocations.

#### 1.2 Authorship

Peidong Mei & Charlie Lewis ^1^ 

^1^ Lancaster University 

#### 1.3 Description

Several factors influence children’s resource distribution with a peer. This study compared 104 3-5-year-olds with 100 6-7-year-olds’ allocations of rewards within repeated trials in which three structural factors (age, gender and culture: the UK vs China) and three contextual manipulations (whether a trial involved competition, was with a friend or an unknown peer, or whether equal allocation incurred a cost) were examined together. Bayesian multinomial modelling revealed a complexity of interactions between the factors, with the cost of losing part of the reward central to these. A dynamic cost model is proposed to account for developmental processes in which children’s growing sense of fairness consists is balanced against influences which persuade the child to make selfish or selfless allocations.  

#### 1.4 License
Not applicable.

#### 1.5 Tags 
- Fairness
- Resource Allocation
- Cross-cultural Comparison 
- Early Moral Development
- Bayesian Multinomial Modelling
- R package 'brms' 


#### 1.6 Summary
This project was registered after the study had been completed. We want to include a full picture of the complex modelling employed in the study.

This registration involves the dataset description and the analysis process. In the dataset section, we provided detailed definitions of all variables with their specific levels. A descriptive summary of the dataset can also be found. The analysis process includes the R package we used for modelling, prior and baseline information, model construction and selection. Output summaries of all the models that we run were also included. 



## 2 Dataset Description

Our independent variables involoved three within subject variables: 
- **`cost`**(<u>cost</u> vs cost-free)
- **`competition`**(<u>competition</u> vs free play) 
- **`relation`** (<u>friend</u> vs unknown peer) 

We also collected three between subject variables: 
- **`country`**(<u>China</u> vs UK)
- **`age group`**(<u>Older</u> vs Younger)
- **`gender`**(<u>Female</u> vs Male) 

These are all binary and the former level in each bracket was used as the baseline in following models. 

The dependent variable is **`allocation`** and has three levels: fair (equal split), unfair but advantageous (the distributor gets more) and unfair but disadvantageous (the distributor gets less). 



```r
summary(Subject)
```

```
##     child                age          age_group   country     gender   
##  Length:204         Min.   :38.00   Older  : 96   CN:104   Female: 87  
##  Class :character   1st Qu.:53.00   Younger:108   UK:100   Male  :117  
##  Mode  :character   Median :62.00                                      
##                     Mean   :61.64                                      
##                     3rd Qu.:72.00                                      
##                     Max.   :82.00
```

```r
summary(Response)
```

```
##    fairness    allocation       justification
##  Fair  :1052   Fair :1052   Desire     :457  
##  Unfair: 580   UFad : 425   Norm       :932  
##                UFdis: 155   Affiliation:135  
##                             Conduct    : 52  
##                             Uncodable  : 56
```

## 3 Statistical Analysis 
#### 3.1  Modelling Process
We used [**`brm()`**](https://cran.r-project.org/web/packages/brms/brms.pdf), which is a Bayesian Regression Modelling approach using the platform *'Stan'* to explore the predictive effects of all six factors and their possible interactions on children's allocations.

The *default priors* were used for two reasons:  first, model diagnosis indicated that the default priors were vague enough to avoid any substantial effects on the model and allowed the dataset to be converted well; secondly, to the best of our knowledge, there were no solid grounds from previous research on this topic available for us to consider more beneficial priors. 

In addition, the response baseline used in the modelling, also known as the referential information to posterior distribution, was the ‘fair’ response from the allocation measurement (i.e. the child and recipeint both get the same amount). The ‘fair’ response was predominant (64.46%), compared to an ‘advantage-self’ response of 26.04% and a ‘disadvantage-self’ response of 9.5%. Taking the majority response as the baseline can favour the stability and accuracy of these models. 


```r
######### reparameterize baseline #########
R2long$age_group=relevel(R2long$age_group, ref='Older')
R2long$country=relevel(R2long$country, ref='CN')
R2long$gender=relevel(R2long$gender, ref='Female')
R2long$cost=relevel(R2long$cost, ref='Cost')
R2long$competition=relevel(R2long$competition, ref='Competition')
R2long$relation=relevel(R2long$relation, ref='Friend')
R2long$allocation<-factor(R2long$allocation,
                          levels=c("Fair","UFad","UFdis"))
```

Six models were computed using the bottom-up approach: starting with the simplest (Model 1) with only the main effect of each variable, then the following models added interactions from 2-way (Model 2) up to  6-way (Model 6). All the models included a random intercept of subject to minimize any participant differences and were fitted using four chains, each with 2,000 iterations (1,000 warm up) under the family setting “categorical’. 


```r
##################### just 6 main effects
brm2.1 <- brm(allocation~age_group+gender+country+competition+relation+cost
              +(1|id),data=R2long,family=categorical,save_all_pars=T)

##################### add all 6 effects with 2way interactions
brm2.2 <- brm(allocation~(age_group+gender+country+competition+relation+cost)^2
              +(1|id),data=R2long,family=categorical,save_all_pars=T)

##################### add all 6 effects with 3way interactions
brm2.3 <- brm(allocation~(age_group+gender+country+competition+relation+cost)^3
              +(1|id),data=R2long,family=categorical,save_all_pars=T)

##################### add all 6 effects with 4way interactions
brm2.4 <- brm(allocation~(age_group+gender+country+competition+relation+cost)^4
              +(1|id),data=R2long,family=categorical,save_all_pars=T)

##################### add all 6 effects with 5way interactions
brm2.5 <- brm(allocation~(age_group+gender+country+competition+relation+cost)^5
              +(1|id),data=R2long,family=categorical)

##################### add all 6 effects with 6way interactions
brm2.6 <- brm(allocation~(age_group+gender+country+competition+relation+cost)^6
              +(1|id),data=R2long,family=categorical,save_all_pars=T)
```

#### 3.2 Model outputs 

As the Models involving five and six-way interactions *brm2.5* and *brm2.6* didnt converged, they were excluded from any further anylysis. 
1: in model **`brm2.1`**, we add all 6 effects and their 2-way interactions.


```r
summary(brm2.1) 
```

```
##  Family: categorical 
##   Links: muUFad = logit; muUFdis = logit 
## Formula: allocation ~ age_group + gender + country + competition + relation + cost + (1 | id) 
##    Data: R2long (Number of observations: 1632) 
## Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
##          total post-warmup samples = 4000
## 
## Group-Level Effects: 
## ~id (Number of levels: 204) 
##                       Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS
## sd(muUFad_Intercept)      1.18      0.12     0.96     1.44 1.00     1550
## sd(muUFdis_Intercept)     0.89      0.17     0.56     1.22 1.00     1004
##                       Tail_ESS
## sd(muUFad_Intercept)      2606
## sd(muUFdis_Intercept)     1599
## 
## Population-Level Effects: 
##                                  Estimate Est.Error l-95% CI u-95% CI Rhat
## muUFad_Intercept                    -0.47      0.26    -0.97     0.03 1.00
## muUFdis_Intercept                   -2.22      0.30    -2.82    -1.66 1.00
## muUFad_age_groupYounger             -0.03      0.22    -0.46     0.39 1.00
## muUFad_genderMale                    0.19      0.22    -0.25     0.61 1.00
## muUFad_countryUK                     0.48      0.22     0.05     0.91 1.00
## muUFad_competitionNOcompetition     -0.24      0.13    -0.50     0.01 1.00
## muUFad_relationUnknownMpeer         -0.29      0.14    -0.55    -0.02 1.00
## muUFad_costNOcost                   -1.85      0.15    -2.14    -1.56 1.00
## muUFdis_age_groupYounger             0.59      0.23     0.14     1.05 1.00
## muUFdis_genderMale                   0.21      0.24    -0.24     0.68 1.00
## muUFdis_countryUK                   -0.49      0.23    -0.95    -0.04 1.00
## muUFdis_competitionNOcompetition    -0.01      0.18    -0.36     0.35 1.00
## muUFdis_relationUnknownMpeer        -0.15      0.18    -0.50     0.20 1.00
## muUFdis_costNOcost                  -0.33      0.18    -0.68     0.02 1.00
##                                  Bulk_ESS Tail_ESS
## muUFad_Intercept                     2429     2823
## muUFdis_Intercept                    3755     3089
## muUFad_age_groupYounger              2257     2638
## muUFad_genderMale                    1969     2539
## muUFad_countryUK                     1975     2905
## muUFad_competitionNOcompetition      7213     3035
## muUFad_relationUnknownMpeer          6345     3075
## muUFad_costNOcost                    4832     2844
## muUFdis_age_groupYounger             3155     3059
## muUFdis_genderMale                   3375     3164
## muUFdis_countryUK                    3957     3235
## muUFdis_competitionNOcompetition     6542     2951
## muUFdis_relationUnknownMpeer         7003     2905
## muUFdis_costNOcost                   7091     3051
## 
## Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
## and Tail_ESS are effective sample size measures, and Rhat is the potential
## scale reduction factor on split chains (at convergence, Rhat = 1).
```

2: in model **`brm2.2`**, we add all 6 effects and their 2-way interactions.


```r
summary(brm2.2)
```

```
##  Family: categorical 
##   Links: muUFad = logit; muUFdis = logit 
## Formula: allocation ~ (age_group + gender + country + competition + relation + cost)^2 + (1 | id) 
##    Data: R2long (Number of observations: 1632) 
## Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
##          total post-warmup samples = 4000
## 
## Group-Level Effects: 
## ~id (Number of levels: 204) 
##                       Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS
## sd(muUFad_Intercept)      1.24      0.13     1.00     1.52 1.00     1847
## sd(muUFdis_Intercept)     0.98      0.18     0.64     1.33 1.00     1162
##                       Tail_ESS
## sd(muUFad_Intercept)      2790
## sd(muUFdis_Intercept)     2136
## 
## Population-Level Effects: 
##                                                       Estimate Est.Error
## muUFad_Intercept                                         -0.36      0.40
## muUFdis_Intercept                                        -1.67      0.48
## muUFad_age_groupYounger                                  -0.48      0.49
## muUFad_genderMale                                         0.34      0.47
## muUFad_countryUK                                          1.73      0.48
## muUFad_competitionNOcompetition                          -0.66      0.34
## muUFad_relationUnknownMpeer                              -0.80      0.33
## muUFad_costNOcost                                        -2.47      0.39
## muUFad_age_groupYounger:genderMale                        0.36      0.48
## muUFad_age_groupYounger:countryUK                        -0.25      0.46
## muUFad_age_groupYounger:competitionNOcompetition         -0.12      0.27
## muUFad_age_groupYounger:relationUnknownMpeer              0.35      0.27
## muUFad_age_groupYounger:costNOcost                        0.55      0.29
## muUFad_genderMale:countryUK                              -0.99      0.46
## muUFad_genderMale:competitionNOcompetition                0.27      0.30
## muUFad_genderMale:relationUnknownMpeer                   -0.04      0.28
## muUFad_genderMale:costNOcost                              0.07      0.31
## muUFad_countryUK:competitionNOcompetition                -0.31      0.27
## muUFad_countryUK:relationUnknownMpeer                    -0.20      0.27
## muUFad_countryUK:costNOcost                              -0.75      0.30
## muUFad_competitionNOcompetition:relationUnknownMpeer      0.46      0.28
## muUFad_competitionNOcompetition:costNOcost                0.66      0.29
## muUFad_relationUnknownMpeer:costNOcost                    0.56      0.30
## muUFdis_age_groupYounger                                 -0.37      0.58
## muUFdis_genderMale                                        0.07      0.52
## muUFdis_countryUK                                         0.50      0.57
## muUFdis_competitionNOcompetition                         -0.11      0.45
## muUFdis_relationUnknownMpeer                             -1.02      0.49
## muUFdis_costNOcost                                       -1.73      0.51
## muUFdis_age_groupYounger:genderMale                       0.65      0.53
## muUFdis_age_groupYounger:countryUK                       -1.20      0.53
## muUFdis_age_groupYounger:competitionNOcompetition         0.61      0.41
## muUFdis_age_groupYounger:relationUnknownMpeer             0.54      0.41
## muUFdis_age_groupYounger:costNOcost                       1.10      0.41
## muUFdis_genderMale:countryUK                             -0.19      0.52
## muUFdis_genderMale:competitionNOcompetition              -0.76      0.40
## muUFdis_genderMale:relationUnknownMpeer                   0.04      0.39
## muUFdis_genderMale:costNOcost                             0.36      0.40
## muUFdis_countryUK:competitionNOcompetition               -0.63      0.39
## muUFdis_countryUK:relationUnknownMpeer                   -0.08      0.38
## muUFdis_countryUK:costNOcost                              0.40      0.39
## muUFdis_competitionNOcompetition:relationUnknownMpeer     0.60      0.40
## muUFdis_competitionNOcompetition:costNOcost               0.24      0.38
## muUFdis_relationUnknownMpeer:costNOcost                   0.44      0.39
##                                                       l-95% CI u-95% CI Rhat
## muUFad_Intercept                                         -1.14     0.41 1.00
## muUFdis_Intercept                                        -2.66    -0.80 1.00
## muUFad_age_groupYounger                                  -1.42     0.48 1.00
## muUFad_genderMale                                        -0.59     1.26 1.00
## muUFad_countryUK                                          0.82     2.66 1.00
## muUFad_competitionNOcompetition                          -1.31    -0.00 1.00
## muUFad_relationUnknownMpeer                              -1.48    -0.14 1.00
## muUFad_costNOcost                                        -3.25    -1.73 1.00
## muUFad_age_groupYounger:genderMale                       -0.59     1.32 1.00
## muUFad_age_groupYounger:countryUK                        -1.15     0.64 1.00
## muUFad_age_groupYounger:competitionNOcompetition         -0.65     0.45 1.00
## muUFad_age_groupYounger:relationUnknownMpeer             -0.18     0.86 1.00
## muUFad_age_groupYounger:costNOcost                       -0.01     1.14 1.00
## muUFad_genderMale:countryUK                              -1.90    -0.11 1.00
## muUFad_genderMale:competitionNOcompetition               -0.30     0.84 1.00
## muUFad_genderMale:relationUnknownMpeer                   -0.62     0.51 1.00
## muUFad_genderMale:costNOcost                             -0.51     0.69 1.00
## muUFad_countryUK:competitionNOcompetition                -0.83     0.22 1.00
## muUFad_countryUK:relationUnknownMpeer                    -0.73     0.31 1.00
## muUFad_countryUK:costNOcost                              -1.34    -0.16 1.00
## muUFad_competitionNOcompetition:relationUnknownMpeer     -0.06     1.00 1.00
## muUFad_competitionNOcompetition:costNOcost                0.09     1.22 1.00
## muUFad_relationUnknownMpeer:costNOcost                   -0.04     1.15 1.00
## muUFdis_age_groupYounger                                 -1.49     0.77 1.00
## muUFdis_genderMale                                       -0.94     1.07 1.00
## muUFdis_countryUK                                        -0.62     1.59 1.00
## muUFdis_competitionNOcompetition                         -0.98     0.77 1.00
## muUFdis_relationUnknownMpeer                             -1.98    -0.04 1.00
## muUFdis_costNOcost                                       -2.75    -0.74 1.00
## muUFdis_age_groupYounger:genderMale                      -0.36     1.69 1.00
## muUFdis_age_groupYounger:countryUK                       -2.26    -0.19 1.00
## muUFdis_age_groupYounger:competitionNOcompetition        -0.18     1.42 1.00
## muUFdis_age_groupYounger:relationUnknownMpeer            -0.26     1.35 1.00
## muUFdis_age_groupYounger:costNOcost                       0.30     1.91 1.00
## muUFdis_genderMale:countryUK                             -1.22     0.84 1.00
## muUFdis_genderMale:competitionNOcompetition              -1.56     0.02 1.00
## muUFdis_genderMale:relationUnknownMpeer                  -0.73     0.80 1.00
## muUFdis_genderMale:costNOcost                            -0.44     1.13 1.00
## muUFdis_countryUK:competitionNOcompetition               -1.41     0.13 1.00
## muUFdis_countryUK:relationUnknownMpeer                   -0.83     0.67 1.00
## muUFdis_countryUK:costNOcost                             -0.37     1.17 1.00
## muUFdis_competitionNOcompetition:relationUnknownMpeer    -0.19     1.38 1.00
## muUFdis_competitionNOcompetition:costNOcost              -0.51     0.98 1.00
## muUFdis_relationUnknownMpeer:costNOcost                  -0.33     1.18 1.00
##                                                       Bulk_ESS Tail_ESS
## muUFad_Intercept                                          2247     2664
## muUFdis_Intercept                                         2876     2701
## muUFad_age_groupYounger                                   2206     2575
## muUFad_genderMale                                         2259     2660
## muUFad_countryUK                                          2010     2820
## muUFad_competitionNOcompetition                           3580     2843
## muUFad_relationUnknownMpeer                               3739     3250
## muUFad_costNOcost                                         3560     3024
## muUFad_age_groupYounger:genderMale                        2063     2286
## muUFad_age_groupYounger:countryUK                         2059     2826
## muUFad_age_groupYounger:competitionNOcompetition          6699     2757
## muUFad_age_groupYounger:relationUnknownMpeer              6834     3352
## muUFad_age_groupYounger:costNOcost                        5766     3031
## muUFad_genderMale:countryUK                               2425     2690
## muUFad_genderMale:competitionNOcompetition                5166     2956
## muUFad_genderMale:relationUnknownMpeer                    5485     3237
## muUFad_genderMale:costNOcost                              6504     3034
## muUFad_countryUK:competitionNOcompetition                 5925     2875
## muUFad_countryUK:relationUnknownMpeer                     6402     3013
## muUFad_countryUK:costNOcost                               6194     3218
## muUFad_competitionNOcompetition:relationUnknownMpeer      5717     3250
## muUFad_competitionNOcompetition:costNOcost                6051     2949
## muUFad_relationUnknownMpeer:costNOcost                    6046     2749
## muUFdis_age_groupYounger                                  2949     2936
## muUFdis_genderMale                                        2969     2815
## muUFdis_countryUK                                         3066     2920
## muUFdis_competitionNOcompetition                          3983     3021
## muUFdis_relationUnknownMpeer                              4157     2727
## muUFdis_costNOcost                                        3611     3190
## muUFdis_age_groupYounger:genderMale                       3254     2981
## muUFdis_age_groupYounger:countryUK                        2774     3074
## muUFdis_age_groupYounger:competitionNOcompetition         5088     3154
## muUFdis_age_groupYounger:relationUnknownMpeer             6159     2986
## muUFdis_age_groupYounger:costNOcost                       5289     3372
## muUFdis_genderMale:countryUK                              2972     2996
## muUFdis_genderMale:competitionNOcompetition               5443     3062
## muUFdis_genderMale:relationUnknownMpeer                   5016     2957
## muUFdis_genderMale:costNOcost                             5936     2741
## muUFdis_countryUK:competitionNOcompetition                5390     3069
## muUFdis_countryUK:relationUnknownMpeer                    6341     3333
## muUFdis_countryUK:costNOcost                              5292     3176
## muUFdis_competitionNOcompetition:relationUnknownMpeer     5448     2984
## muUFdis_competitionNOcompetition:costNOcost               5380     3225
## muUFdis_relationUnknownMpeer:costNOcost                   6159     3184
## 
## Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
## and Tail_ESS are effective sample size measures, and Rhat is the potential
## scale reduction factor on split chains (at convergence, Rhat = 1).
```

3: in model **`brm2.3`**, we add all 6 effects and their 3-way interactions.


```r
summary(brm2.3)
```

```
##  Family: categorical 
##   Links: muUFad = logit; muUFdis = logit 
## Formula: allocation ~ (age_group + gender + country + competition + relation + cost)^3 + (1 | id) 
##    Data: R2long (Number of observations: 1632) 
## Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
##          total post-warmup samples = 4000
## 
## Group-Level Effects: 
## ~id (Number of levels: 204) 
##                       Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS
## sd(muUFad_Intercept)      1.33      0.14     1.07     1.61 1.00     1556
## sd(muUFdis_Intercept)     1.05      0.19     0.71     1.44 1.00     1368
##                       Tail_ESS
## sd(muUFad_Intercept)      2339
## sd(muUFdis_Intercept)     2251
## 
## Population-Level Effects: 
##                                                                        Estimate
## muUFad_Intercept                                                           0.11
## muUFdis_Intercept                                                         -1.54
## muUFad_age_groupYounger                                                   -1.13
## muUFad_genderMale                                                          0.16
## muUFad_countryUK                                                           1.00
## muUFad_competitionNOcompetition                                           -1.11
## muUFad_relationUnknownMpeer                                               -1.51
## muUFad_costNOcost                                                         -3.07
## muUFad_age_groupYounger:genderMale                                         0.17
## muUFad_age_groupYounger:countryUK                                          0.39
## muUFad_age_groupYounger:competitionNOcompetition                           0.41
## muUFad_age_groupYounger:relationUnknownMpeer                               1.15
## muUFad_age_groupYounger:costNOcost                                         1.99
## muUFad_genderMale:countryUK                                               -0.71
## muUFad_genderMale:competitionNOcompetition                                 0.52
## muUFad_genderMale:relationUnknownMpeer                                     0.60
## muUFad_genderMale:costNOcost                                               0.30
## muUFad_countryUK:competitionNOcompetition                                  0.14
## muUFad_countryUK:relationUnknownMpeer                                      1.11
## muUFad_countryUK:costNOcost                                               -0.18
## muUFad_competitionNOcompetition:relationUnknownMpeer                       0.70
## muUFad_competitionNOcompetition:costNOcost                                 1.09
## muUFad_relationUnknownMpeer:costNOcost                                    -0.42
## muUFad_age_groupYounger:genderMale:countryUK                               1.15
## muUFad_age_groupYounger:genderMale:competitionNOcompetition               -0.54
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                    0.24
## muUFad_age_groupYounger:genderMale:costNOcost                             -0.90
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                -0.45
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                    -1.33
## muUFad_age_groupYounger:countryUK:costNOcost                              -1.35
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer     -0.15
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                0.15
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                   -0.47
## muUFad_genderMale:countryUK:competitionNOcompetition                       0.04
## muUFad_genderMale:countryUK:relationUnknownMpeer                          -1.85
## muUFad_genderMale:countryUK:costNOcost                                    -0.19
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer           -0.17
## muUFad_genderMale:competitionNOcompetition:costNOcost                      0.06
## muUFad_genderMale:relationUnknownMpeer:costNOcost                          0.73
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer             0.05
## muUFad_countryUK:competitionNOcompetition:costNOcost                      -0.95
## muUFad_countryUK:relationUnknownMpeer:costNOcost                           1.51
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost           -0.06
## muUFdis_age_groupYounger                                                  -1.81
## muUFdis_genderMale                                                         0.10
## muUFdis_countryUK                                                         -0.43
## muUFdis_competitionNOcompetition                                           0.36
## muUFdis_relationUnknownMpeer                                              -0.76
## muUFdis_costNOcost                                                        -2.74
## muUFdis_age_groupYounger:genderMale                                        1.82
## muUFdis_age_groupYounger:countryUK                                         0.04
## muUFdis_age_groupYounger:competitionNOcompetition                          0.95
## muUFdis_age_groupYounger:relationUnknownMpeer                              1.32
## muUFdis_age_groupYounger:costNOcost                                        3.67
## muUFdis_genderMale:countryUK                                               0.85
## muUFdis_genderMale:competitionNOcompetition                               -1.63
## muUFdis_genderMale:relationUnknownMpeer                                   -1.82
## muUFdis_genderMale:costNOcost                                              1.03
## muUFdis_countryUK:competitionNOcompetition                                -0.99
## muUFdis_countryUK:relationUnknownMpeer                                     0.69
## muUFdis_countryUK:costNOcost                                               1.28
## muUFdis_competitionNOcompetition:relationUnknownMpeer                     -0.30
## muUFdis_competitionNOcompetition:costNOcost                                0.00
## muUFdis_relationUnknownMpeer:costNOcost                                    0.65
## muUFdis_age_groupYounger:genderMale:countryUK                             -0.66
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition              -0.13
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                   1.14
## muUFdis_age_groupYounger:genderMale:costNOcost                            -2.29
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition               -0.09
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                   -1.43
## muUFdis_age_groupYounger:countryUK:costNOcost                             -0.63
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer     0.46
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost              -0.48
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                  -1.70
## muUFdis_genderMale:countryUK:competitionNOcompetition                      0.16
## muUFdis_genderMale:countryUK:relationUnknownMpeer                         -0.18
## muUFdis_genderMale:countryUK:costNOcost                                   -1.06
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer           0.49
## muUFdis_genderMale:competitionNOcompetition:costNOcost                     0.96
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                         1.42
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer            0.66
## muUFdis_countryUK:competitionNOcompetition:costNOcost                     -0.01
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                          0.01
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost           0.02
##                                                                        Est.Error
## muUFad_Intercept                                                            0.52
## muUFdis_Intercept                                                           0.66
## muUFad_age_groupYounger                                                     0.71
## muUFad_genderMale                                                           0.65
## muUFad_countryUK                                                            0.70
## muUFad_competitionNOcompetition                                             0.57
## muUFad_relationUnknownMpeer                                                 0.58
## muUFad_costNOcost                                                           0.68
## muUFad_age_groupYounger:genderMale                                          0.84
## muUFad_age_groupYounger:countryUK                                           0.87
## muUFad_age_groupYounger:competitionNOcompetition                            0.66
## muUFad_age_groupYounger:relationUnknownMpeer                                0.67
## muUFad_age_groupYounger:costNOcost                                          0.73
## muUFad_genderMale:countryUK                                                 0.84
## muUFad_genderMale:competitionNOcompetition                                  0.64
## muUFad_genderMale:relationUnknownMpeer                                      0.63
## muUFad_genderMale:costNOcost                                                0.72
## muUFad_countryUK:competitionNOcompetition                                   0.64
## muUFad_countryUK:relationUnknownMpeer                                       0.64
## muUFad_countryUK:costNOcost                                                 0.74
## muUFad_competitionNOcompetition:relationUnknownMpeer                        0.66
## muUFad_competitionNOcompetition:costNOcost                                  0.76
## muUFad_relationUnknownMpeer:costNOcost                                      0.79
## muUFad_age_groupYounger:genderMale:countryUK                                1.00
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                 0.59
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                     0.59
## muUFad_age_groupYounger:genderMale:costNOcost                               0.63
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                  0.57
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                      0.58
## muUFad_age_groupYounger:countryUK:costNOcost                                0.64
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer       0.58
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                 0.61
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                     0.59
## muUFad_genderMale:countryUK:competitionNOcompetition                        0.59
## muUFad_genderMale:countryUK:relationUnknownMpeer                            0.58
## muUFad_genderMale:countryUK:costNOcost                                      0.65
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer             0.58
## muUFad_genderMale:competitionNOcompetition:costNOcost                       0.62
## muUFad_genderMale:relationUnknownMpeer:costNOcost                           0.63
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer              0.57
## muUFad_countryUK:competitionNOcompetition:costNOcost                        0.62
## muUFad_countryUK:relationUnknownMpeer:costNOcost                            0.61
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost             0.61
## muUFdis_age_groupYounger                                                    0.99
## muUFdis_genderMale                                                          0.84
## muUFdis_countryUK                                                           0.99
## muUFdis_competitionNOcompetition                                            0.74
## muUFdis_relationUnknownMpeer                                                0.81
## muUFdis_costNOcost                                                          1.01
## muUFdis_age_groupYounger:genderMale                                         1.08
## muUFdis_age_groupYounger:countryUK                                          1.16
## muUFdis_age_groupYounger:competitionNOcompetition                           0.97
## muUFdis_age_groupYounger:relationUnknownMpeer                               0.99
## muUFdis_age_groupYounger:costNOcost                                         1.10
## muUFdis_genderMale:countryUK                                                1.12
## muUFdis_genderMale:competitionNOcompetition                                 0.95
## muUFdis_genderMale:relationUnknownMpeer                                     0.99
## muUFdis_genderMale:costNOcost                                               1.02
## muUFdis_countryUK:competitionNOcompetition                                  1.00
## muUFdis_countryUK:relationUnknownMpeer                                      1.00
## muUFdis_countryUK:costNOcost                                                1.08
## muUFdis_competitionNOcompetition:relationUnknownMpeer                       0.95
## muUFdis_competitionNOcompetition:costNOcost                                 1.05
## muUFdis_relationUnknownMpeer:costNOcost                                     1.09
## muUFdis_age_groupYounger:genderMale:countryUK                               1.15
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                0.93
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                    0.93
## muUFdis_age_groupYounger:genderMale:costNOcost                              0.98
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                 0.90
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                     0.90
## muUFdis_age_groupYounger:countryUK:costNOcost                               0.89
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer      0.89
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                0.94
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                    0.92
## muUFdis_genderMale:countryUK:competitionNOcompetition                       0.86
## muUFdis_genderMale:countryUK:relationUnknownMpeer                           0.87
## muUFdis_genderMale:countryUK:costNOcost                                     0.90
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer            0.83
## muUFdis_genderMale:competitionNOcompetition:costNOcost                      0.88
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                          0.86
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer             0.84
## muUFdis_countryUK:competitionNOcompetition:costNOcost                       0.86
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                           0.84
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost            0.82
##                                                                        l-95% CI
## muUFad_Intercept                                                          -0.90
## muUFdis_Intercept                                                         -2.89
## muUFad_age_groupYounger                                                   -2.56
## muUFad_genderMale                                                         -1.11
## muUFad_countryUK                                                          -0.39
## muUFad_competitionNOcompetition                                           -2.24
## muUFad_relationUnknownMpeer                                               -2.65
## muUFad_costNOcost                                                         -4.41
## muUFad_age_groupYounger:genderMale                                        -1.45
## muUFad_age_groupYounger:countryUK                                         -1.39
## muUFad_age_groupYounger:competitionNOcompetition                          -0.87
## muUFad_age_groupYounger:relationUnknownMpeer                              -0.12
## muUFad_age_groupYounger:costNOcost                                         0.55
## muUFad_genderMale:countryUK                                               -2.38
## muUFad_genderMale:competitionNOcompetition                                -0.75
## muUFad_genderMale:relationUnknownMpeer                                    -0.65
## muUFad_genderMale:costNOcost                                              -1.12
## muUFad_countryUK:competitionNOcompetition                                 -1.08
## muUFad_countryUK:relationUnknownMpeer                                     -0.12
## muUFad_countryUK:costNOcost                                               -1.59
## muUFad_competitionNOcompetition:relationUnknownMpeer                      -0.56
## muUFad_competitionNOcompetition:costNOcost                                -0.36
## muUFad_relationUnknownMpeer:costNOcost                                    -2.06
## muUFad_age_groupYounger:genderMale:countryUK                              -0.76
## muUFad_age_groupYounger:genderMale:competitionNOcompetition               -1.70
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                   -0.91
## muUFad_age_groupYounger:genderMale:costNOcost                             -2.14
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                -1.57
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                    -2.50
## muUFad_age_groupYounger:countryUK:costNOcost                              -2.62
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer     -1.26
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost               -1.07
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                   -1.62
## muUFad_genderMale:countryUK:competitionNOcompetition                      -1.12
## muUFad_genderMale:countryUK:relationUnknownMpeer                          -3.00
## muUFad_genderMale:countryUK:costNOcost                                    -1.47
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer           -1.30
## muUFad_genderMale:competitionNOcompetition:costNOcost                     -1.18
## muUFad_genderMale:relationUnknownMpeer:costNOcost                         -0.48
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer            -1.06
## muUFad_countryUK:competitionNOcompetition:costNOcost                      -2.17
## muUFad_countryUK:relationUnknownMpeer:costNOcost                           0.34
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost           -1.23
## muUFdis_age_groupYounger                                                  -3.80
## muUFdis_genderMale                                                        -1.53
## muUFdis_countryUK                                                         -2.44
## muUFdis_competitionNOcompetition                                          -1.11
## muUFdis_relationUnknownMpeer                                              -2.38
## muUFdis_costNOcost                                                        -4.83
## muUFdis_age_groupYounger:genderMale                                       -0.25
## muUFdis_age_groupYounger:countryUK                                        -2.23
## muUFdis_age_groupYounger:competitionNOcompetition                         -0.96
## muUFdis_age_groupYounger:relationUnknownMpeer                             -0.60
## muUFdis_age_groupYounger:costNOcost                                        1.64
## muUFdis_genderMale:countryUK                                              -1.31
## muUFdis_genderMale:competitionNOcompetition                               -3.51
## muUFdis_genderMale:relationUnknownMpeer                                   -3.78
## muUFdis_genderMale:costNOcost                                             -0.94
## muUFdis_countryUK:competitionNOcompetition                                -2.95
## muUFdis_countryUK:relationUnknownMpeer                                    -1.24
## muUFdis_countryUK:costNOcost                                              -0.83
## muUFdis_competitionNOcompetition:relationUnknownMpeer                     -2.15
## muUFdis_competitionNOcompetition:costNOcost                               -2.10
## muUFdis_relationUnknownMpeer:costNOcost                                   -1.48
## muUFdis_age_groupYounger:genderMale:countryUK                             -2.91
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition              -1.98
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                  -0.65
## muUFdis_age_groupYounger:genderMale:costNOcost                            -4.24
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition               -1.81
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                   -3.20
## muUFdis_age_groupYounger:countryUK:costNOcost                             -2.42
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer    -1.27
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost              -2.32
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                  -3.51
## muUFdis_genderMale:countryUK:competitionNOcompetition                     -1.58
## muUFdis_genderMale:countryUK:relationUnknownMpeer                         -1.91
## muUFdis_genderMale:countryUK:costNOcost                                   -2.86
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer          -1.16
## muUFdis_genderMale:competitionNOcompetition:costNOcost                    -0.73
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                        -0.24
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer           -0.98
## muUFdis_countryUK:competitionNOcompetition:costNOcost                     -1.64
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                         -1.62
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost          -1.61
##                                                                        u-95% CI
## muUFad_Intercept                                                           1.12
## muUFdis_Intercept                                                         -0.29
## muUFad_age_groupYounger                                                    0.29
## muUFad_genderMale                                                          1.43
## muUFad_countryUK                                                           2.41
## muUFad_competitionNOcompetition                                            0.00
## muUFad_relationUnknownMpeer                                               -0.38
## muUFad_costNOcost                                                         -1.76
## muUFad_age_groupYounger:genderMale                                         1.80
## muUFad_age_groupYounger:countryUK                                          2.08
## muUFad_age_groupYounger:competitionNOcompetition                           1.75
## muUFad_age_groupYounger:relationUnknownMpeer                               2.48
## muUFad_age_groupYounger:costNOcost                                         3.43
## muUFad_genderMale:countryUK                                                0.89
## muUFad_genderMale:competitionNOcompetition                                 1.77
## muUFad_genderMale:relationUnknownMpeer                                     1.88
## muUFad_genderMale:costNOcost                                               1.74
## muUFad_countryUK:competitionNOcompetition                                  1.41
## muUFad_countryUK:relationUnknownMpeer                                      2.38
## muUFad_countryUK:costNOcost                                                1.27
## muUFad_competitionNOcompetition:relationUnknownMpeer                       2.03
## muUFad_competitionNOcompetition:costNOcost                                 2.62
## muUFad_relationUnknownMpeer:costNOcost                                     1.08
## muUFad_age_groupYounger:genderMale:countryUK                               3.11
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                0.60
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                    1.37
## muUFad_age_groupYounger:genderMale:costNOcost                              0.32
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                 0.67
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                    -0.19
## muUFad_age_groupYounger:countryUK:costNOcost                              -0.10
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer      0.99
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                1.36
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                    0.69
## muUFad_genderMale:countryUK:competitionNOcompetition                       1.21
## muUFad_genderMale:countryUK:relationUnknownMpeer                          -0.72
## muUFad_genderMale:countryUK:costNOcost                                     1.07
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer            0.98
## muUFad_genderMale:competitionNOcompetition:costNOcost                      1.23
## muUFad_genderMale:relationUnknownMpeer:costNOcost                          1.97
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer             1.15
## muUFad_countryUK:competitionNOcompetition:costNOcost                       0.20
## muUFad_countryUK:relationUnknownMpeer:costNOcost                           2.71
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost            1.15
## muUFdis_age_groupYounger                                                   0.17
## muUFdis_genderMale                                                         1.76
## muUFdis_countryUK                                                          1.47
## muUFdis_competitionNOcompetition                                           1.77
## muUFdis_relationUnknownMpeer                                               0.86
## muUFdis_costNOcost                                                        -0.84
## muUFdis_age_groupYounger:genderMale                                        3.89
## muUFdis_age_groupYounger:countryUK                                         2.38
## muUFdis_age_groupYounger:competitionNOcompetition                          2.90
## muUFdis_age_groupYounger:relationUnknownMpeer                              3.27
## muUFdis_age_groupYounger:costNOcost                                        5.91
## muUFdis_genderMale:countryUK                                               3.13
## muUFdis_genderMale:competitionNOcompetition                                0.17
## muUFdis_genderMale:relationUnknownMpeer                                    0.15
## muUFdis_genderMale:costNOcost                                              3.00
## muUFdis_countryUK:competitionNOcompetition                                 0.94
## muUFdis_countryUK:relationUnknownMpeer                                     2.65
## muUFdis_countryUK:costNOcost                                               3.47
## muUFdis_competitionNOcompetition:relationUnknownMpeer                      1.59
## muUFdis_competitionNOcompetition:costNOcost                                2.11
## muUFdis_relationUnknownMpeer:costNOcost                                    2.86
## muUFdis_age_groupYounger:genderMale:countryUK                              1.61
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition               1.70
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                   3.01
## muUFdis_age_groupYounger:genderMale:costNOcost                            -0.42
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                1.67
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                    0.32
## muUFdis_age_groupYounger:countryUK:costNOcost                              1.12
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer     2.20
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost               1.35
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                   0.05
## muUFdis_genderMale:countryUK:competitionNOcompetition                      1.85
## muUFdis_genderMale:countryUK:relationUnknownMpeer                          1.52
## muUFdis_genderMale:countryUK:costNOcost                                    0.65
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer           2.10
## muUFdis_genderMale:competitionNOcompetition:costNOcost                     2.69
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                         3.12
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer            2.31
## muUFdis_countryUK:competitionNOcompetition:costNOcost                      1.66
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                          1.63
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost           1.60
##                                                                        Rhat
## muUFad_Intercept                                                       1.00
## muUFdis_Intercept                                                      1.00
## muUFad_age_groupYounger                                                1.00
## muUFad_genderMale                                                      1.00
## muUFad_countryUK                                                       1.00
## muUFad_competitionNOcompetition                                        1.00
## muUFad_relationUnknownMpeer                                            1.00
## muUFad_costNOcost                                                      1.00
## muUFad_age_groupYounger:genderMale                                     1.00
## muUFad_age_groupYounger:countryUK                                      1.00
## muUFad_age_groupYounger:competitionNOcompetition                       1.00
## muUFad_age_groupYounger:relationUnknownMpeer                           1.00
## muUFad_age_groupYounger:costNOcost                                     1.00
## muUFad_genderMale:countryUK                                            1.00
## muUFad_genderMale:competitionNOcompetition                             1.00
## muUFad_genderMale:relationUnknownMpeer                                 1.00
## muUFad_genderMale:costNOcost                                           1.00
## muUFad_countryUK:competitionNOcompetition                              1.00
## muUFad_countryUK:relationUnknownMpeer                                  1.00
## muUFad_countryUK:costNOcost                                            1.00
## muUFad_competitionNOcompetition:relationUnknownMpeer                   1.00
## muUFad_competitionNOcompetition:costNOcost                             1.00
## muUFad_relationUnknownMpeer:costNOcost                                 1.00
## muUFad_age_groupYounger:genderMale:countryUK                           1.00
## muUFad_age_groupYounger:genderMale:competitionNOcompetition            1.00
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                1.00
## muUFad_age_groupYounger:genderMale:costNOcost                          1.00
## muUFad_age_groupYounger:countryUK:competitionNOcompetition             1.00
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                 1.00
## muUFad_age_groupYounger:countryUK:costNOcost                           1.00
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer  1.00
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost            1.00
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                1.00
## muUFad_genderMale:countryUK:competitionNOcompetition                   1.00
## muUFad_genderMale:countryUK:relationUnknownMpeer                       1.00
## muUFad_genderMale:countryUK:costNOcost                                 1.00
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer        1.00
## muUFad_genderMale:competitionNOcompetition:costNOcost                  1.00
## muUFad_genderMale:relationUnknownMpeer:costNOcost                      1.00
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer         1.00
## muUFad_countryUK:competitionNOcompetition:costNOcost                   1.00
## muUFad_countryUK:relationUnknownMpeer:costNOcost                       1.00
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost        1.00
## muUFdis_age_groupYounger                                               1.00
## muUFdis_genderMale                                                     1.00
## muUFdis_countryUK                                                      1.00
## muUFdis_competitionNOcompetition                                       1.00
## muUFdis_relationUnknownMpeer                                           1.00
## muUFdis_costNOcost                                                     1.00
## muUFdis_age_groupYounger:genderMale                                    1.00
## muUFdis_age_groupYounger:countryUK                                     1.00
## muUFdis_age_groupYounger:competitionNOcompetition                      1.00
## muUFdis_age_groupYounger:relationUnknownMpeer                          1.00
## muUFdis_age_groupYounger:costNOcost                                    1.00
## muUFdis_genderMale:countryUK                                           1.00
## muUFdis_genderMale:competitionNOcompetition                            1.00
## muUFdis_genderMale:relationUnknownMpeer                                1.00
## muUFdis_genderMale:costNOcost                                          1.00
## muUFdis_countryUK:competitionNOcompetition                             1.00
## muUFdis_countryUK:relationUnknownMpeer                                 1.00
## muUFdis_countryUK:costNOcost                                           1.00
## muUFdis_competitionNOcompetition:relationUnknownMpeer                  1.00
## muUFdis_competitionNOcompetition:costNOcost                            1.00
## muUFdis_relationUnknownMpeer:costNOcost                                1.00
## muUFdis_age_groupYounger:genderMale:countryUK                          1.00
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition           1.00
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer               1.00
## muUFdis_age_groupYounger:genderMale:costNOcost                         1.00
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition            1.00
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                1.00
## muUFdis_age_groupYounger:countryUK:costNOcost                          1.00
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer 1.00
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost           1.00
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost               1.00
## muUFdis_genderMale:countryUK:competitionNOcompetition                  1.00
## muUFdis_genderMale:countryUK:relationUnknownMpeer                      1.00
## muUFdis_genderMale:countryUK:costNOcost                                1.00
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer       1.00
## muUFdis_genderMale:competitionNOcompetition:costNOcost                 1.00
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                     1.00
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer        1.00
## muUFdis_countryUK:competitionNOcompetition:costNOcost                  1.00
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                      1.00
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost       1.00
##                                                                        Bulk_ESS
## muUFad_Intercept                                                            962
## muUFdis_Intercept                                                          1426
## muUFad_age_groupYounger                                                     930
## muUFad_genderMale                                                          1171
## muUFad_countryUK                                                           1171
## muUFad_competitionNOcompetition                                            1108
## muUFad_relationUnknownMpeer                                                1314
## muUFad_costNOcost                                                          1198
## muUFad_age_groupYounger:genderMale                                         1155
## muUFad_age_groupYounger:countryUK                                          1095
## muUFad_age_groupYounger:competitionNOcompetition                           1614
## muUFad_age_groupYounger:relationUnknownMpeer                               1943
## muUFad_age_groupYounger:costNOcost                                         1919
## muUFad_genderMale:countryUK                                                1273
## muUFad_genderMale:competitionNOcompetition                                 1863
## muUFad_genderMale:relationUnknownMpeer                                     2051
## muUFad_genderMale:costNOcost                                               1633
## muUFad_countryUK:competitionNOcompetition                                  1860
## muUFad_countryUK:relationUnknownMpeer                                      1746
## muUFad_countryUK:costNOcost                                                1911
## muUFad_competitionNOcompetition:relationUnknownMpeer                       1612
## muUFad_competitionNOcompetition:costNOcost                                 1598
## muUFad_relationUnknownMpeer:costNOcost                                     2196
## muUFad_age_groupYounger:genderMale:countryUK                               1101
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                2917
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                    3112
## muUFad_age_groupYounger:genderMale:costNOcost                              3212
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                 3195
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                     2794
## muUFad_age_groupYounger:countryUK:costNOcost                               2907
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer      2984
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                3359
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                    3754
## muUFad_genderMale:countryUK:competitionNOcompetition                       3164
## muUFad_genderMale:countryUK:relationUnknownMpeer                           2488
## muUFad_genderMale:countryUK:costNOcost                                     2703
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer            2749
## muUFad_genderMale:competitionNOcompetition:costNOcost                      2318
## muUFad_genderMale:relationUnknownMpeer:costNOcost                          3424
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer             2694
## muUFad_countryUK:competitionNOcompetition:costNOcost                       3201
## muUFad_countryUK:relationUnknownMpeer:costNOcost                           3384
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost            3636
## muUFdis_age_groupYounger                                                   1111
## muUFdis_genderMale                                                         1468
## muUFdis_countryUK                                                          1423
## muUFdis_competitionNOcompetition                                           1740
## muUFdis_relationUnknownMpeer                                               1622
## muUFdis_costNOcost                                                         1425
## muUFdis_age_groupYounger:genderMale                                        1377
## muUFdis_age_groupYounger:countryUK                                         1587
## muUFdis_age_groupYounger:competitionNOcompetition                          1696
## muUFdis_age_groupYounger:relationUnknownMpeer                              1874
## muUFdis_age_groupYounger:costNOcost                                        1683
## muUFdis_genderMale:countryUK                                               1466
## muUFdis_genderMale:competitionNOcompetition                                2317
## muUFdis_genderMale:relationUnknownMpeer                                    2076
## muUFdis_genderMale:costNOcost                                              1652
## muUFdis_countryUK:competitionNOcompetition                                 2474
## muUFdis_countryUK:relationUnknownMpeer                                     2083
## muUFdis_countryUK:costNOcost                                               2116
## muUFdis_competitionNOcompetition:relationUnknownMpeer                      1920
## muUFdis_competitionNOcompetition:costNOcost                                1885
## muUFdis_relationUnknownMpeer:costNOcost                                    2185
## muUFdis_age_groupYounger:genderMale:countryUK                              1647
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition               2354
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                   2485
## muUFdis_age_groupYounger:genderMale:costNOcost                             2292
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                2805
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                    2955
## muUFdis_age_groupYounger:countryUK:costNOcost                              2722
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer     2394
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost               2509
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                   3025
## muUFdis_genderMale:countryUK:competitionNOcompetition                      3753
## muUFdis_genderMale:countryUK:relationUnknownMpeer                          3130
## muUFdis_genderMale:countryUK:costNOcost                                    2675
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer           3100
## muUFdis_genderMale:competitionNOcompetition:costNOcost                     2526
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                         2989
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer            3294
## muUFdis_countryUK:competitionNOcompetition:costNOcost                      2782
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                          3781
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost           3835
##                                                                        Tail_ESS
## muUFad_Intercept                                                           1717
## muUFdis_Intercept                                                          2056
## muUFad_age_groupYounger                                                    1782
## muUFad_genderMale                                                          2127
## muUFad_countryUK                                                           1957
## muUFad_competitionNOcompetition                                            2261
## muUFad_relationUnknownMpeer                                                2133
## muUFad_costNOcost                                                          1798
## muUFad_age_groupYounger:genderMale                                         1979
## muUFad_age_groupYounger:countryUK                                          1652
## muUFad_age_groupYounger:competitionNOcompetition                           2694
## muUFad_age_groupYounger:relationUnknownMpeer                               2933
## muUFad_age_groupYounger:costNOcost                                         2304
## muUFad_genderMale:countryUK                                                1895
## muUFad_genderMale:competitionNOcompetition                                 2310
## muUFad_genderMale:relationUnknownMpeer                                     2476
## muUFad_genderMale:costNOcost                                               2443
## muUFad_countryUK:competitionNOcompetition                                  2675
## muUFad_countryUK:relationUnknownMpeer                                      2311
## muUFad_countryUK:costNOcost                                                2529
## muUFad_competitionNOcompetition:relationUnknownMpeer                       2533
## muUFad_competitionNOcompetition:costNOcost                                 2101
## muUFad_relationUnknownMpeer:costNOcost                                     2530
## muUFad_age_groupYounger:genderMale:countryUK                               1663
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                3033
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                    3122
## muUFad_age_groupYounger:genderMale:costNOcost                              3217
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                 2984
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                     3021
## muUFad_age_groupYounger:countryUK:costNOcost                               2745
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer      2941
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                3059
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                    3079
## muUFad_genderMale:countryUK:competitionNOcompetition                       3085
## muUFad_genderMale:countryUK:relationUnknownMpeer                           2869
## muUFad_genderMale:countryUK:costNOcost                                     2949
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer            2971
## muUFad_genderMale:competitionNOcompetition:costNOcost                      2500
## muUFad_genderMale:relationUnknownMpeer:costNOcost                          2685
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer             3275
## muUFad_countryUK:competitionNOcompetition:costNOcost                       2843
## muUFad_countryUK:relationUnknownMpeer:costNOcost                           2958
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost            3107
## muUFdis_age_groupYounger                                                   2103
## muUFdis_genderMale                                                         2176
## muUFdis_countryUK                                                          2126
## muUFdis_competitionNOcompetition                                           2530
## muUFdis_relationUnknownMpeer                                               2328
## muUFdis_costNOcost                                                         1851
## muUFdis_age_groupYounger:genderMale                                        2288
## muUFdis_age_groupYounger:countryUK                                         2657
## muUFdis_age_groupYounger:competitionNOcompetition                          2209
## muUFdis_age_groupYounger:relationUnknownMpeer                              2668
## muUFdis_age_groupYounger:costNOcost                                        2294
## muUFdis_genderMale:countryUK                                               1981
## muUFdis_genderMale:competitionNOcompetition                                2974
## muUFdis_genderMale:relationUnknownMpeer                                    2660
## muUFdis_genderMale:costNOcost                                              1954
## muUFdis_countryUK:competitionNOcompetition                                 2641
## muUFdis_countryUK:relationUnknownMpeer                                     2605
## muUFdis_countryUK:costNOcost                                               2943
## muUFdis_competitionNOcompetition:relationUnknownMpeer                      2573
## muUFdis_competitionNOcompetition:costNOcost                                2439
## muUFdis_relationUnknownMpeer:costNOcost                                    2186
## muUFdis_age_groupYounger:genderMale:countryUK                              2154
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition               2852
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                   2755
## muUFdis_age_groupYounger:genderMale:costNOcost                             2749
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                2850
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                    2887
## muUFdis_age_groupYounger:countryUK:costNOcost                              2849
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer     3010
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost               2852
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                   2641
## muUFdis_genderMale:countryUK:competitionNOcompetition                      3076
## muUFdis_genderMale:countryUK:relationUnknownMpeer                          3139
## muUFdis_genderMale:countryUK:costNOcost                                    2530
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer           3208
## muUFdis_genderMale:competitionNOcompetition:costNOcost                     2925
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                         3315
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer            2894
## muUFdis_countryUK:competitionNOcompetition:costNOcost                      2584
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                          2672
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost           3191
## 
## Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
## and Tail_ESS are effective sample size measures, and Rhat is the potential
## scale reduction factor on split chains (at convergence, Rhat = 1).
```

4: in model **`brm2.4`**, we add all 6 effects and their 4-way interactions.


```r
summary(brm2.4)
```

```
##  Family: categorical 
##   Links: muUFad = logit; muUFdis = logit 
## Formula: allocation ~ (age_group + gender + country + competition + relation + cost)^4 + (1 | id) 
##    Data: R2long (Number of observations: 1632) 
## Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
##          total post-warmup samples = 4000
## 
## Group-Level Effects: 
## ~id (Number of levels: 204) 
##                       Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS
## sd(muUFad_Intercept)      1.40      0.14     1.14     1.71 1.00     1483
## sd(muUFdis_Intercept)     1.13      0.19     0.78     1.53 1.00     1444
##                       Tail_ESS
## sd(muUFad_Intercept)      2586
## sd(muUFdis_Intercept)     2469
## 
## Population-Level Effects: 
##                                                                                   Estimate
## muUFad_Intercept                                                                      0.10
## muUFdis_Intercept                                                                    -2.29
## muUFad_age_groupYounger                                                              -0.74
## muUFad_genderMale                                                                    -0.07
## muUFad_countryUK                                                                      0.72
## muUFad_competitionNOcompetition                                                      -1.06
## muUFad_relationUnknownMpeer                                                          -1.08
## muUFad_costNOcost                                                                    -7.26
## muUFad_age_groupYounger:genderMale                                                   -0.21
## muUFad_age_groupYounger:countryUK                                                    -0.01
## muUFad_age_groupYounger:competitionNOcompetition                                     -1.14
## muUFad_age_groupYounger:relationUnknownMpeer                                          0.15
## muUFad_age_groupYounger:costNOcost                                                    6.06
## muUFad_genderMale:countryUK                                                           0.35
## muUFad_genderMale:competitionNOcompetition                                            0.48
## muUFad_genderMale:relationUnknownMpeer                                                0.24
## muUFad_genderMale:costNOcost                                                          5.48
## muUFad_countryUK:competitionNOcompetition                                             0.43
## muUFad_countryUK:relationUnknownMpeer                                                 1.10
## muUFad_countryUK:costNOcost                                                           4.83
## muUFad_competitionNOcompetition:relationUnknownMpeer                                  0.19
## muUFad_competitionNOcompetition:costNOcost                                            5.68
## muUFad_relationUnknownMpeer:costNOcost                                               -5.81
## muUFad_age_groupYounger:genderMale:countryUK                                          1.13
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                           1.69
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                               1.31
## muUFad_age_groupYounger:genderMale:costNOcost                                        -5.86
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                            1.52
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                               -0.64
## muUFad_age_groupYounger:countryUK:costNOcost                                         -5.82
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                 1.14
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                          -3.30
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                               2.84
## muUFad_genderMale:countryUK:competitionNOcompetition                                 -0.77
## muUFad_genderMale:countryUK:relationUnknownMpeer                                     -2.80
## muUFad_genderMale:countryUK:costNOcost                                               -8.00
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                       0.53
## muUFad_genderMale:competitionNOcompetition:costNOcost                                -5.31
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                     5.35
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                       -0.30
## muUFad_countryUK:competitionNOcompetition:costNOcost                                 -6.39
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                      6.22
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                       2.71
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                -2.75
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                     0.27
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                               6.69
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer     -1.92
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                3.03
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                   -2.43
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer      -0.47
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                 1.51
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                    -2.91
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost      0.21
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer             1.21
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                       7.18
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                          -1.92
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost           -2.37
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            -1.69
## muUFdis_age_groupYounger                                                             -1.66
## muUFdis_genderMale                                                                    0.66
## muUFdis_countryUK                                                                     0.03
## muUFdis_competitionNOcompetition                                                      1.08
## muUFdis_relationUnknownMpeer                                                          0.29
## muUFdis_costNOcost                                                                   -1.01
## muUFdis_age_groupYounger:genderMale                                                   2.21
## muUFdis_age_groupYounger:countryUK                                                   -6.51
## muUFdis_age_groupYounger:competitionNOcompetition                                     1.35
## muUFdis_age_groupYounger:relationUnknownMpeer                                         1.32
## muUFdis_age_groupYounger:costNOcost                                                   2.08
## muUFdis_genderMale:countryUK                                                          1.00
## muUFdis_genderMale:competitionNOcompetition                                          -1.76
## muUFdis_genderMale:relationUnknownMpeer                                              -3.48
## muUFdis_genderMale:costNOcost                                                        -0.30
## muUFdis_countryUK:competitionNOcompetition                                           -1.13
## muUFdis_countryUK:relationUnknownMpeer                                                0.83
## muUFdis_countryUK:costNOcost                                                         -0.28
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                -1.51
## muUFdis_competitionNOcompetition:costNOcost                                          -1.73
## muUFdis_relationUnknownMpeer:costNOcost                                              -5.36
## muUFdis_age_groupYounger:genderMale:countryUK                                         4.81
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                         -2.52
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                              0.71
## muUFdis_age_groupYounger:genderMale:costNOcost                                       -1.86
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                           5.66
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                              -0.32
## muUFdis_age_groupYounger:countryUK:costNOcost                                         7.99
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer               -0.51
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                          0.18
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                              3.60
## muUFdis_genderMale:countryUK:competitionNOcompetition                                -1.31
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                    -1.40
## muUFdis_genderMale:countryUK:costNOcost                                              -0.69
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                      2.67
## muUFdis_genderMale:competitionNOcompetition:costNOcost                               -0.75
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                    7.73
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                      -1.03
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                -0.64
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                     1.16
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                      6.63
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition               -2.80
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                    1.48
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                             -6.83
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer     2.43
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost               4.93
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                  -3.96
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer      1.48
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost               -6.28
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                   -3.87
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost    -4.15
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer           -2.42
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                      5.48
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                          0.87
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost          -6.02
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            2.17
##                                                                                   Est.Error
## muUFad_Intercept                                                                       0.56
## muUFdis_Intercept                                                                      0.90
## muUFad_age_groupYounger                                                                0.83
## muUFad_genderMale                                                                      0.78
## muUFad_countryUK                                                                       0.84
## muUFad_competitionNOcompetition                                                        0.72
## muUFad_relationUnknownMpeer                                                            0.71
## muUFad_costNOcost                                                                      1.97
## muUFad_age_groupYounger:genderMale                                                     1.07
## muUFad_age_groupYounger:countryUK                                                      1.15
## muUFad_age_groupYounger:competitionNOcompetition                                       1.12
## muUFad_age_groupYounger:relationUnknownMpeer                                           1.02
## muUFad_age_groupYounger:costNOcost                                                     1.98
## muUFad_genderMale:countryUK                                                            1.13
## muUFad_genderMale:competitionNOcompetition                                             0.94
## muUFad_genderMale:relationUnknownMpeer                                                 0.93
## muUFad_genderMale:costNOcost                                                           1.98
## muUFad_countryUK:competitionNOcompetition                                              0.98
## muUFad_countryUK:relationUnknownMpeer                                                  0.98
## muUFad_countryUK:costNOcost                                                            1.99
## muUFad_competitionNOcompetition:relationUnknownMpeer                                   1.02
## muUFad_competitionNOcompetition:costNOcost                                             1.96
## muUFad_relationUnknownMpeer:costNOcost                                                 2.44
## muUFad_age_groupYounger:genderMale:countryUK                                           1.51
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                            1.24
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                1.21
## muUFad_age_groupYounger:genderMale:costNOcost                                          1.96
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                             1.30
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                 1.23
## muUFad_age_groupYounger:countryUK:costNOcost                                           2.01
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                  1.36
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                            1.89
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                1.85
## muUFad_genderMale:countryUK:competitionNOcompetition                                   1.24
## muUFad_genderMale:countryUK:relationUnknownMpeer                                       1.24
## muUFad_genderMale:countryUK:costNOcost                                                 2.03
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                        1.21
## muUFad_genderMale:competitionNOcompetition:costNOcost                                  1.97
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                      2.32
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                         1.25
## muUFad_countryUK:competitionNOcompetition:costNOcost                                   1.99
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                       2.26
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                        1.82
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                  1.42
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                      1.38
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                1.93
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer       1.30
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                 1.62
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                     1.57
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer        1.25
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                  1.54
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                      1.50
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost       1.34
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer              1.36
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                        1.86
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                            1.81
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost             1.58
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost              1.46
## muUFdis_age_groupYounger                                                               1.83
## muUFdis_genderMale                                                                     1.16
## muUFdis_countryUK                                                                      1.43
## muUFdis_competitionNOcompetition                                                       1.01
## muUFdis_relationUnknownMpeer                                                           1.08
## muUFdis_costNOcost                                                                     1.21
## muUFdis_age_groupYounger:genderMale                                                    2.01
## muUFdis_age_groupYounger:countryUK                                                     3.32
## muUFdis_age_groupYounger:competitionNOcompetition                                      1.90
## muUFdis_age_groupYounger:relationUnknownMpeer                                          2.01
## muUFdis_age_groupYounger:costNOcost                                                    2.17
## muUFdis_genderMale:countryUK                                                           1.67
## muUFdis_genderMale:competitionNOcompetition                                            1.42
## muUFdis_genderMale:relationUnknownMpeer                                                1.98
## muUFdis_genderMale:costNOcost                                                          1.60
## muUFdis_countryUK:competitionNOcompetition                                             1.70
## muUFdis_countryUK:relationUnknownMpeer                                                 1.68
## muUFdis_countryUK:costNOcost                                                           1.98
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                  1.38
## muUFdis_competitionNOcompetition:costNOcost                                            1.73
## muUFdis_relationUnknownMpeer:costNOcost                                                2.95
## muUFdis_age_groupYounger:genderMale:countryUK                                          3.20
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                           2.21
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                               2.50
## muUFdis_age_groupYounger:genderMale:costNOcost                                         2.45
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                            3.10
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                2.85
## muUFdis_age_groupYounger:countryUK:costNOcost                                          3.31
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                 2.23
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                           2.52
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                               3.08
## muUFdis_genderMale:countryUK:competitionNOcompetition                                  2.35
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                      2.51
## muUFdis_genderMale:countryUK:costNOcost                                                2.29
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                       2.51
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                 2.68
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                     2.95
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                        2.33
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                  2.72
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                      2.75
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                       2.95
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                 2.85
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                     2.69
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                               3.01
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer      2.79
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                2.98
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                    2.88
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer       2.54
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                 2.86
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                     2.56
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost      2.92
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer             2.20
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                       3.16
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                           2.41
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost            2.58
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost             2.15
##                                                                                   l-95% CI
## muUFad_Intercept                                                                     -1.01
## muUFdis_Intercept                                                                    -4.25
## muUFad_age_groupYounger                                                              -2.39
## muUFad_genderMale                                                                    -1.57
## muUFad_countryUK                                                                     -0.88
## muUFad_competitionNOcompetition                                                      -2.51
## muUFad_relationUnknownMpeer                                                          -2.52
## muUFad_costNOcost                                                                   -11.61
## muUFad_age_groupYounger:genderMale                                                   -2.34
## muUFad_age_groupYounger:countryUK                                                    -2.23
## muUFad_age_groupYounger:competitionNOcompetition                                     -3.47
## muUFad_age_groupYounger:relationUnknownMpeer                                         -1.81
## muUFad_age_groupYounger:costNOcost                                                    2.49
## muUFad_genderMale:countryUK                                                          -1.91
## muUFad_genderMale:competitionNOcompetition                                           -1.32
## muUFad_genderMale:relationUnknownMpeer                                               -1.60
## muUFad_genderMale:costNOcost                                                          1.93
## muUFad_countryUK:competitionNOcompetition                                            -1.48
## muUFad_countryUK:relationUnknownMpeer                                                -0.77
## muUFad_countryUK:costNOcost                                                           1.35
## muUFad_competitionNOcompetition:relationUnknownMpeer                                 -1.87
## muUFad_competitionNOcompetition:costNOcost                                            2.27
## muUFad_relationUnknownMpeer:costNOcost                                              -10.86
## muUFad_age_groupYounger:genderMale:countryUK                                         -1.86
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                          -0.62
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                              -1.07
## muUFad_age_groupYounger:genderMale:costNOcost                                       -10.02
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                           -0.98
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                               -3.08
## muUFad_age_groupYounger:countryUK:costNOcost                                        -10.12
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                -1.57
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                          -7.21
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                              -0.82
## muUFad_genderMale:countryUK:competitionNOcompetition                                 -3.19
## muUFad_genderMale:countryUK:relationUnknownMpeer                                     -5.24
## muUFad_genderMale:countryUK:costNOcost                                              -12.18
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                      -1.82
## muUFad_genderMale:competitionNOcompetition:costNOcost                                -9.38
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                     1.18
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                       -2.79
## muUFad_countryUK:competitionNOcompetition:costNOcost                                -10.53
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                      2.06
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                      -0.70
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                -5.60
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                    -2.41
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                               3.20
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer     -4.51
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost               -0.04
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                   -5.60
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer      -2.88
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                -1.36
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                    -5.90
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost     -2.42
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer            -1.43
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                       3.68
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                          -5.65
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost           -5.68
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            -4.58
## muUFdis_age_groupYounger                                                             -5.80
## muUFdis_genderMale                                                                   -1.50
## muUFdis_countryUK                                                                    -2.83
## muUFdis_competitionNOcompetition                                                     -0.76
## muUFdis_relationUnknownMpeer                                                         -1.77
## muUFdis_costNOcost                                                                   -3.52
## muUFdis_age_groupYounger:genderMale                                                  -1.29
## muUFdis_age_groupYounger:countryUK                                                  -13.74
## muUFdis_age_groupYounger:competitionNOcompetition                                    -1.86
## muUFdis_age_groupYounger:relationUnknownMpeer                                        -2.26
## muUFdis_age_groupYounger:costNOcost                                                  -1.84
## muUFdis_genderMale:countryUK                                                         -2.18
## muUFdis_genderMale:competitionNOcompetition                                          -4.58
## muUFdis_genderMale:relationUnknownMpeer                                              -7.73
## muUFdis_genderMale:costNOcost                                                        -3.41
## muUFdis_countryUK:competitionNOcompetition                                           -4.44
## muUFdis_countryUK:relationUnknownMpeer                                               -2.40
## muUFdis_countryUK:costNOcost                                                         -4.28
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                -4.38
## muUFdis_competitionNOcompetition:costNOcost                                          -5.34
## muUFdis_relationUnknownMpeer:costNOcost                                             -12.18
## muUFdis_age_groupYounger:genderMale:countryUK                                        -0.74
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                         -7.34
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                             -4.36
## muUFdis_age_groupYounger:genderMale:costNOcost                                       -6.93
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                           0.33
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                              -6.06
## muUFdis_age_groupYounger:countryUK:costNOcost                                         2.13
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer               -5.24
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                         -5.02
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                             -2.17
## muUFdis_genderMale:countryUK:competitionNOcompetition                                -6.32
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                    -6.40
## muUFdis_genderMale:countryUK:costNOcost                                              -5.07
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                     -2.38
## muUFdis_genderMale:competitionNOcompetition:costNOcost                               -6.36
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                    2.29
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                      -5.96
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                -6.14
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                    -4.10
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                      1.56
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition               -8.87
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                   -3.70
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                            -13.41
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer    -2.77
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost              -0.39
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                  -9.78
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer     -3.39
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost              -12.41
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                   -8.86
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost   -10.02
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer           -6.91
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                     -0.00
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                         -3.82
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost         -11.31
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost           -1.99
##                                                                                   u-95% CI
## muUFad_Intercept                                                                      1.21
## muUFdis_Intercept                                                                    -0.72
## muUFad_age_groupYounger                                                               0.85
## muUFad_genderMale                                                                     1.46
## muUFad_countryUK                                                                      2.44
## muUFad_competitionNOcompetition                                                       0.35
## muUFad_relationUnknownMpeer                                                           0.32
## muUFad_costNOcost                                                                    -3.94
## muUFad_age_groupYounger:genderMale                                                    1.89
## muUFad_age_groupYounger:countryUK                                                     2.25
## muUFad_age_groupYounger:competitionNOcompetition                                      0.92
## muUFad_age_groupYounger:relationUnknownMpeer                                          2.16
## muUFad_age_groupYounger:costNOcost                                                   10.37
## muUFad_genderMale:countryUK                                                           2.54
## muUFad_genderMale:competitionNOcompetition                                            2.33
## muUFad_genderMale:relationUnknownMpeer                                                2.02
## muUFad_genderMale:costNOcost                                                          9.75
## muUFad_countryUK:competitionNOcompetition                                             2.35
## muUFad_countryUK:relationUnknownMpeer                                                 3.08
## muUFad_countryUK:costNOcost                                                           9.12
## muUFad_competitionNOcompetition:relationUnknownMpeer                                  2.18
## muUFad_competitionNOcompetition:costNOcost                                            9.84
## muUFad_relationUnknownMpeer:costNOcost                                               -1.32
## muUFad_age_groupYounger:genderMale:countryUK                                          4.07
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                           4.25
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                               3.69
## muUFad_age_groupYounger:genderMale:costNOcost                                        -2.29
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                            4.18
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                1.74
## muUFad_age_groupYounger:countryUK:costNOcost                                         -2.19
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                 3.85
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                           0.23
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                               6.54
## muUFad_genderMale:countryUK:competitionNOcompetition                                  1.61
## muUFad_genderMale:countryUK:relationUnknownMpeer                                     -0.35
## muUFad_genderMale:countryUK:costNOcost                                               -4.25
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                       2.93
## muUFad_genderMale:competitionNOcompetition:costNOcost                                -1.66
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                    10.18
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                        2.08
## muUFad_countryUK:competitionNOcompetition:costNOcost                                 -2.75
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                     10.95
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                       6.35
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                -0.03
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                     3.04
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                              10.71
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer      0.66
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                6.29
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                    0.53
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer       2.02
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                 4.64
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                    -0.10
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost      2.83
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer             3.85
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                      11.08
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                           1.42
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost            0.57
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost             1.12
## muUFdis_age_groupYounger                                                              1.40
## muUFdis_genderMale                                                                    3.00
## muUFdis_countryUK                                                                     2.80
## muUFdis_competitionNOcompetition                                                      3.24
## muUFdis_relationUnknownMpeer                                                          2.52
## muUFdis_costNOcost                                                                    1.41
## muUFdis_age_groupYounger:genderMale                                                   6.61
## muUFdis_age_groupYounger:countryUK                                                   -0.73
## muUFdis_age_groupYounger:competitionNOcompetition                                     5.69
## muUFdis_age_groupYounger:relationUnknownMpeer                                         5.77
## muUFdis_age_groupYounger:costNOcost                                                   6.83
## muUFdis_genderMale:countryUK                                                          4.37
## muUFdis_genderMale:competitionNOcompetition                                           0.93
## muUFdis_genderMale:relationUnknownMpeer                                               0.12
## muUFdis_genderMale:costNOcost                                                         2.87
## muUFdis_countryUK:competitionNOcompetition                                            2.20
## muUFdis_countryUK:relationUnknownMpeer                                                4.15
## muUFdis_countryUK:costNOcost                                                          3.55
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                 1.15
## muUFdis_competitionNOcompetition:costNOcost                                           1.54
## muUFdis_relationUnknownMpeer:costNOcost                                              -0.37
## muUFdis_age_groupYounger:genderMale:countryUK                                        11.71
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                          1.43
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                              5.42
## muUFdis_age_groupYounger:genderMale:costNOcost                                        2.83
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                          12.53
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                               5.14
## muUFdis_age_groupYounger:countryUK:costNOcost                                        15.23
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                3.48
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                          4.88
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                             10.23
## muUFdis_genderMale:countryUK:competitionNOcompetition                                 3.13
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                     3.51
## muUFdis_genderMale:countryUK:costNOcost                                               3.79
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                      7.56
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                4.09
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                   14.13
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                       3.41
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                 4.70
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                     6.70
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                     13.00
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                2.39
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                    7.12
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                             -1.49
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer     8.07
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost              11.28
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                   1.43
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer      6.63
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost               -1.03
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                    1.09
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost     1.30
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer            1.72
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                     12.53
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                          5.60
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost          -1.09
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            6.33
##                                                                                   Rhat
## muUFad_Intercept                                                                  1.00
## muUFdis_Intercept                                                                 1.00
## muUFad_age_groupYounger                                                           1.00
## muUFad_genderMale                                                                 1.00
## muUFad_countryUK                                                                  1.00
## muUFad_competitionNOcompetition                                                   1.00
## muUFad_relationUnknownMpeer                                                       1.00
## muUFad_costNOcost                                                                 1.00
## muUFad_age_groupYounger:genderMale                                                1.00
## muUFad_age_groupYounger:countryUK                                                 1.00
## muUFad_age_groupYounger:competitionNOcompetition                                  1.00
## muUFad_age_groupYounger:relationUnknownMpeer                                      1.00
## muUFad_age_groupYounger:costNOcost                                                1.00
## muUFad_genderMale:countryUK                                                       1.00
## muUFad_genderMale:competitionNOcompetition                                        1.00
## muUFad_genderMale:relationUnknownMpeer                                            1.00
## muUFad_genderMale:costNOcost                                                      1.00
## muUFad_countryUK:competitionNOcompetition                                         1.00
## muUFad_countryUK:relationUnknownMpeer                                             1.00
## muUFad_countryUK:costNOcost                                                       1.00
## muUFad_competitionNOcompetition:relationUnknownMpeer                              1.00
## muUFad_competitionNOcompetition:costNOcost                                        1.00
## muUFad_relationUnknownMpeer:costNOcost                                            1.00
## muUFad_age_groupYounger:genderMale:countryUK                                      1.00
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                       1.00
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                           1.00
## muUFad_age_groupYounger:genderMale:costNOcost                                     1.00
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                        1.00
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                            1.00
## muUFad_age_groupYounger:countryUK:costNOcost                                      1.00
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer             1.00
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                       1.00
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                           1.00
## muUFad_genderMale:countryUK:competitionNOcompetition                              1.00
## muUFad_genderMale:countryUK:relationUnknownMpeer                                  1.00
## muUFad_genderMale:countryUK:costNOcost                                            1.00
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                   1.00
## muUFad_genderMale:competitionNOcompetition:costNOcost                             1.00
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                 1.00
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                    1.00
## muUFad_countryUK:competitionNOcompetition:costNOcost                              1.00
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                  1.00
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                   1.00
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition             1.00
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                 1.00
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                           1.00
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer  1.00
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost            1.00
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                1.00
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer   1.00
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost             1.00
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                 1.00
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost  1.00
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer         1.00
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                   1.00
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                       1.00
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost        1.00
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost         1.00
## muUFdis_age_groupYounger                                                          1.00
## muUFdis_genderMale                                                                1.00
## muUFdis_countryUK                                                                 1.00
## muUFdis_competitionNOcompetition                                                  1.00
## muUFdis_relationUnknownMpeer                                                      1.00
## muUFdis_costNOcost                                                                1.00
## muUFdis_age_groupYounger:genderMale                                               1.00
## muUFdis_age_groupYounger:countryUK                                                1.00
## muUFdis_age_groupYounger:competitionNOcompetition                                 1.00
## muUFdis_age_groupYounger:relationUnknownMpeer                                     1.00
## muUFdis_age_groupYounger:costNOcost                                               1.00
## muUFdis_genderMale:countryUK                                                      1.00
## muUFdis_genderMale:competitionNOcompetition                                       1.00
## muUFdis_genderMale:relationUnknownMpeer                                           1.00
## muUFdis_genderMale:costNOcost                                                     1.00
## muUFdis_countryUK:competitionNOcompetition                                        1.00
## muUFdis_countryUK:relationUnknownMpeer                                            1.00
## muUFdis_countryUK:costNOcost                                                      1.00
## muUFdis_competitionNOcompetition:relationUnknownMpeer                             1.00
## muUFdis_competitionNOcompetition:costNOcost                                       1.00
## muUFdis_relationUnknownMpeer:costNOcost                                           1.00
## muUFdis_age_groupYounger:genderMale:countryUK                                     1.00
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                      1.00
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                          1.00
## muUFdis_age_groupYounger:genderMale:costNOcost                                    1.00
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                       1.00
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                           1.00
## muUFdis_age_groupYounger:countryUK:costNOcost                                     1.00
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer            1.00
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                      1.00
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                          1.00
## muUFdis_genderMale:countryUK:competitionNOcompetition                             1.00
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                 1.00
## muUFdis_genderMale:countryUK:costNOcost                                           1.00
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                  1.00
## muUFdis_genderMale:competitionNOcompetition:costNOcost                            1.00
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                1.00
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                   1.00
## muUFdis_countryUK:competitionNOcompetition:costNOcost                             1.00
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                 1.00
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                  1.00
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition            1.00
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                1.00
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                          1.00
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer 1.00
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost           1.00
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost               1.00
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer  1.00
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost            1.00
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                1.00
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost 1.00
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer        1.00
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                  1.00
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                      1.00
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost       1.00
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost        1.00
##                                                                                   Bulk_ESS
## muUFad_Intercept                                                                      1858
## muUFdis_Intercept                                                                     1370
## muUFad_age_groupYounger                                                               2005
## muUFad_genderMale                                                                     1768
## muUFad_countryUK                                                                      1907
## muUFad_competitionNOcompetition                                                       1795
## muUFad_relationUnknownMpeer                                                           1668
## muUFad_costNOcost                                                                      712
## muUFad_age_groupYounger:genderMale                                                    1953
## muUFad_age_groupYounger:countryUK                                                     2045
## muUFad_age_groupYounger:competitionNOcompetition                                      1955
## muUFad_age_groupYounger:relationUnknownMpeer                                          2028
## muUFad_age_groupYounger:costNOcost                                                     743
## muUFad_genderMale:countryUK                                                           1859
## muUFad_genderMale:competitionNOcompetition                                            1975
## muUFad_genderMale:relationUnknownMpeer                                                1887
## muUFad_genderMale:costNOcost                                                           725
## muUFad_countryUK:competitionNOcompetition                                             2130
## muUFad_countryUK:relationUnknownMpeer                                                 1836
## muUFad_countryUK:costNOcost                                                            739
## muUFad_competitionNOcompetition:relationUnknownMpeer                                  1870
## muUFad_competitionNOcompetition:costNOcost                                             857
## muUFad_relationUnknownMpeer:costNOcost                                                1722
## muUFad_age_groupYounger:genderMale:countryUK                                          1973
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                           2022
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                               2138
## muUFad_age_groupYounger:genderMale:costNOcost                                          828
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                            2128
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                2117
## muUFad_age_groupYounger:countryUK:costNOcost                                           882
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                 2493
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                           1170
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                               2789
## muUFad_genderMale:countryUK:competitionNOcompetition                                  2195
## muUFad_genderMale:countryUK:relationUnknownMpeer                                      2119
## muUFad_genderMale:countryUK:costNOcost                                                 964
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                       2017
## muUFad_genderMale:competitionNOcompetition:costNOcost                                  941
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                     1765
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                        2143
## muUFad_countryUK:competitionNOcompetition:costNOcost                                   965
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                      2036
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                       2835
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                 1753
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                     2424
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                               1249
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer      2608
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                1430
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                    3106
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer       3177
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                 2098
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                     3786
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost      5034
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer             2439
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                       1297
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                           2392
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost            3331
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost             4301
## muUFdis_age_groupYounger                                                               981
## muUFdis_genderMale                                                                    1449
## muUFdis_countryUK                                                                     1567
## muUFdis_competitionNOcompetition                                                      1430
## muUFdis_relationUnknownMpeer                                                          1539
## muUFdis_costNOcost                                                                    1321
## muUFdis_age_groupYounger:genderMale                                                   1024
## muUFdis_age_groupYounger:countryUK                                                    1415
## muUFdis_age_groupYounger:competitionNOcompetition                                      944
## muUFdis_age_groupYounger:relationUnknownMpeer                                         1095
## muUFdis_age_groupYounger:costNOcost                                                    960
## muUFdis_genderMale:countryUK                                                          1597
## muUFdis_genderMale:competitionNOcompetition                                           1639
## muUFdis_genderMale:relationUnknownMpeer                                               2070
## muUFdis_genderMale:costNOcost                                                         1487
## muUFdis_countryUK:competitionNOcompetition                                            1680
## muUFdis_countryUK:relationUnknownMpeer                                                1765
## muUFdis_countryUK:costNOcost                                                          1644
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                 1946
## muUFdis_competitionNOcompetition:costNOcost                                           1744
## muUFdis_relationUnknownMpeer:costNOcost                                               1761
## muUFdis_age_groupYounger:genderMale:countryUK                                         1535
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                          1169
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                              1475
## muUFdis_age_groupYounger:genderMale:costNOcost                                        1048
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                           1511
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                               2498
## muUFdis_age_groupYounger:countryUK:costNOcost                                         1513
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                1292
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                          1136
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                              1598
## muUFdis_genderMale:countryUK:competitionNOcompetition                                 1759
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                     2243
## muUFdis_genderMale:countryUK:costNOcost                                               1603
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                      2205
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                1737
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                    1948
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                       2346
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                 1858
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                     2807
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                      1810
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                2028
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                    2448
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                              1654
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer     1729
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost               1413
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                   2002
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer      2974
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                2103
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                    2801
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost     1739
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer            2855
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                      1295
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                          2728
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost           2080
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            2701
##                                                                                   Tail_ESS
## muUFad_Intercept                                                                      2584
## muUFdis_Intercept                                                                     1777
## muUFad_age_groupYounger                                                               2879
## muUFad_genderMale                                                                     2203
## muUFad_countryUK                                                                      2256
## muUFad_competitionNOcompetition                                                       2616
## muUFad_relationUnknownMpeer                                                           2365
## muUFad_costNOcost                                                                     1360
## muUFad_age_groupYounger:genderMale                                                    2636
## muUFad_age_groupYounger:countryUK                                                     2679
## muUFad_age_groupYounger:competitionNOcompetition                                      2725
## muUFad_age_groupYounger:relationUnknownMpeer                                          2667
## muUFad_age_groupYounger:costNOcost                                                    1200
## muUFad_genderMale:countryUK                                                           2231
## muUFad_genderMale:competitionNOcompetition                                            2472
## muUFad_genderMale:relationUnknownMpeer                                                2658
## muUFad_genderMale:costNOcost                                                          1338
## muUFad_countryUK:competitionNOcompetition                                             2922
## muUFad_countryUK:relationUnknownMpeer                                                 2450
## muUFad_countryUK:costNOcost                                                           1436
## muUFad_competitionNOcompetition:relationUnknownMpeer                                  2594
## muUFad_competitionNOcompetition:costNOcost                                            1627
## muUFad_relationUnknownMpeer:costNOcost                                                1508
## muUFad_age_groupYounger:genderMale:countryUK                                          2339
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                           2794
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                               2820
## muUFad_age_groupYounger:genderMale:costNOcost                                         1417
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                            2942
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                2902
## muUFad_age_groupYounger:countryUK:costNOcost                                          1242
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                 3055
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                           1903
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                               2620
## muUFad_genderMale:countryUK:competitionNOcompetition                                  2484
## muUFad_genderMale:countryUK:relationUnknownMpeer                                      2651
## muUFad_genderMale:countryUK:costNOcost                                                1389
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                       2699
## muUFad_genderMale:competitionNOcompetition:costNOcost                                 1789
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                     1673
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                        2962
## muUFad_countryUK:competitionNOcompetition:costNOcost                                  1911
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                      1763
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                       3092
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                 2697
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                     2828
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                               1538
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer      3066
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                2055
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                    2889
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer       3064
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                 2802
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                     3036
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost      3071
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer             2547
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                       1635
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                           2072
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost            2844
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost             3287
## muUFdis_age_groupYounger                                                              1217
## muUFdis_genderMale                                                                    1939
## muUFdis_countryUK                                                                     2241
## muUFdis_competitionNOcompetition                                                      1812
## muUFdis_relationUnknownMpeer                                                          2118
## muUFdis_costNOcost                                                                    2159
## muUFdis_age_groupYounger:genderMale                                                   1418
## muUFdis_age_groupYounger:countryUK                                                    1812
## muUFdis_age_groupYounger:competitionNOcompetition                                     1461
## muUFdis_age_groupYounger:relationUnknownMpeer                                         1404
## muUFdis_age_groupYounger:costNOcost                                                   1231
## muUFdis_genderMale:countryUK                                                          2248
## muUFdis_genderMale:competitionNOcompetition                                           2116
## muUFdis_genderMale:relationUnknownMpeer                                               2336
## muUFdis_genderMale:costNOcost                                                         1800
## muUFdis_countryUK:competitionNOcompetition                                            2518
## muUFdis_countryUK:relationUnknownMpeer                                                2366
## muUFdis_countryUK:costNOcost                                                          2287
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                 2135
## muUFdis_competitionNOcompetition:costNOcost                                           2595
## muUFdis_relationUnknownMpeer:costNOcost                                               1671
## muUFdis_age_groupYounger:genderMale:countryUK                                         1809
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                          1470
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                              1709
## muUFdis_age_groupYounger:genderMale:costNOcost                                        1488
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                           2002
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                               2588
## muUFdis_age_groupYounger:countryUK:costNOcost                                         1865
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                1531
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                          1359
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                              1978
## muUFdis_genderMale:countryUK:competitionNOcompetition                                 2373
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                     2503
## muUFdis_genderMale:countryUK:costNOcost                                               2247
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                      2409
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                1958
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                    2273
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                       2980
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                 2317
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                     2321
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                      1657
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                1755
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                    2791
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                              2184
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer     2075
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost               1783
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                   2541
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer      2524
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                2138
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                    3141
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost     2175
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer            2670
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                      1654
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                          2732
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost           2278
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            3114
## 
## Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
## and Tail_ESS are effective sample size measures, and Rhat is the potential
## scale reduction factor on split chains (at convergence, Rhat = 1).
```

5: in model **`brm2.5`**, we add all 6 effects and their 5-way interactions.


```r
summary(brm2.5)
```

```
## Warning: Parts of the model have not converged (some Rhats are > 1.05). Be
## careful when analysing the results! We recommend running more iterations and/or
## setting stronger priors.
```

```
##  Family: categorical 
##   Links: muUFad = logit; muUFdis = logit 
## Formula: allocation ~ (age_group + gender + country + competition + relation + cost)^5 + (1 | id) 
##    Data: R2long (Number of observations: 1632) 
## Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
##          total post-warmup samples = 4000
## 
## Group-Level Effects: 
## ~id (Number of levels: 204) 
##                       Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS
## sd(muUFad_Intercept)      1.37      0.14     1.11     1.65 1.02       93
## sd(muUFdis_Intercept)     1.13      0.20     0.78     1.57 1.09       41
##                       Tail_ESS
## sd(muUFad_Intercept)       170
## sd(muUFdis_Intercept)      108
## 
## Population-Level Effects: 
##                                                                                              Estimate
## muUFad_Intercept                                                                                -0.08
## muUFdis_Intercept                                                                               -2.45
## muUFad_age_groupYounger                                                                         -0.56
## muUFad_genderMale                                                                                0.17
## muUFad_countryUK                                                                                 1.07
## muUFad_competitionNOcompetition                                                                 -0.83
## muUFad_relationUnknownMpeer                                                                     -0.76
## muUFad_costNOcost                                                                              -21.12
## muUFad_age_groupYounger:genderMale                                                              -0.40
## muUFad_age_groupYounger:countryUK                                                               -0.49
## muUFad_age_groupYounger:competitionNOcompetition                                                -1.16
## muUFad_age_groupYounger:relationUnknownMpeer                                                    -0.23
## muUFad_age_groupYounger:costNOcost                                                              20.05
## muUFad_genderMale:countryUK                                                                     -0.16
## muUFad_genderMale:competitionNOcompetition                                                       0.35
## muUFad_genderMale:relationUnknownMpeer                                                          -0.16
## muUFad_genderMale:costNOcost                                                                    19.47
## muUFad_countryUK:competitionNOcompetition                                                       -0.07
## muUFad_countryUK:relationUnknownMpeer                                                            0.49
## muUFad_countryUK:costNOcost                                                                     18.39
## muUFad_competitionNOcompetition:relationUnknownMpeer                                            -0.16
## muUFad_competitionNOcompetition:costNOcost                                                      19.70
## muUFad_relationUnknownMpeer:costNOcost                                                        -217.77
## muUFad_age_groupYounger:genderMale:countryUK                                                     1.86
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                      1.45
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                          1.70
## muUFad_age_groupYounger:genderMale:costNOcost                                                  -20.37
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                       1.94
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                           0.24
## muUFad_age_groupYounger:countryUK:costNOcost                                                   -19.40
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                            1.09
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                    -18.01
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                         61.78
## muUFad_genderMale:countryUK:competitionNOcompetition                                            -0.27
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                -1.93
## muUFad_genderMale:countryUK:costNOcost                                                         -21.99
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                  0.75
## muUFad_genderMale:competitionNOcompetition:costNOcost                                          -19.94
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                              216.89
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                   0.49
## muUFad_countryUK:competitionNOcompetition:costNOcost                                           -20.06
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                               218.71
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                160.27
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                           -3.02
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                               -0.89
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                         21.22
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                -1.47
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                          19.25
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                             -60.36
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                 -1.12
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                           16.12
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                              -62.67
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                -3.36
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                        0.37
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                 22.12
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                   -213.74
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                    -158.72
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                     -159.94
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer       0.41
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost               -17.17
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                    58.25
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost      0.95
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost       4.08
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost           156.37
## muUFdis_age_groupYounger                                                                        -0.81
## muUFdis_genderMale                                                                               1.22
## muUFdis_countryUK                                                                                0.03
## muUFdis_competitionNOcompetition                                                                 1.25
## muUFdis_relationUnknownMpeer                                                                     0.47
## muUFdis_costNOcost                                                                              -0.64
## muUFdis_age_groupYounger:genderMale                                                              0.77
## muUFdis_age_groupYounger:countryUK                                                            -165.67
## muUFdis_age_groupYounger:competitionNOcompetition                                                0.47
## muUFdis_age_groupYounger:relationUnknownMpeer                                                    0.30
## muUFdis_age_groupYounger:costNOcost                                                              0.61
## muUFdis_genderMale:countryUK                                                                     0.28
## muUFdis_genderMale:competitionNOcompetition                                                     -2.53
## muUFdis_genderMale:relationUnknownMpeer                                                        -47.23
## muUFdis_genderMale:costNOcost                                                                   -2.04
## muUFdis_countryUK:competitionNOcompetition                                                      -0.75
## muUFdis_countryUK:relationUnknownMpeer                                                           0.97
## muUFdis_countryUK:costNOcost                                                                    -0.47
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                           -1.54
## muUFdis_competitionNOcompetition:costNOcost                                                     -2.33
## muUFdis_relationUnknownMpeer:costNOcost                                                       -244.71
## muUFdis_age_groupYounger:genderMale:countryUK                                                  165.21
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                    -0.55
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                        45.81
## muUFdis_age_groupYounger:genderMale:costNOcost                                                   1.45
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                    164.18
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                        -44.01
## muUFdis_age_groupYounger:countryUK:costNOcost                                                  167.90
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                           0.33
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                     1.97
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                       244.50
## muUFdis_genderMale:countryUK:competitionNOcompetition                                           -0.39
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                               42.74
## muUFdis_genderMale:countryUK:costNOcost                                                          1.48
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                47.02
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                           3.31
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                             292.32
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                               -345.70
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                         -181.67
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                               39.24
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                               245.78
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                        -164.13
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                              -0.35
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                      -170.12
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer              -43.61
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                         -1.24
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                           -291.06
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer               390.39
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                          15.24
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                               0.62
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost             -244.89
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                      56.93
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                               181.44
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                   -83.70
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                   -528.86
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                     490.06
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer   -101.15
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost              -13.63
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                   44.66
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost   526.05
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost   -531.12
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost           38.01
##                                                                                              Est.Error
## muUFad_Intercept                                                                                  0.58
## muUFdis_Intercept                                                                                 0.80
## muUFad_age_groupYounger                                                                           0.90
## muUFad_genderMale                                                                                 0.84
## muUFad_countryUK                                                                                  0.92
## muUFad_competitionNOcompetition                                                                   0.76
## muUFad_relationUnknownMpeer                                                                       0.79
## muUFad_costNOcost                                                                                 9.53
## muUFad_age_groupYounger:genderMale                                                                1.24
## muUFad_age_groupYounger:countryUK                                                                 1.31
## muUFad_age_groupYounger:competitionNOcompetition                                                  1.27
## muUFad_age_groupYounger:relationUnknownMpeer                                                      1.19
## muUFad_age_groupYounger:costNOcost                                                                9.42
## muUFad_genderMale:countryUK                                                                       1.22
## muUFad_genderMale:competitionNOcompetition                                                        1.05
## muUFad_genderMale:relationUnknownMpeer                                                            1.06
## muUFad_genderMale:costNOcost                                                                      9.53
## muUFad_countryUK:competitionNOcompetition                                                         1.16
## muUFad_countryUK:relationUnknownMpeer                                                             1.14
## muUFad_countryUK:costNOcost                                                                       9.63
## muUFad_competitionNOcompetition:relationUnknownMpeer                                              1.18
## muUFad_competitionNOcompetition:costNOcost                                                        9.50
## muUFad_relationUnknownMpeer:costNOcost                                                          137.56
## muUFad_age_groupYounger:genderMale:countryUK                                                      1.62
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                       1.63
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                           1.54
## muUFad_age_groupYounger:genderMale:costNOcost                                                     9.42
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                        1.53
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                            1.56
## muUFad_age_groupYounger:countryUK:costNOcost                                                      9.61
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                             1.88
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                       9.41
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                          48.05
## muUFad_genderMale:countryUK:competitionNOcompetition                                              1.58
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                  1.60
## muUFad_genderMale:countryUK:costNOcost                                                            9.68
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                   1.50
## muUFad_genderMale:competitionNOcompetition:costNOcost                                             9.52
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                               137.64
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                    1.67
## muUFad_countryUK:competitionNOcompetition:costNOcost                                              9.68
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                137.62
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                 110.64
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                             2.01
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                 2.12
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                           9.60
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                  2.30
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                            9.44
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                               47.98
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                   2.15
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                             9.83
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                48.17
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                  4.31
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                         2.20
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                   9.79
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                     137.79
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                      110.81
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                       110.60
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer        2.78
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                  9.87
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                     48.06
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost       3.85
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost        3.32
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            110.92
## muUFdis_age_groupYounger                                                                          1.44
## muUFdis_genderMale                                                                                0.95
## muUFdis_countryUK                                                                                 1.46
## muUFdis_competitionNOcompetition                                                                  0.91
## muUFdis_relationUnknownMpeer                                                                      1.01
## muUFdis_costNOcost                                                                                1.02
## muUFdis_age_groupYounger:genderMale                                                               1.58
## muUFdis_age_groupYounger:countryUK                                                              162.34
## muUFdis_age_groupYounger:competitionNOcompetition                                                 1.57
## muUFdis_age_groupYounger:relationUnknownMpeer                                                     1.73
## muUFdis_age_groupYounger:costNOcost                                                               1.91
## muUFdis_genderMale:countryUK                                                                      1.66
## muUFdis_genderMale:competitionNOcompetition                                                       1.26
## muUFdis_genderMale:relationUnknownMpeer                                                          37.46
## muUFdis_genderMale:costNOcost                                                                     1.58
## muUFdis_countryUK:competitionNOcompetition                                                        1.76
## muUFdis_countryUK:relationUnknownMpeer                                                            1.74
## muUFdis_countryUK:costNOcost                                                                      2.03
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                             1.36
## muUFdis_competitionNOcompetition:costNOcost                                                       2.11
## muUFdis_relationUnknownMpeer:costNOcost                                                         174.06
## muUFdis_age_groupYounger:genderMale:countryUK                                                   162.24
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                      1.88
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                         37.83
## muUFdis_age_groupYounger:genderMale:costNOcost                                                    2.47
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                     162.40
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                         165.08
## muUFdis_age_groupYounger:countryUK:costNOcost                                                   162.14
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                            2.13
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                      2.99
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                        173.91
## muUFdis_genderMale:countryUK:competitionNOcompetition                                             2.37
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                37.39
## muUFdis_genderMale:countryUK:costNOcost                                                           2.61
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                 37.49
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                            2.97
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                              172.43
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                 278.13
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                           159.24
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                               203.44
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                173.73
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                          162.28
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                              165.43
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                        161.87
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                37.87
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                           3.90
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                             172.37
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                323.05
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                          204.93
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                              155.03
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost               173.47
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                      232.32
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                159.24
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                    205.61
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                     265.96
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                      369.65
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer     292.83
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost               204.83
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                   153.84
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost    265.97
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost     319.75
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost           165.05
##                                                                                              l-95% CI
## muUFad_Intercept                                                                                -1.26
## muUFdis_Intercept                                                                               -4.25
## muUFad_age_groupYounger                                                                         -2.59
## muUFad_genderMale                                                                               -1.45
## muUFad_countryUK                                                                                -0.73
## muUFad_competitionNOcompetition                                                                 -2.25
## muUFad_relationUnknownMpeer                                                                     -2.17
## muUFad_costNOcost                                                                              -44.01
## muUFad_age_groupYounger:genderMale                                                              -2.89
## muUFad_age_groupYounger:countryUK                                                               -3.13
## muUFad_age_groupYounger:competitionNOcompetition                                                -3.67
## muUFad_age_groupYounger:relationUnknownMpeer                                                    -2.44
## muUFad_age_groupYounger:costNOcost                                                               5.30
## muUFad_genderMale:countryUK                                                                     -2.65
## muUFad_genderMale:competitionNOcompetition                                                      -2.00
## muUFad_genderMale:relationUnknownMpeer                                                          -2.28
## muUFad_genderMale:costNOcost                                                                     4.63
## muUFad_countryUK:competitionNOcompetition                                                       -2.34
## muUFad_countryUK:relationUnknownMpeer                                                           -1.42
## muUFad_countryUK:costNOcost                                                                      3.36
## muUFad_competitionNOcompetition:relationUnknownMpeer                                            -2.44
## muUFad_competitionNOcompetition:costNOcost                                                       5.00
## muUFad_relationUnknownMpeer:costNOcost                                                        -526.58
## muUFad_age_groupYounger:genderMale:countryUK                                                    -1.54
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                     -1.48
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                         -1.68
## muUFad_age_groupYounger:genderMale:costNOcost                                                  -43.18
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                      -1.05
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                          -2.90
## muUFad_age_groupYounger:countryUK:costNOcost                                                   -43.01
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                           -2.14
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                    -40.75
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                          7.72
## muUFad_genderMale:countryUK:competitionNOcompetition                                            -3.08
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                -5.31
## muUFad_genderMale:countryUK:costNOcost                                                         -45.45
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                 -1.95
## muUFad_genderMale:competitionNOcompetition:costNOcost                                          -42.91
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                               38.97
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                  -3.49
## muUFad_countryUK:competitionNOcompetition:costNOcost                                           -43.05
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                41.30
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                 13.19
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                           -6.93
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                               -4.83
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                          5.87
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                -6.10
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                           5.15
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                            -187.48
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                 -5.59
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                            0.81
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                             -189.72
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost               -12.20
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                       -3.48
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                  6.54
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                   -523.67
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                    -376.00
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                     -376.54
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer      -4.61
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost               -40.54
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                     4.21
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost     -6.28
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost      -2.88
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost             8.92
## muUFdis_age_groupYounger                                                                        -3.66
## muUFdis_genderMale                                                                              -0.64
## muUFdis_countryUK                                                                               -3.20
## muUFdis_competitionNOcompetition                                                                -0.44
## muUFdis_relationUnknownMpeer                                                                    -1.34
## muUFdis_costNOcost                                                                              -2.54
## muUFdis_age_groupYounger:genderMale                                                             -2.24
## muUFdis_age_groupYounger:countryUK                                                            -719.17
## muUFdis_age_groupYounger:competitionNOcompetition                                               -2.60
## muUFdis_age_groupYounger:relationUnknownMpeer                                                   -2.93
## muUFdis_age_groupYounger:costNOcost                                                             -3.58
## muUFdis_genderMale:countryUK                                                                    -2.86
## muUFdis_genderMale:competitionNOcompetition                                                     -5.13
## muUFdis_genderMale:relationUnknownMpeer                                                       -127.10
## muUFdis_genderMale:costNOcost                                                                   -5.26
## muUFdis_countryUK:competitionNOcompetition                                                      -4.02
## muUFdis_countryUK:relationUnknownMpeer                                                          -2.26
## muUFdis_countryUK:costNOcost                                                                    -4.32
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                           -4.49
## muUFdis_competitionNOcompetition:costNOcost                                                     -7.49
## muUFdis_relationUnknownMpeer:costNOcost                                                       -620.35
## muUFdis_age_groupYounger:genderMale:countryUK                                                    8.86
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                    -4.22
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                         1.45
## muUFdis_age_groupYounger:genderMale:costNOcost                                                  -2.72
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                      6.62
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                       -472.89
## muUFdis_age_groupYounger:countryUK:costNOcost                                                   10.62
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                          -3.97
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                    -3.07
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                        18.99
## muUFdis_genderMale:countryUK:competitionNOcompetition                                           -5.41
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                               -1.60
## muUFdis_genderMale:countryUK:costNOcost                                                         -3.67
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                 3.07
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                          -2.28
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                              96.05
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                              -1063.28
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                         -544.89
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                             -297.87
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                19.91
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                        -716.80
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                            -220.97
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                      -720.93
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer             -123.42
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                         -9.69
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                           -685.02
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer               -97.85
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                        -513.76
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                            -284.27
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost             -618.85
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                    -350.31
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                 4.35
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                  -577.89
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                  -1106.47
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                    -105.95
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer   -662.44
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost             -433.21
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                 -235.02
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost   178.71
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost  -1395.93
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost         -184.00
##                                                                                              u-95% CI
## muUFad_Intercept                                                                                 0.97
## muUFdis_Intercept                                                                               -1.06
## muUFad_age_groupYounger                                                                          1.11
## muUFad_genderMale                                                                                1.99
## muUFad_countryUK                                                                                 2.74
## muUFad_competitionNOcompetition                                                                  0.74
## muUFad_relationUnknownMpeer                                                                      0.81
## muUFad_costNOcost                                                                               -6.58
## muUFad_age_groupYounger:genderMale                                                               2.32
## muUFad_age_groupYounger:countryUK                                                                1.96
## muUFad_age_groupYounger:competitionNOcompetition                                                 1.12
## muUFad_age_groupYounger:relationUnknownMpeer                                                     2.24
## muUFad_age_groupYounger:costNOcost                                                              42.99
## muUFad_genderMale:countryUK                                                                      2.07
## muUFad_genderMale:competitionNOcompetition                                                       2.27
## muUFad_genderMale:relationUnknownMpeer                                                           1.74
## muUFad_genderMale:costNOcost                                                                    42.53
## muUFad_countryUK:competitionNOcompetition                                                        2.19
## muUFad_countryUK:relationUnknownMpeer                                                            3.25
## muUFad_countryUK:costNOcost                                                                     41.58
## muUFad_competitionNOcompetition:relationUnknownMpeer                                             1.97
## muUFad_competitionNOcompetition:costNOcost                                                      42.38
## muUFad_relationUnknownMpeer:costNOcost                                                         -40.60
## muUFad_age_groupYounger:genderMale:countryUK                                                     5.32
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                      4.94
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                          4.45
## muUFad_age_groupYounger:genderMale:costNOcost                                                   -5.45
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                       5.31
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                           3.07
## muUFad_age_groupYounger:countryUK:costNOcost                                                    -4.19
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                            4.94
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                     -4.05
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                        189.14
## muUFad_genderMale:countryUK:competitionNOcompetition                                             2.86
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                 0.96
## muUFad_genderMale:countryUK:costNOcost                                                          -6.82
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                  3.61
## muUFad_genderMale:competitionNOcompetition:costNOcost                                           -5.44
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                              526.07
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                   3.63
## muUFad_countryUK:competitionNOcompetition:costNOcost                                            -4.90
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                               528.06
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                376.81
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                            0.83
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                3.50
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                         44.44
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                 2.62
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                          42.27
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                              -6.43
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                  2.59
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                           39.51
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                               -8.37
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                 4.87
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                        5.14
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                 45.64
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                    -35.85
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                     -11.26
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                      -13.41
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer       5.79
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                -2.04
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                   182.89
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost      8.70
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost      10.40
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost           375.61
## muUFdis_age_groupYounger                                                                         1.85
## muUFdis_genderMale                                                                               3.07
## muUFdis_countryUK                                                                                2.58
## muUFdis_competitionNOcompetition                                                                 3.22
## muUFdis_relationUnknownMpeer                                                                     2.72
## muUFdis_costNOcost                                                                               1.38
## muUFdis_age_groupYounger:genderMale                                                              3.81
## muUFdis_age_groupYounger:countryUK                                                              -9.11
## muUFdis_age_groupYounger:competitionNOcompetition                                                3.80
## muUFdis_age_groupYounger:relationUnknownMpeer                                                    3.66
## muUFdis_age_groupYounger:costNOcost                                                              3.98
## muUFdis_genderMale:countryUK                                                                     3.82
## muUFdis_genderMale:competitionNOcompetition                                                     -0.08
## muUFdis_genderMale:relationUnknownMpeer                                                         -3.62
## muUFdis_genderMale:costNOcost                                                                    0.96
## muUFdis_countryUK:competitionNOcompetition                                                       2.91
## muUFdis_countryUK:relationUnknownMpeer                                                           4.57
## muUFdis_countryUK:costNOcost                                                                     3.61
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                            0.95
## muUFdis_competitionNOcompetition:costNOcost                                                      0.89
## muUFdis_relationUnknownMpeer:costNOcost                                                        -18.38
## muUFdis_age_groupYounger:genderMale:countryUK                                                  718.64
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                     2.91
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                       125.78
## muUFdis_age_groupYounger:genderMale:costNOcost                                                   6.64
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                    716.49
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                        178.18
## muUFdis_age_groupYounger:countryUK:costNOcost                                                  720.14
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                           4.41
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                     8.81
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                       619.13
## muUFdis_genderMale:countryUK:competitionNOcompetition                                            3.93
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                              122.20
## muUFdis_genderMale:countryUK:costNOcost                                                          6.71
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                               126.82
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                           9.59
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                             685.85
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                -17.18
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                           -4.43
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                              531.71
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                               619.26
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                          -6.69
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                             427.15
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                       -13.22
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                1.53
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                          5.53
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                            -94.88
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer              1094.95
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                         433.47
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                             272.64
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost              -19.03
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                     631.80
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                               547.85
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                   263.55
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                   -180.57
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                    1456.16
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer    400.79
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost              511.83
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                  338.75
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost  1104.80
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost   -122.54
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost          469.12
##                                                                                              Rhat
## muUFad_Intercept                                                                             1.16
## muUFdis_Intercept                                                                            1.08
## muUFad_age_groupYounger                                                                      1.25
## muUFad_genderMale                                                                            1.17
## muUFad_countryUK                                                                             1.18
## muUFad_competitionNOcompetition                                                              1.22
## muUFad_relationUnknownMpeer                                                                  1.21
## muUFad_costNOcost                                                                            1.65
## muUFad_age_groupYounger:genderMale                                                           1.20
## muUFad_age_groupYounger:countryUK                                                            1.32
## muUFad_age_groupYounger:competitionNOcompetition                                             1.18
## muUFad_age_groupYounger:relationUnknownMpeer                                                 1.20
## muUFad_age_groupYounger:costNOcost                                                           1.66
## muUFad_genderMale:countryUK                                                                  1.25
## muUFad_genderMale:competitionNOcompetition                                                   1.18
## muUFad_genderMale:relationUnknownMpeer                                                       1.23
## muUFad_genderMale:costNOcost                                                                 1.65
## muUFad_countryUK:competitionNOcompetition                                                    1.23
## muUFad_countryUK:relationUnknownMpeer                                                        1.21
## muUFad_countryUK:costNOcost                                                                  1.62
## muUFad_competitionNOcompetition:relationUnknownMpeer                                         1.23
## muUFad_competitionNOcompetition:costNOcost                                                   1.64
## muUFad_relationUnknownMpeer:costNOcost                                                       2.05
## muUFad_age_groupYounger:genderMale:countryUK                                                 1.30
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                  1.18
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                      1.22
## muUFad_age_groupYounger:genderMale:costNOcost                                                1.65
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                   1.22
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                       1.22
## muUFad_age_groupYounger:countryUK:costNOcost                                                 1.62
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                        1.15
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                  1.68
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                      1.30
## muUFad_genderMale:countryUK:competitionNOcompetition                                         1.21
## muUFad_genderMale:countryUK:relationUnknownMpeer                                             1.24
## muUFad_genderMale:countryUK:costNOcost                                                       1.57
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                              1.22
## muUFad_genderMale:competitionNOcompetition:costNOcost                                        1.63
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                            2.05
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                               1.24
## muUFad_countryUK:competitionNOcompetition:costNOcost                                         1.58
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                             2.04
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                              2.16
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                        1.24
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                            1.28
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                      1.54
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer             1.16
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                       1.64
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                           1.30
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer              1.19
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                        1.59
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                            1.30
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost             1.06
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                    1.25
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                              1.50
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                  2.05
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                   2.17
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                    2.16
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer   1.24
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost             1.47
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                 1.30
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost  1.04
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost   1.04
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost         2.17
## muUFdis_age_groupYounger                                                                     1.11
## muUFdis_genderMale                                                                           1.04
## muUFdis_countryUK                                                                            1.11
## muUFdis_competitionNOcompetition                                                             1.08
## muUFdis_relationUnknownMpeer                                                                 1.07
## muUFdis_costNOcost                                                                           1.09
## muUFdis_age_groupYounger:genderMale                                                          1.09
## muUFdis_age_groupYounger:countryUK                                                           1.75
## muUFdis_age_groupYounger:competitionNOcompetition                                            1.08
## muUFdis_age_groupYounger:relationUnknownMpeer                                                1.05
## muUFdis_age_groupYounger:costNOcost                                                          1.11
## muUFdis_genderMale:countryUK                                                                 1.08
## muUFdis_genderMale:competitionNOcompetition                                                  1.06
## muUFdis_genderMale:relationUnknownMpeer                                                      1.78
## muUFdis_genderMale:costNOcost                                                                1.03
## muUFdis_countryUK:competitionNOcompetition                                                   1.09
## muUFdis_countryUK:relationUnknownMpeer                                                       1.10
## muUFdis_countryUK:costNOcost                                                                 1.10
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                        1.06
## muUFdis_competitionNOcompetition:costNOcost                                                  1.09
## muUFdis_relationUnknownMpeer:costNOcost                                                      2.84
## muUFdis_age_groupYounger:genderMale:countryUK                                                1.74
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                 1.07
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                     1.80
## muUFdis_age_groupYounger:genderMale:costNOcost                                               1.10
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                  1.75
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                      1.63
## muUFdis_age_groupYounger:countryUK:costNOcost                                                1.74
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                       1.06
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                 1.14
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                     2.84
## muUFdis_genderMale:countryUK:competitionNOcompetition                                        1.08
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                            1.77
## muUFdis_genderMale:countryUK:costNOcost                                                      1.05
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                             1.77
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                       1.11
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                           2.26
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                              1.24
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                        1.28
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                            1.85
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                             2.82
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                       1.75
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                           1.64
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                     1.74
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer            1.80
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                      1.14
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                          2.25
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer             1.54
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                       1.58
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                           1.21
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost            2.81
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                   1.19
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                             1.27
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                 1.83
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                  1.70
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                   1.33
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer  1.56
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost            1.58
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                1.21
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost 1.70
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost  1.29
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost        1.62
##                                                                                              Bulk_ESS
## muUFad_Intercept                                                                                   22
## muUFdis_Intercept                                                                                  57
## muUFad_age_groupYounger                                                                            14
## muUFad_genderMale                                                                                  21
## muUFad_countryUK                                                                                   18
## muUFad_competitionNOcompetition                                                                    15
## muUFad_relationUnknownMpeer                                                                        18
## muUFad_costNOcost                                                                                   7
## muUFad_age_groupYounger:genderMale                                                                 17
## muUFad_age_groupYounger:countryUK                                                                  10
## muUFad_age_groupYounger:competitionNOcompetition                                                   16
## muUFad_age_groupYounger:relationUnknownMpeer                                                       16
## muUFad_age_groupYounger:costNOcost                                                                  7
## muUFad_genderMale:countryUK                                                                        13
## muUFad_genderMale:competitionNOcompetition                                                         19
## muUFad_genderMale:relationUnknownMpeer                                                             13
## muUFad_genderMale:costNOcost                                                                        7
## muUFad_countryUK:competitionNOcompetition                                                          13
## muUFad_countryUK:relationUnknownMpeer                                                              14
## muUFad_countryUK:costNOcost                                                                         7
## muUFad_competitionNOcompetition:relationUnknownMpeer                                               13
## muUFad_competitionNOcompetition:costNOcost                                                          7
## muUFad_relationUnknownMpeer:costNOcost                                                              5
## muUFad_age_groupYounger:genderMale:countryUK                                                       11
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                        16
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                            15
## muUFad_age_groupYounger:genderMale:costNOcost                                                       7
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                         14
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                             13
## muUFad_age_groupYounger:countryUK:costNOcost                                                        7
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                              20
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                         7
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                            11
## muUFad_genderMale:countryUK:competitionNOcompetition                                               14
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                   12
## muUFad_genderMale:countryUK:costNOcost                                                              7
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                    14
## muUFad_genderMale:competitionNOcompetition:costNOcost                                               7
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                   5
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                     12
## muUFad_countryUK:competitionNOcompetition:costNOcost                                                7
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                    5
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                     5
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                              12
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                  11
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                             7
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                   19
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                              7
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                 11
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                    15
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                               7
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                  11
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                   62
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                          12
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                     8
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                         5
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                          5
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                           5
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer         12
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                    8
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                       10
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost        68
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost         68
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                5
## muUFdis_age_groupYounger                                                                           31
## muUFdis_genderMale                                                                                 88
## muUFdis_countryUK                                                                                  29
## muUFdis_competitionNOcompetition                                                                   57
## muUFdis_relationUnknownMpeer                                                                       64
## muUFdis_costNOcost                                                                                 32
## muUFdis_age_groupYounger:genderMale                                                                36
## muUFdis_age_groupYounger:countryUK                                                                  6
## muUFdis_age_groupYounger:competitionNOcompetition                                                  39
## muUFdis_age_groupYounger:relationUnknownMpeer                                                      59
## muUFdis_age_groupYounger:costNOcost                                                                25
## muUFdis_genderMale:countryUK                                                                       74
## muUFdis_genderMale:competitionNOcompetition                                                        86
## muUFdis_genderMale:relationUnknownMpeer                                                             6
## muUFdis_genderMale:costNOcost                                                                      94
## muUFdis_countryUK:competitionNOcompetition                                                         41
## muUFdis_countryUK:relationUnknownMpeer                                                             38
## muUFdis_countryUK:costNOcost                                                                       27
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                              63
## muUFdis_competitionNOcompetition:costNOcost                                                        32
## muUFdis_relationUnknownMpeer:costNOcost                                                             5
## muUFdis_age_groupYounger:genderMale:countryUK                                                       6
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                       67
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                            6
## muUFdis_age_groupYounger:genderMale:costNOcost                                                     28
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                         6
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                             7
## muUFdis_age_groupYounger:countryUK:costNOcost                                                       6
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                             56
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                       21
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                            5
## muUFdis_genderMale:countryUK:competitionNOcompetition                                              49
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                   6
## muUFdis_genderMale:countryUK:costNOcost                                                            65
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                    6
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                             27
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                  5
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                    14
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                              12
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                   6
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                    5
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                              6
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                  7
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                            6
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                   6
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                            21
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                 5
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                    7
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                              7
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                 17
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                   5
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                         21
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                   12
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                        6
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                         6
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                         10
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer         7
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                   7
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                      17
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost        6
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost        11
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost               7
##                                                                                              Tail_ESS
## muUFad_Intercept                                                                                   77
## muUFdis_Intercept                                                                                 112
## muUFad_age_groupYounger                                                                            41
## muUFad_genderMale                                                                                  50
## muUFad_countryUK                                                                                   59
## muUFad_competitionNOcompetition                                                                    68
## muUFad_relationUnknownMpeer                                                                        72
## muUFad_costNOcost                                                                                  19
## muUFad_age_groupYounger:genderMale                                                                 37
## muUFad_age_groupYounger:countryUK                                                                  18
## muUFad_age_groupYounger:competitionNOcompetition                                                   46
## muUFad_age_groupYounger:relationUnknownMpeer                                                       38
## muUFad_age_groupYounger:costNOcost                                                                 19
## muUFad_genderMale:countryUK                                                                        22
## muUFad_genderMale:competitionNOcompetition                                                         48
## muUFad_genderMale:relationUnknownMpeer                                                             76
## muUFad_genderMale:costNOcost                                                                       21
## muUFad_countryUK:competitionNOcompetition                                                          40
## muUFad_countryUK:relationUnknownMpeer                                                              25
## muUFad_countryUK:costNOcost                                                                        22
## muUFad_competitionNOcompetition:relationUnknownMpeer                                               96
## muUFad_competitionNOcompetition:costNOcost                                                         20
## muUFad_relationUnknownMpeer:costNOcost                                                             19
## muUFad_age_groupYounger:genderMale:countryUK                                                       17
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                       102
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                            19
## muUFad_age_groupYounger:genderMale:costNOcost                                                      22
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                         38
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                             63
## muUFad_age_groupYounger:countryUK:costNOcost                                                       20
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                             103
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                        17
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                            22
## muUFad_genderMale:countryUK:competitionNOcompetition                                               76
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                   34
## muUFad_genderMale:countryUK:costNOcost                                                             25
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                   188
## muUFad_genderMale:competitionNOcompetition:costNOcost                                              24
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                  19
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                     29
## muUFad_countryUK:competitionNOcompetition:costNOcost                                               25
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                   19
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                    13
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                              34
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                  48
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                            25
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                   90
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                             22
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                 20
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                    70
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                              24
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                  25
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                   83
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                          36
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                    27
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                        19
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                         13
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                          13
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer         79
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                   30
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                       24
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost        94
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost        140
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost               13
## muUFdis_age_groupYounger                                                                          110
## muUFdis_genderMale                                                                                188
## muUFdis_countryUK                                                                                 103
## muUFdis_competitionNOcompetition                                                                   52
## muUFdis_relationUnknownMpeer                                                                      115
## muUFdis_costNOcost                                                                                 84
## muUFdis_age_groupYounger:genderMale                                                               134
## muUFdis_age_groupYounger:countryUK                                                                 12
## muUFdis_age_groupYounger:competitionNOcompetition                                                 118
## muUFdis_age_groupYounger:relationUnknownMpeer                                                     183
## muUFdis_age_groupYounger:costNOcost                                                               130
## muUFdis_genderMale:countryUK                                                                      131
## muUFdis_genderMale:competitionNOcompetition                                                       170
## muUFdis_genderMale:relationUnknownMpeer                                                            34
## muUFdis_genderMale:costNOcost                                                                     203
## muUFdis_countryUK:competitionNOcompetition                                                        124
## muUFdis_countryUK:relationUnknownMpeer                                                            157
## muUFdis_countryUK:costNOcost                                                                      194
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                             186
## muUFdis_competitionNOcompetition:costNOcost                                                        80
## muUFdis_relationUnknownMpeer:costNOcost                                                            13
## muUFdis_age_groupYounger:genderMale:countryUK                                                      12
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                      157
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                           33
## muUFdis_age_groupYounger:genderMale:costNOcost                                                    172
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                        12
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                            24
## muUFdis_age_groupYounger:countryUK:costNOcost                                                      12
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                            196
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                       91
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                           14
## muUFdis_genderMale:countryUK:competitionNOcompetition                                             175
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                  39
## muUFdis_genderMale:countryUK:costNOcost                                                           139
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                   31
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                             82
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                 29
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                    28
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                              31
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                  13
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                   14
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                             12
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                 26
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                           12
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                  33
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                           104
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                29
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                   28
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                             12
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                 58
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                  14
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                         37
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                   32
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                       14
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                        25
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                         13
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer        28
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                  12
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                      48
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost       25
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost        13
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost              26
## 
## Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
## and Tail_ESS are effective sample size measures, and Rhat is the potential
## scale reduction factor on split chains (at convergence, Rhat = 1).
```

6: in model **`brm2.6`**, we add all 6 effects and their 6-way interactions.


```r
summary(brm2.6)
```

```
## Warning: Parts of the model have not converged (some Rhats are > 1.05). Be
## careful when analysing the results! We recommend running more iterations and/or
## setting stronger priors.
```

```
##  Family: categorical 
##   Links: muUFad = logit; muUFdis = logit 
## Formula: allocation ~ (age_group + gender + country + competition + relation + cost)^6 + (1 | id) 
##    Data: R2long (Number of observations: 1632) 
## Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
##          total post-warmup samples = 4000
## 
## Group-Level Effects: 
## ~id (Number of levels: 204) 
##                       Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS
## sd(muUFad_Intercept)      1.42      0.13     1.18     1.67 1.06       57
## sd(muUFdis_Intercept)     1.20      0.17     0.88     1.55 1.11       36
##                       Tail_ESS
## sd(muUFad_Intercept)       167
## sd(muUFdis_Intercept)       85
## 
## Population-Level Effects: 
##                                                                                                        Estimate
## muUFad_Intercept                                                                                          -0.00
## muUFdis_Intercept                                                                                         -2.13
## muUFad_age_groupYounger                                                                                   -0.55
## muUFad_genderMale                                                                                         -0.09
## muUFad_countryUK                                                                                           0.98
## muUFad_competitionNOcompetition                                                                           -1.05
## muUFad_relationUnknownMpeer                                                                               -1.01
## muUFad_costNOcost                                                                                        -28.53
## muUFad_age_groupYounger:genderMale                                                                        -0.25
## muUFad_age_groupYounger:countryUK                                                                         -0.58
## muUFad_age_groupYounger:competitionNOcompetition                                                          -0.92
## muUFad_age_groupYounger:relationUnknownMpeer                                                               0.09
## muUFad_age_groupYounger:costNOcost                                                                        27.42
## muUFad_genderMale:countryUK                                                                                0.05
## muUFad_genderMale:competitionNOcompetition                                                                 0.83
## muUFad_genderMale:relationUnknownMpeer                                                                     0.34
## muUFad_genderMale:costNOcost                                                                              27.07
## muUFad_countryUK:competitionNOcompetition                                                                  0.08
## muUFad_countryUK:relationUnknownMpeer                                                                      0.61
## muUFad_countryUK:costNOcost                                                                               25.68
## muUFad_competitionNOcompetition:relationUnknownMpeer                                                       0.21
## muUFad_competitionNOcompetition:costNOcost                                                                27.12
## muUFad_relationUnknownMpeer:costNOcost                                                                  -137.45
## muUFad_age_groupYounger:genderMale:countryUK                                                               1.77
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                                0.94
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                                    1.13
## muUFad_age_groupYounger:genderMale:costNOcost                                                            -27.98
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                                 1.78
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                                     0.14
## muUFad_age_groupYounger:countryUK:costNOcost                                                             -26.60
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                      0.56
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                              -25.45
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                                   35.34
## muUFad_genderMale:countryUK:competitionNOcompetition                                                      -0.67
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                          -2.27
## muUFad_genderMale:countryUK:costNOcost                                                                   -29.47
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                            0.01
## muUFad_genderMale:competitionNOcompetition:costNOcost                                                    -27.63
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                        136.31
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                             0.18
## muUFad_countryUK:competitionNOcompetition:costNOcost                                                     -27.38
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                         138.55
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                           87.31
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                     -2.60
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                         -0.59
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                                   28.63
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                          -0.55
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                    26.98
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                       -33.65
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                           -0.66
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                     23.38
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                        -36.47
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                          15.81
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                  1.07
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                           29.72
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                             -133.36
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                               -85.38
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                                -87.07
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                -0.46
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                         -24.72
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                              31.90
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost              -18.62
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost               -14.92
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                      83.09
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost     19.38
## muUFdis_age_groupYounger                                                                                  -1.40
## muUFdis_genderMale                                                                                         0.56
## muUFdis_countryUK                                                                                         -0.08
## muUFdis_competitionNOcompetition                                                                           0.74
## muUFdis_relationUnknownMpeer                                                                              -0.03
## muUFdis_costNOcost                                                                                        -0.71
## muUFdis_age_groupYounger:genderMale                                                                        1.72
## muUFdis_age_groupYounger:countryUK                                                                      -253.79
## muUFdis_age_groupYounger:competitionNOcompetition                                                          1.36
## muUFdis_age_groupYounger:relationUnknownMpeer                                                              1.13
## muUFdis_age_groupYounger:costNOcost                                                                        0.52
## muUFdis_genderMale:countryUK                                                                               0.69
## muUFdis_genderMale:competitionNOcompetition                                                               -1.58
## muUFdis_genderMale:relationUnknownMpeer                                                                  -34.49
## muUFdis_genderMale:costNOcost                                                                             -1.67
## muUFdis_countryUK:competitionNOcompetition                                                                -0.55
## muUFdis_countryUK:relationUnknownMpeer                                                                     1.18
## muUFdis_countryUK:costNOcost                                                                              -0.92
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                                     -0.81
## muUFdis_competitionNOcompetition:costNOcost                                                               -1.87
## muUFdis_relationUnknownMpeer:costNOcost                                                                  -93.61
## muUFdis_age_groupYounger:genderMale:countryUK                                                            253.05
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                              -1.88
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                                  32.72
## muUFdis_age_groupYounger:genderMale:costNOcost                                                             1.25
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                              251.87
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                                  -10.37
## muUFdis_age_groupYounger:countryUK:costNOcost                                                            256.90
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                    -0.91
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                               1.53
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                                  93.42
## muUFdis_genderMale:countryUK:competitionNOcompetition                                                     -1.19
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                         30.33
## muUFdis_genderMale:countryUK:costNOcost                                                                    1.66
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                          33.84
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                                     2.41
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                       128.97
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                         -440.71
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                                   -145.97
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                       -159.05
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                          94.21
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                  -251.28
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                       -21.84
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                                -258.89
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                        -29.90
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                   -0.38
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                     -127.72
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                         451.80
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                  -108.79
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                       164.59
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                        -93.19
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                 5.14
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                         146.36
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                             126.80
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                             -422.66
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                               747.77
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer              -15.70
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                        109.85
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                           -131.45
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost             419.70
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost             -754.68
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                    -15.09
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost    18.90
##                                                                                                        Est.Error
## muUFad_Intercept                                                                                            0.61
## muUFdis_Intercept                                                                                           1.03
## muUFad_age_groupYounger                                                                                     0.84
## muUFad_genderMale                                                                                           0.79
## muUFad_countryUK                                                                                            0.93
## muUFad_competitionNOcompetition                                                                             0.82
## muUFad_relationUnknownMpeer                                                                                 0.74
## muUFad_costNOcost                                                                                          17.17
## muUFad_age_groupYounger:genderMale                                                                          1.20
## muUFad_age_groupYounger:countryUK                                                                           1.21
## muUFad_age_groupYounger:competitionNOcompetition                                                            1.53
## muUFad_age_groupYounger:relationUnknownMpeer                                                                1.04
## muUFad_age_groupYounger:costNOcost                                                                         17.38
## muUFad_genderMale:countryUK                                                                                 1.12
## muUFad_genderMale:competitionNOcompetition                                                                  0.97
## muUFad_genderMale:relationUnknownMpeer                                                                      0.97
## muUFad_genderMale:costNOcost                                                                               17.14
## muUFad_countryUK:competitionNOcompetition                                                                   1.02
## muUFad_countryUK:relationUnknownMpeer                                                                       1.02
## muUFad_countryUK:costNOcost                                                                                17.27
## muUFad_competitionNOcompetition:relationUnknownMpeer                                                        1.08
## muUFad_competitionNOcompetition:costNOcost                                                                 17.37
## muUFad_relationUnknownMpeer:costNOcost                                                                     98.90
## muUFad_age_groupYounger:genderMale:countryUK                                                                1.70
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                                 1.88
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                                     1.39
## muUFad_age_groupYounger:genderMale:costNOcost                                                              17.49
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                                  1.76
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                                      1.26
## muUFad_age_groupYounger:countryUK:costNOcost                                                               17.57
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                       2.14
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                                17.78
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                                   123.90
## muUFad_genderMale:countryUK:competitionNOcompetition                                                        1.13
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                            1.33
## muUFad_genderMale:countryUK:costNOcost                                                                     17.39
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                             1.27
## muUFad_genderMale:competitionNOcompetition:costNOcost                                                      17.37
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                          98.88
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                              1.33
## muUFad_countryUK:competitionNOcompetition:costNOcost                                                       17.61
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                           98.78
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                           107.82
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                       2.17
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                           1.77
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                                    17.88
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                            2.52
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                     17.97
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                        123.82
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                             2.30
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                      18.18
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                         123.67
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                          135.42
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                   1.62
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                            17.76
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                                98.67
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                                107.85
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                                 107.58
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                  2.87
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                           18.60
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                              123.33
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost               135.41
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                134.99
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                      107.51
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost     134.70
## muUFdis_age_groupYounger                                                                                    1.67
## muUFdis_genderMale                                                                                          1.40
## muUFdis_countryUK                                                                                           1.46
## muUFdis_competitionNOcompetition                                                                            1.11
## muUFdis_relationUnknownMpeer                                                                                1.18
## muUFdis_costNOcost                                                                                          1.29
## muUFdis_age_groupYounger:genderMale                                                                         1.97
## muUFdis_age_groupYounger:countryUK                                                                        317.40
## muUFdis_age_groupYounger:competitionNOcompetition                                                           1.70
## muUFdis_age_groupYounger:relationUnknownMpeer                                                               1.93
## muUFdis_age_groupYounger:costNOcost                                                                         1.94
## muUFdis_genderMale:countryUK                                                                                1.79
## muUFdis_genderMale:competitionNOcompetition                                                                 1.63
## muUFdis_genderMale:relationUnknownMpeer                                                                    38.82
## muUFdis_genderMale:costNOcost                                                                               1.93
## muUFdis_countryUK:competitionNOcompetition                                                                  1.76
## muUFdis_countryUK:relationUnknownMpeer                                                                      1.68
## muUFdis_countryUK:costNOcost                                                                                2.10
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                                       1.50
## muUFdis_competitionNOcompetition:costNOcost                                                                 2.00
## muUFdis_relationUnknownMpeer:costNOcost                                                                    65.23
## muUFdis_age_groupYounger:genderMale:countryUK                                                             317.54
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                                2.16
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                                   38.77
## muUFdis_age_groupYounger:genderMale:costNOcost                                                              2.37
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                               317.63
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                                   357.07
## muUFdis_age_groupYounger:countryUK:costNOcost                                                             317.23
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                      2.25
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                                2.29
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                                   65.50
## muUFdis_genderMale:countryUK:competitionNOcompetition                                                       2.37
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                          38.75
## muUFdis_genderMale:countryUK:costNOcost                                                                     2.79
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                           39.16
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                                      2.75
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                         65.06
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                           398.68
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                                     102.16
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                         272.29
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                           64.80
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                    317.75
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                        348.17
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                                  317.26
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                          39.24
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                     2.99
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                        65.28
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                          402.91
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                    310.24
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                        298.82
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                          64.98
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                611.13
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                          102.19
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                              265.60
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                               274.38
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                                623.08
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer               612.64
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                         310.06
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                             279.09
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost              274.15
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost               505.57
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                     778.67
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost    701.05
##                                                                                                        l-95% CI
## muUFad_Intercept                                                                                          -1.24
## muUFdis_Intercept                                                                                         -4.19
## muUFad_age_groupYounger                                                                                   -2.11
## muUFad_genderMale                                                                                         -1.52
## muUFad_countryUK                                                                                          -0.94
## muUFad_competitionNOcompetition                                                                           -2.40
## muUFad_relationUnknownMpeer                                                                               -2.44
## muUFad_costNOcost                                                                                        -57.06
## muUFad_age_groupYounger:genderMale                                                                        -2.52
## muUFad_age_groupYounger:countryUK                                                                         -2.67
## muUFad_age_groupYounger:competitionNOcompetition                                                          -4.24
## muUFad_age_groupYounger:relationUnknownMpeer                                                              -1.86
## muUFad_age_groupYounger:costNOcost                                                                         2.69
## muUFad_genderMale:countryUK                                                                               -1.90
## muUFad_genderMale:competitionNOcompetition                                                                -0.96
## muUFad_genderMale:relationUnknownMpeer                                                                    -1.78
## muUFad_genderMale:costNOcost                                                                               2.89
## muUFad_countryUK:competitionNOcompetition                                                                 -1.97
## muUFad_countryUK:relationUnknownMpeer                                                                     -1.26
## muUFad_countryUK:costNOcost                                                                                1.30
## muUFad_competitionNOcompetition:relationUnknownMpeer                                                      -1.93
## muUFad_competitionNOcompetition:costNOcost                                                                 2.05
## muUFad_relationUnknownMpeer:costNOcost                                                                  -282.29
## muUFad_age_groupYounger:genderMale:countryUK                                                              -1.39
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                               -2.34
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                                   -1.43
## muUFad_age_groupYounger:genderMale:costNOcost                                                            -56.51
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                                -1.54
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                                    -2.20
## muUFad_age_groupYounger:countryUK:costNOcost                                                             -54.44
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                     -2.85
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                              -54.39
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                                 -188.36
## muUFad_genderMale:countryUK:competitionNOcompetition                                                      -2.71
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                          -4.58
## muUFad_genderMale:countryUK:costNOcost                                                                   -57.34
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                           -2.26
## muUFad_genderMale:competitionNOcompetition:costNOcost                                                    -56.07
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                        -18.66
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                            -2.48
## muUFad_countryUK:competitionNOcompetition:costNOcost                                                     -55.97
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                         -16.37
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                         -106.88
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                     -6.72
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                         -4.34
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                                    3.01
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                          -6.47
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                     0.08
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                      -208.35
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                           -5.79
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                     -4.02
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                       -210.52
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                        -179.99
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                 -1.92
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                            3.50
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                             -279.07
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                              -242.99
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                               -244.35
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                -5.42
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                         -53.49
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                            -190.28
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost             -293.55
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost              -292.36
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                    -109.36
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost   -175.71
## muUFdis_age_groupYounger                                                                                  -4.85
## muUFdis_genderMale                                                                                        -2.12
## muUFdis_countryUK                                                                                         -2.97
## muUFdis_competitionNOcompetition                                                                          -1.37
## muUFdis_relationUnknownMpeer                                                                              -2.15
## muUFdis_costNOcost                                                                                        -3.04
## muUFdis_age_groupYounger:genderMale                                                                       -1.80
## muUFdis_age_groupYounger:countryUK                                                                     -1030.29
## muUFdis_age_groupYounger:competitionNOcompetition                                                         -1.51
## muUFdis_age_groupYounger:relationUnknownMpeer                                                             -2.52
## muUFdis_age_groupYounger:costNOcost                                                                       -2.93
## muUFdis_genderMale:countryUK                                                                              -2.97
## muUFdis_genderMale:competitionNOcompetition                                                               -4.21
## muUFdis_genderMale:relationUnknownMpeer                                                                 -146.52
## muUFdis_genderMale:costNOcost                                                                             -5.49
## muUFdis_countryUK:competitionNOcompetition                                                                -4.03
## muUFdis_countryUK:relationUnknownMpeer                                                                    -1.90
## muUFdis_countryUK:costNOcost                                                                              -4.35
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                                     -3.38
## muUFdis_competitionNOcompetition:costNOcost                                                               -5.71
## muUFdis_relationUnknownMpeer:costNOcost                                                                 -293.07
## muUFdis_age_groupYounger:genderMale:countryUK                                                              2.69
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                              -6.14
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                                  -1.27
## muUFdis_age_groupYounger:genderMale:costNOcost                                                            -3.29
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                                1.14
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                                 -665.89
## muUFdis_age_groupYounger:countryUK:costNOcost                                                              6.95
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                    -5.27
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                              -3.36
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                                  13.98
## muUFdis_genderMale:countryUK:competitionNOcompetition                                                     -6.86
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                         -3.04
## muUFdis_genderMale:countryUK:costNOcost                                                                   -3.67
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                          -0.43
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                                    -3.43
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                        42.34
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                        -1255.56
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                                   -345.25
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                       -759.94
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                          14.75
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                 -1030.27
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                      -718.05
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                               -1037.03
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                       -143.34
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                   -6.54
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                     -298.45
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                         -73.61
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                  -859.66
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                      -287.36
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                       -293.68
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                             -2248.31
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                           3.26
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                            -241.00
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                            -1135.11
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                              -120.07
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer            -1069.71
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                       -230.59
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                           -820.20
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost             113.84
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            -2046.43
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                  -1060.22
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost -2217.69
##                                                                                                        u-95% CI
## muUFad_Intercept                                                                                           1.06
## muUFdis_Intercept                                                                                         -0.25
## muUFad_age_groupYounger                                                                                    1.13
## muUFad_genderMale                                                                                          1.48
## muUFad_countryUK                                                                                           2.61
## muUFad_competitionNOcompetition                                                                            0.68
## muUFad_relationUnknownMpeer                                                                                0.45
## muUFad_costNOcost                                                                                         -4.32
## muUFad_age_groupYounger:genderMale                                                                         2.02
## muUFad_age_groupYounger:countryUK                                                                          1.68
## muUFad_age_groupYounger:competitionNOcompetition                                                           1.66
## muUFad_age_groupYounger:relationUnknownMpeer                                                               1.98
## muUFad_age_groupYounger:costNOcost                                                                        56.12
## muUFad_genderMale:countryUK                                                                                2.18
## muUFad_genderMale:competitionNOcompetition                                                                 2.63
## muUFad_genderMale:relationUnknownMpeer                                                                     1.96
## muUFad_genderMale:costNOcost                                                                              55.49
## muUFad_countryUK:competitionNOcompetition                                                                  1.79
## muUFad_countryUK:relationUnknownMpeer                                                                      2.69
## muUFad_countryUK:costNOcost                                                                               54.05
## muUFad_competitionNOcompetition:relationUnknownMpeer                                                       2.23
## muUFad_competitionNOcompetition:costNOcost                                                                55.62
## muUFad_relationUnknownMpeer:costNOcost                                                                    17.35
## muUFad_age_groupYounger:genderMale:countryUK                                                               4.70
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                                4.92
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                                    4.09
## muUFad_age_groupYounger:genderMale:costNOcost                                                             -2.97
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                                 5.16
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                                     2.54
## muUFad_age_groupYounger:countryUK:costNOcost                                                              -1.57
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                      5.52
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                                0.76
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                                  209.84
## muUFad_genderMale:countryUK:competitionNOcompetition                                                       1.47
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                           0.48
## muUFad_genderMale:countryUK:costNOcost                                                                    -4.72
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                            2.55
## muUFad_genderMale:competitionNOcompetition:costNOcost                                                     -2.43
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                        280.68
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                             2.47
## muUFad_countryUK:competitionNOcompetition:costNOcost                                                      -2.23
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                         283.99
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                          245.09
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                      1.47
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                          2.56
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                                   56.14
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                           3.28
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                    56.07
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                       188.79
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                            3.35
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                     52.33
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                        186.90
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                         291.27
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                  4.26
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                           57.94
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                               22.01
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                               107.91
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                                106.75
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                 5.58
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                           3.82
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                             206.42
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost              175.84
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost               180.47
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                     239.92
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost    293.69
## muUFdis_age_groupYounger                                                                                   1.44
## muUFdis_genderMale                                                                                         3.30
## muUFdis_countryUK                                                                                          2.41
## muUFdis_competitionNOcompetition                                                                           2.92
## muUFdis_relationUnknownMpeer                                                                               2.31
## muUFdis_costNOcost                                                                                         1.83
## muUFdis_age_groupYounger:genderMale                                                                        6.36
## muUFdis_age_groupYounger:countryUK                                                                        -3.23
## muUFdis_age_groupYounger:competitionNOcompetition                                                          5.00
## muUFdis_age_groupYounger:relationUnknownMpeer                                                              4.93
## muUFdis_age_groupYounger:costNOcost                                                                        5.01
## muUFdis_genderMale:countryUK                                                                               4.14
## muUFdis_genderMale:competitionNOcompetition                                                                1.65
## muUFdis_genderMale:relationUnknownMpeer                                                                   -1.16
## muUFdis_genderMale:costNOcost                                                                              1.85
## muUFdis_countryUK:competitionNOcompetition                                                                 2.55
## muUFdis_countryUK:relationUnknownMpeer                                                                     4.48
## muUFdis_countryUK:costNOcost                                                                               3.28
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                                      2.32
## muUFdis_competitionNOcompetition:costNOcost                                                                1.67
## muUFdis_relationUnknownMpeer:costNOcost                                                                  -14.82
## muUFdis_age_groupYounger:genderMale:countryUK                                                           1030.05
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                               1.84
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                                 145.68
## muUFdis_age_groupYounger:genderMale:costNOcost                                                             5.85
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                             1029.89
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                                  701.76
## muUFdis_age_groupYounger:countryUK:costNOcost                                                           1035.50
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                     2.99
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                               5.92
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                                 293.34
## muUFdis_genderMale:countryUK:competitionNOcompetition                                                      3.03
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                        142.62
## muUFdis_genderMale:countryUK:costNOcost                                                                    6.67
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                         146.72
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                                     7.53
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                       298.80
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                           -8.08
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                                     -2.91
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                        229.23
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                         294.67
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                    -0.43
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                       537.39
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                                 -10.31
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                          5.46
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                    5.50
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                      -41.27
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                        1569.50
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                   231.30
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                       943.63
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                        -13.43
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                               695.64
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                         345.94
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                             700.96
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                             -115.21
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                              1845.38
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer             2219.95
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                        856.81
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                            326.39
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost            1131.98
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost             -126.77
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                   2260.34
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost  1327.54
##                                                                                                        Rhat
## muUFad_Intercept                                                                                       1.35
## muUFdis_Intercept                                                                                      1.37
## muUFad_age_groupYounger                                                                                1.53
## muUFad_genderMale                                                                                      1.22
## muUFad_countryUK                                                                                       1.41
## muUFad_competitionNOcompetition                                                                        1.44
## muUFad_relationUnknownMpeer                                                                            1.42
## muUFad_costNOcost                                                                                      2.20
## muUFad_age_groupYounger:genderMale                                                                     1.54
## muUFad_age_groupYounger:countryUK                                                                      1.49
## muUFad_age_groupYounger:competitionNOcompetition                                                       1.66
## muUFad_age_groupYounger:relationUnknownMpeer                                                           1.34
## muUFad_age_groupYounger:costNOcost                                                                     2.18
## muUFad_genderMale:countryUK                                                                            1.27
## muUFad_genderMale:competitionNOcompetition                                                             1.36
## muUFad_genderMale:relationUnknownMpeer                                                                 1.19
## muUFad_genderMale:costNOcost                                                                           2.17
## muUFad_countryUK:competitionNOcompetition                                                              1.45
## muUFad_countryUK:relationUnknownMpeer                                                                  1.22
## muUFad_countryUK:costNOcost                                                                            2.21
## muUFad_competitionNOcompetition:relationUnknownMpeer                                                   1.44
## muUFad_competitionNOcompetition:costNOcost                                                             2.19
## muUFad_relationUnknownMpeer:costNOcost                                                                 3.03
## muUFad_age_groupYounger:genderMale:countryUK                                                           1.44
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                            1.63
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                                1.27
## muUFad_age_groupYounger:genderMale:costNOcost                                                          2.13
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                             1.76
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                                 1.20
## muUFad_age_groupYounger:countryUK:costNOcost                                                           2.17
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                  1.53
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                            2.23
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                                2.20
## muUFad_genderMale:countryUK:competitionNOcompetition                                                   1.29
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                       1.13
## muUFad_genderMale:countryUK:costNOcost                                                                 2.12
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                        1.29
## muUFad_genderMale:competitionNOcompetition:costNOcost                                                  2.14
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                      3.03
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                         1.25
## muUFad_countryUK:competitionNOcompetition:costNOcost                                                   2.18
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                       3.03
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                        2.56
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                  1.63
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                      1.18
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                                2.05
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                       1.48
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                 2.14
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                     2.21
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                        1.47
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                  2.15
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                      2.21
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                       2.34
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                              1.12
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                        2.10
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                            3.03
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                             2.56
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                              2.57
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer             1.40
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                       2.03
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                           2.21
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost            2.36
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost             2.34
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                   2.56
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost  2.36
## muUFdis_age_groupYounger                                                                               1.17
## muUFdis_genderMale                                                                                     1.38
## muUFdis_countryUK                                                                                      1.31
## muUFdis_competitionNOcompetition                                                                       1.45
## muUFdis_relationUnknownMpeer                                                                           1.42
## muUFdis_costNOcost                                                                                     1.43
## muUFdis_age_groupYounger:genderMale                                                                    1.15
## muUFdis_age_groupYounger:countryUK                                                                     2.20
## muUFdis_age_groupYounger:competitionNOcompetition                                                      1.23
## muUFdis_age_groupYounger:relationUnknownMpeer                                                          1.16
## muUFdis_age_groupYounger:costNOcost                                                                    1.13
## muUFdis_genderMale:countryUK                                                                           1.34
## muUFdis_genderMale:competitionNOcompetition                                                            1.39
## muUFdis_genderMale:relationUnknownMpeer                                                                1.72
## muUFdis_genderMale:costNOcost                                                                          1.35
## muUFdis_countryUK:competitionNOcompetition                                                             1.27
## muUFdis_countryUK:relationUnknownMpeer                                                                 1.29
## muUFdis_countryUK:costNOcost                                                                           1.32
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                                  1.39
## muUFdis_competitionNOcompetition:costNOcost                                                            1.42
## muUFdis_relationUnknownMpeer:costNOcost                                                                1.50
## muUFdis_age_groupYounger:genderMale:countryUK                                                          2.20
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                           1.24
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                               1.71
## muUFdis_age_groupYounger:genderMale:costNOcost                                                         1.15
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                            2.20
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                                2.05
## muUFdis_age_groupYounger:countryUK:costNOcost                                                          2.20
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                 1.18
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                           1.23
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                               1.49
## muUFdis_genderMale:countryUK:competitionNOcompetition                                                  1.17
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                      1.70
## muUFdis_genderMale:countryUK:costNOcost                                                                1.35
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                       1.75
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                                 1.27
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                     1.59
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                        2.01
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                                  1.31
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                      2.02
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                       1.48
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                 2.20
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                     2.12
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                               2.20
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                      1.75
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                1.14
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                    1.59
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                       1.78
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                 2.38
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                     2.38
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                      1.48
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                             2.03
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                       1.31
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                           1.92
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                            1.47
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                             2.41
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer            1.66
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                      2.39
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                          2.40
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost           1.47
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost            2.49
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                  2.30
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost 1.96
##                                                                                                        Bulk_ESS
## muUFad_Intercept                                                                                             10
## muUFdis_Intercept                                                                                             9
## muUFad_age_groupYounger                                                                                       7
## muUFad_genderMale                                                                                            14
## muUFad_countryUK                                                                                              9
## muUFad_competitionNOcompetition                                                                               9
## muUFad_relationUnknownMpeer                                                                                   8
## muUFad_costNOcost                                                                                             5
## muUFad_age_groupYounger:genderMale                                                                            7
## muUFad_age_groupYounger:countryUK                                                                             8
## muUFad_age_groupYounger:competitionNOcompetition                                                              7
## muUFad_age_groupYounger:relationUnknownMpeer                                                                 10
## muUFad_age_groupYounger:costNOcost                                                                            5
## muUFad_genderMale:countryUK                                                                                  12
## muUFad_genderMale:competitionNOcompetition                                                                   10
## muUFad_genderMale:relationUnknownMpeer                                                                       16
## muUFad_genderMale:costNOcost                                                                                  5
## muUFad_countryUK:competitionNOcompetition                                                                     8
## muUFad_countryUK:relationUnknownMpeer                                                                        15
## muUFad_countryUK:costNOcost                                                                                   5
## muUFad_competitionNOcompetition:relationUnknownMpeer                                                          8
## muUFad_competitionNOcompetition:costNOcost                                                                    5
## muUFad_relationUnknownMpeer:costNOcost                                                                        5
## muUFad_age_groupYounger:genderMale:countryUK                                                                  8
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                                   7
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                                      12
## muUFad_age_groupYounger:genderMale:costNOcost                                                                 5
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                                    6
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                                       15
## muUFad_age_groupYounger:countryUK:costNOcost                                                                  5
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                         8
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                                   5
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                                       5
## muUFad_genderMale:countryUK:competitionNOcompetition                                                         11
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                             24
## muUFad_genderMale:countryUK:costNOcost                                                                        5
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                              11
## muUFad_genderMale:competitionNOcompetition:costNOcost                                                         5
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                             5
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                               12
## muUFad_countryUK:competitionNOcompetition:costNOcost                                                          5
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                              5
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                               5
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                         7
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                            17
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                                       5
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                              8
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                        5
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                            5
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                               8
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                         5
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                             5
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                              5
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                    23
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                               5
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                                   5
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                                    5
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                                     5
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                    9
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                              5
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                                  5
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                   5
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                    5
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                          5
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost         5
## muUFdis_age_groupYounger                                                                                     24
## muUFdis_genderMale                                                                                            9
## muUFdis_countryUK                                                                                            11
## muUFdis_competitionNOcompetition                                                                              8
## muUFdis_relationUnknownMpeer                                                                                  9
## muUFdis_costNOcost                                                                                            8
## muUFdis_age_groupYounger:genderMale                                                                          23
## muUFdis_age_groupYounger:countryUK                                                                            5
## muUFdis_age_groupYounger:competitionNOcompetition                                                            15
## muUFdis_age_groupYounger:relationUnknownMpeer                                                                24
## muUFdis_age_groupYounger:costNOcost                                                                          24
## muUFdis_genderMale:countryUK                                                                                 10
## muUFdis_genderMale:competitionNOcompetition                                                                   9
## muUFdis_genderMale:relationUnknownMpeer                                                                       6
## muUFdis_genderMale:costNOcost                                                                                 9
## muUFdis_countryUK:competitionNOcompetition                                                                   12
## muUFdis_countryUK:relationUnknownMpeer                                                                       11
## muUFdis_countryUK:costNOcost                                                                                 10
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                                         9
## muUFdis_competitionNOcompetition:costNOcost                                                                   8
## muUFdis_relationUnknownMpeer:costNOcost                                                                       8
## muUFdis_age_groupYounger:genderMale:countryUK                                                                 5
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                                 14
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                                      6
## muUFdis_age_groupYounger:genderMale:costNOcost                                                               18
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                                   5
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                                       5
## muUFdis_age_groupYounger:countryUK:costNOcost                                                                 5
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                       20
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                                 14
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                                      8
## muUFdis_genderMale:countryUK:competitionNOcompetition                                                        17
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                             6
## muUFdis_genderMale:countryUK:costNOcost                                                                       9
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                              6
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                                       12
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                            7
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                               5
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                                        10
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                             5
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                              8
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                        5
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                            6
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                                      5
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                             6
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                      22
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                           7
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                              6
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                        5
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                            5
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                             8
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                    5
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                             10
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                                  6
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                                   8
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                                    5
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                   7
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                             5
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                                 5
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                  8
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                   5
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                         5
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost        6
##                                                                                                        Tail_ESS
## muUFad_Intercept                                                                                             24
## muUFdis_Intercept                                                                                            30
## muUFad_age_groupYounger                                                                                      16
## muUFad_genderMale                                                                                            45
## muUFad_countryUK                                                                                             55
## muUFad_competitionNOcompetition                                                                              31
## muUFad_relationUnknownMpeer                                                                                  20
## muUFad_costNOcost                                                                                            24
## muUFad_age_groupYounger:genderMale                                                                           41
## muUFad_age_groupYounger:countryUK                                                                            37
## muUFad_age_groupYounger:competitionNOcompetition                                                             16
## muUFad_age_groupYounger:relationUnknownMpeer                                                                 32
## muUFad_age_groupYounger:costNOcost                                                                           23
## muUFad_genderMale:countryUK                                                                                  33
## muUFad_genderMale:competitionNOcompetition                                                                   56
## muUFad_genderMale:relationUnknownMpeer                                                                       26
## muUFad_genderMale:costNOcost                                                                                 24
## muUFad_countryUK:competitionNOcompetition                                                                    46
## muUFad_countryUK:relationUnknownMpeer                                                                        55
## muUFad_countryUK:costNOcost                                                                                  24
## muUFad_competitionNOcompetition:relationUnknownMpeer                                                         19
## muUFad_competitionNOcompetition:costNOcost                                                                   26
## muUFad_relationUnknownMpeer:costNOcost                                                                       19
## muUFad_age_groupYounger:genderMale:countryUK                                                                 18
## muUFad_age_groupYounger:genderMale:competitionNOcompetition                                                  18
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer                                                      26
## muUFad_age_groupYounger:genderMale:costNOcost                                                                23
## muUFad_age_groupYounger:countryUK:competitionNOcompetition                                                   18
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer                                                      106
## muUFad_age_groupYounger:countryUK:costNOcost                                                                 24
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                        13
## muUFad_age_groupYounger:competitionNOcompetition:costNOcost                                                  21
## muUFad_age_groupYounger:relationUnknownMpeer:costNOcost                                                      14
## muUFad_genderMale:countryUK:competitionNOcompetition                                                        157
## muUFad_genderMale:countryUK:relationUnknownMpeer                                                             59
## muUFad_genderMale:countryUK:costNOcost                                                                       22
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer                                              36
## muUFad_genderMale:competitionNOcompetition:costNOcost                                                        26
## muUFad_genderMale:relationUnknownMpeer:costNOcost                                                            18
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer                                               59
## muUFad_countryUK:competitionNOcompetition:costNOcost                                                         24
## muUFad_countryUK:relationUnknownMpeer:costNOcost                                                             18
## muUFad_competitionNOcompetition:relationUnknownMpeer:costNOcost                                              32
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                        16
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                            83
## muUFad_age_groupYounger:genderMale:countryUK:costNOcost                                                      23
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                             13
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                       20
## muUFad_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                           14
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                              13
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                        28
## muUFad_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                            13
## muUFad_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                             11
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                   100
## muUFad_genderMale:countryUK:competitionNOcompetition:costNOcost                                              23
## muUFad_genderMale:countryUK:relationUnknownMpeer:costNOcost                                                  19
## muUFad_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                                   32
## muUFad_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                                    33
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                   17
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                             28
## muUFad_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                                 14
## muUFad_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                  12
## muUFad_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                   12
## muUFad_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                         34
## muUFad_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost        12
## muUFdis_age_groupYounger                                                                                     29
## muUFdis_genderMale                                                                                           31
## muUFdis_countryUK                                                                                            53
## muUFdis_competitionNOcompetition                                                                             33
## muUFdis_relationUnknownMpeer                                                                                 33
## muUFdis_costNOcost                                                                                           61
## muUFdis_age_groupYounger:genderMale                                                                          24
## muUFdis_age_groupYounger:countryUK                                                                           11
## muUFdis_age_groupYounger:competitionNOcompetition                                                            61
## muUFdis_age_groupYounger:relationUnknownMpeer                                                                50
## muUFdis_age_groupYounger:costNOcost                                                                          58
## muUFdis_genderMale:countryUK                                                                                 28
## muUFdis_genderMale:competitionNOcompetition                                                                  72
## muUFdis_genderMale:relationUnknownMpeer                                                                      15
## muUFdis_genderMale:costNOcost                                                                                50
## muUFdis_countryUK:competitionNOcompetition                                                                   55
## muUFdis_countryUK:relationUnknownMpeer                                                                       20
## muUFdis_countryUK:costNOcost                                                                                 46
## muUFdis_competitionNOcompetition:relationUnknownMpeer                                                        35
## muUFdis_competitionNOcompetition:costNOcost                                                                  47
## muUFdis_relationUnknownMpeer:costNOcost                                                                      16
## muUFdis_age_groupYounger:genderMale:countryUK                                                                11
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition                                                 41
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer                                                     16
## muUFdis_age_groupYounger:genderMale:costNOcost                                                               60
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition                                                  11
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer                                                      13
## muUFdis_age_groupYounger:countryUK:costNOcost                                                                11
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer                                       67
## muUFdis_age_groupYounger:competitionNOcompetition:costNOcost                                                 51
## muUFdis_age_groupYounger:relationUnknownMpeer:costNOcost                                                     17
## muUFdis_genderMale:countryUK:competitionNOcompetition                                                        82
## muUFdis_genderMale:countryUK:relationUnknownMpeer                                                            15
## muUFdis_genderMale:countryUK:costNOcost                                                                     130
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer                                             15
## muUFdis_genderMale:competitionNOcompetition:costNOcost                                                       43
## muUFdis_genderMale:relationUnknownMpeer:costNOcost                                                           37
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer                                              30
## muUFdis_countryUK:competitionNOcompetition:costNOcost                                                        31
## muUFdis_countryUK:relationUnknownMpeer:costNOcost                                                            16
## muUFdis_competitionNOcompetition:relationUnknownMpeer:costNOcost                                             18
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition                                       11
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer                                           13
## muUFdis_age_groupYounger:genderMale:countryUK:costNOcost                                                     11
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer                            15
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:costNOcost                                      36
## muUFdis_age_groupYounger:genderMale:relationUnknownMpeer:costNOcost                                          35
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer                             25
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:costNOcost                                       13
## muUFdis_age_groupYounger:countryUK:relationUnknownMpeer:costNOcost                                           16
## muUFdis_age_groupYounger:competitionNOcompetition:relationUnknownMpeer:costNOcost                            17
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                                   11
## muUFdis_genderMale:countryUK:competitionNOcompetition:costNOcost                                             32
## muUFdis_genderMale:countryUK:relationUnknownMpeer:costNOcost                                                 20
## muUFdis_genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                                  24
## muUFdis_countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                                   22
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer                  11
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:costNOcost                            12
## muUFdis_age_groupYounger:genderMale:countryUK:relationUnknownMpeer:costNOcost                                15
## muUFdis_age_groupYounger:genderMale:competitionNOcompetition:relationUnknownMpeer:costNOcost                 24
## muUFdis_age_groupYounger:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                  13
## muUFdis_genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost                        11
## muUFdis_age_groupYounger:genderMale:countryUK:competitionNOcompetition:relationUnknownMpeer:costNOcost       11
## 
## Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
## and Tail_ESS are effective sample size measures, and Rhat is the potential
## scale reduction factor on split chains (at convergence, Rhat = 1).
```


#### 3.3 Model Selection  
We then selected the best fitting model by using *Leave-One-Out Cross-Validation (LOO)*. THis approach compares models via approximate leave-one-out cross-validation. Since higher LOOIC values indicate better fit, we see that the model accounting for overdispersion fits substantially better. 


```r
##LOO vaule for model 2.1
loo(brm2.1)
```

```
## 
## Computed from 4000 by 1632 log-likelihood matrix
## 
##          Estimate   SE
## elpd_loo  -1226.5 29.5
## p_loo       180.9  5.6
## looic      2453.1 59.0
## ------
## Monte Carlo SE of elpd_loo is 0.2.
## 
## Pareto k diagnostic values:
##                          Count Pct.    Min. n_eff
## (-Inf, 0.5]   (good)     1629  99.8%   1537      
##  (0.5, 0.7]   (ok)          3   0.2%   2599      
##    (0.7, 1]   (bad)         0   0.0%   <NA>      
##    (1, Inf)   (very bad)    0   0.0%   <NA>      
## 
## All Pareto k estimates are ok (k < 0.7).
## See help('pareto-k-diagnostic') for details.
```



```r
##LOO vaule for model 2.2
loo(brm2.2)
```

```
## 
## Computed from 4000 by 1632 log-likelihood matrix
## 
##          Estimate   SE
## elpd_loo  -1228.5 30.8
## p_loo       218.5  7.1
## looic      2457.1 61.5
## ------
## Monte Carlo SE of elpd_loo is 0.3.
## 
## Pareto k diagnostic values:
##                          Count Pct.    Min. n_eff
## (-Inf, 0.5]   (good)     1623  99.4%   877       
##  (0.5, 0.7]   (ok)          9   0.6%   1611      
##    (0.7, 1]   (bad)         0   0.0%   <NA>      
##    (1, Inf)   (very bad)    0   0.0%   <NA>      
## 
## All Pareto k estimates are ok (k < 0.7).
## See help('pareto-k-diagnostic') for details.
```



```r
##LOO vaule for model 2.3
loo(brm2.3)
```

```
## 
## Computed from 4000 by 1632 log-likelihood matrix
## 
##          Estimate   SE
## elpd_loo  -1253.7 33.0
## p_loo       275.7  9.6
## looic      2507.4 65.9
## ------
## Monte Carlo SE of elpd_loo is 0.4.
## 
## Pareto k diagnostic values:
##                          Count Pct.    Min. n_eff
## (-Inf, 0.5]   (good)     1611  98.7%   296       
##  (0.5, 0.7]   (ok)         21   1.3%   411       
##    (0.7, 1]   (bad)         0   0.0%   <NA>      
##    (1, Inf)   (very bad)    0   0.0%   <NA>      
## 
## All Pareto k estimates are ok (k < 0.7).
## See help('pareto-k-diagnostic') for details.
```



```r
##LOO vaule for model 2.4
loo(brm2.4)
```

```
## Warning: Found 10 observations with a pareto_k > 0.7 in model 'brm2.4'. It is
## recommended to set 'moment_match = TRUE' in order to perform moment matching for
## problematic observations.
```

```
## 
## Computed from 4000 by 1632 log-likelihood matrix
## 
##          Estimate   SE
## elpd_loo  -1275.6 36.5
## p_loo       334.3 14.5
## looic      2551.2 73.0
## ------
## Monte Carlo SE of elpd_loo is NA.
## 
## Pareto k diagnostic values:
##                          Count Pct.    Min. n_eff
## (-Inf, 0.5]   (good)     1566  96.0%   506       
##  (0.5, 0.7]   (ok)         56   3.4%   177       
##    (0.7, 1]   (bad)         7   0.4%   41        
##    (1, Inf)   (very bad)    3   0.2%   11        
## See help('pareto-k-diagnostic') for details.
```


```r
##LOO vaule for model 2.5
loo(brm2.5)
```

```
## Warning: Found 42 observations with a pareto_k > 0.7 in model 'brm2.5'. It is
## recommended to set 'moment_match = TRUE' in order to perform moment matching for
## problematic observations.
```

```
## 
## Computed from 4000 by 1632 log-likelihood matrix
## 
##          Estimate   SE
## elpd_loo  -1273.5 36.1
## p_loo       338.5 14.5
## looic      2547.0 72.2
## ------
## Monte Carlo SE of elpd_loo is NA.
## 
## Pareto k diagnostic values:
##                          Count Pct.    Min. n_eff
## (-Inf, 0.5]   (good)     1550  95.0%   9         
##  (0.5, 0.7]   (ok)         40   2.5%   10        
##    (0.7, 1]   (bad)         9   0.6%   4         
##    (1, Inf)   (very bad)   33   2.0%   1         
## See help('pareto-k-diagnostic') for details.
```


```r
##LOO vaule for model 2.6
loo(brm2.6)
```

```
## Warning: Found 51 observations with a pareto_k > 0.7 in model 'brm2.6'. It is
## recommended to set 'moment_match = TRUE' in order to perform moment matching for
## problematic observations.
```

```
## 
## Computed from 4000 by 1632 log-likelihood matrix
## 
##          Estimate   SE
## elpd_loo  -1268.1 35.9
## p_loo       339.2 14.2
## looic      2536.2 71.7
## ------
## Monte Carlo SE of elpd_loo is NA.
## 
## Pareto k diagnostic values:
##                          Count Pct.    Min. n_eff
## (-Inf, 0.5]   (good)     1540  94.4%   2         
##  (0.5, 0.7]   (ok)         41   2.5%   2         
##    (0.7, 1]   (bad)        18   1.1%   2         
##    (1, Inf)   (very bad)   33   2.0%   1         
## See help('pareto-k-diagnostic') for details.
```

The Bayesian criteria for convergence used ‘Rhat’ information. Rhat is the potential scale reduction factor on split chains and if a value is greater than 1, the model has not sufficiently converged [(Bürkner, 2017)](https://arxiv.org/abs/1705.11123). Models 5 and 6 were excluded from the comparison because of their poor convergence (as indicated by their 'Rhat' vaules). 

For Model 4 (with 6 main effects and their 2,3, and 4-way interactions), the Rhat equalled 1, indicating good convergence. Leave-one-out (LOO) cross validation was used for model selection and identified Model 4 as the best fitting model, having the greater LOO value [(Vehtari, Gelman & Gabry, 2017)](https://link.springer.com/article/10.1007/s11222-016-9696-4). 

## 4 Output Visualization 

Based on the output of Model 4, we plotted the estimated possibility(EP) for effects that are significant. In each plot, the red colour dot (fair) represents the estimated mean for probability of fair allocation (with associated credible intervals), the green means and intervals (ufad) show the proportions of allocations which advantage the self, the details in blue (ufdis) represents allocations in which the child disadvantages her or himself.

#### 4.1	Two-way interactions led by cost  
![](Registration_The-Dynamic-Cost-Model_files/figure-latex/plot 4.1-1.pdf)<!-- --> 

#### 4.2	interactions led by cost*competition  


![](Registration_The-Dynamic-Cost-Model_files/figure-latex/plot 4.2-1.pdf)<!-- --> 

#### 4.3	interactions led by cost*relation  

![](Registration_The-Dynamic-Cost-Model_files/figure-latex/plot 4.3-1.pdf)<!-- --> 

#### 4.4	interactions led by cost*contextural factors   

![](Registration_The-Dynamic-Cost-Model_files/figure-latex/plot 4.4-1.pdf)<!-- --> 

#### 4.5 interactions led by cost*structural factors 

![](Registration_The-Dynamic-Cost-Model_files/figure-latex/plot 4.5-1.pdf)<!-- --> 

