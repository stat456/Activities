# Activity 14

### Last-Week’s Recap

- GLMs
- Posterior predictive distributions

### This week

- Hierarchical Models

### Next week

- Project Analysis (for next Tuesday - no videos)
- Final Exam Review (Tuesday)
- In class exam (Thursday)
- Take Home Exam

------------------------------------------------------------------------

Similar to the Space Jam themed exercises previously For this activity
we will use NBA free throw shooting data to compare a few different
approaches for analyzing this dataset.

| player  | position | FTA | FTM |
|:--------|:---------|----:|----:|
| Beal    | G        |  61 |  50 |
| Curry   | G        | 114 | 103 |
| Harden  | G        | 115 | 101 |
| Irving  | G        |  84 |  76 |
| Leonard | G        | 102 |  95 |
| Oladipo | G        |   6 |   6 |
| Parker  | G        |  14 |  14 |
| Paul    | G        |  33 |  29 |
| Wall    | G        |  93 |  78 |

------------------------------------------------------------------------

1.  Fit a frequentist model to estimate free throw shooting percentage,
    $\theta,$ for the players Curry, Beal, and Oladipo. Write the
    sampling model you’ve assumed with your code (hint: `glm()` is one
    possible route). Report a point estimate and uncertainty intervals
    for each player.

------------------------------------------------------------------------

2.  Specify independent models using a uniform prior for $\theta$ for
    the players Curry, Beal, and Oladipo. Write the the sampling model
    and priors for these models. Recall you can find the posterior here
    analytically without using MCMC. Find posterior means and HDI
    intervals for this approach. Recall the HPDinterval function can be
    applied directly to samples from a beta distribution as
    `HPDinterval(mcmc(data  = rbeta(n = 5000, shape1 = a.star, shape2 = b.star)))`

------------------------------------------------------------------------

3.  Specify independent models using an informative prior, of your
    choice, for $\theta$ for the players Curry, Beal, and Oladipo. Write
    the the sampling model and priors for these models. Recall you can
    find the posterior here analytically without using MCMC. Defend your
    prior choice.

------------------------------------------------------------------------

4.  Reflect on the differences/similarities in the credible intervals
    between the two different priors as well as the frequentist model.
    If you were going to bet on the players shooting percentages for the
    next season, which would you prefer?

------------------------------------------------------------------------

5.  Now consider a hierarchical model where

$$FTM_i \sim Binomial(FTA_i, \theta_i)$$

$$\theta_i \sim  Beta(\omega*(\kappa-2)+1, (1-\omega)*(\kappa-2)+1)$$
$$\omega \sim Beta(2.55, .45) $$ $$\kappa-2 \sim Gamma(.01, .01)$$

Note that this specification of the Beta distribution uses a mean
parameter $\omega$ and a dispersion parameter in $\kappa$.

Use the JAGS code to again estimate free throw shooting percentage and
uncertainty intervals for the three players.

``` r
# Model - see page 250 in text
modelString <- 'model {
  for ( s in 1:Nsubj ) {
    z[s] ~ dbin( theta[s], N[s] )
    theta[s] ~ dbeta( omega*(kappa-2)+1, (1-omega)*(kappa-2)+1)
  }
  omega ~ dbeta(2.55, .45) # 85%
  kappa <- kappaMinusTwo + 2
  kappaMinusTwo ~ dgamma(.01, .01)
}'
writeLines( modelString, con='HierModel.txt')
```

Also summarize your estimates for $\omega$ and $\kappa$

------------------------------------------------------------------------

6.  Finally, discuss the differences in all your estimation approaches.
    If you had to choose one analysis: HM, Bayes with uniform priors,
    Bayes with informative priors, or the frequentist approach which
    would you choose and why?
