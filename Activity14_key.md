# Activity 14: key

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

*$$FTM_i \sim Binomial(FTA_i, \theta_i)$$ where $i$ denotes the $i^{th}$
player and $\theta_i$ is the free throw shooting percentage for player
i.*

``` r
beal <- glm(cbind(50,11)~ 1,  family = "binomial")
curry <- glm(cbind(103,11)~ 1,  family = "binomial")
oladipo <- glm(cbind(6,0)~ 1,  family = "binomial")
```

- Beal (50 / 61 = 0.82) MLE is 0.82 CI is (0.71, 0.9)

- Curry (103/ 114 = 0.9) MLE is 0.9 CI is (0.84, 0.95)

- Oladipo (6/ 6 = 1) MLE is 1 CI is (0, NA) - Note other methods will
  result in a confidence interval of (1,1).

*The Oladipo case is obviously troubling here, but in general the
philosophy of using prior information or sharing information from
hierarchical models seems reasonable in this setting.*

------------------------------------------------------------------------

2.  Specify independent models using a uniform prior for $\theta$ for
    the players Curry, Beal, and Oladipo. Write the the sampling model
    and priors for these models. Recall you can find the posterior here
    analytically without using MCMC. Find posterior means and HDI
    intervals for this approach. Recall the HPDinterval function can be
    applied directly to samples from a beta distribution as
    `HPDinterval(mcmc(data  = rbeta(n = 5000, shape1 = a.star, shape2 = b.star)))`

$$FTM_i \sim Binomial(FTA_i, \theta_i)$$ where $i$ denotes the $i^{th}$
player and $\theta_i$ is the free throw shooting percentage for player
i. $\theta_i \sim Beta(1,1)$ for all i.

- Beal (50 / 61 = 0.82) Posterior mean is 0.81 HPD is (0.71, 0.9 )

- Curry (103/ 114 = 0.9) Posterior mean is 0.9 HPD is (0.84, 0.95 )

- Oladipo (6/ 6 = 1) Posterior mean is 0.88 HPD is (0.65, 1 )

------------------------------------------------------------------------

3.  Specify independent models using an informative prior, of your
    choice, for $\theta$ for the players Curry, Beal, and Oladipo. Write
    the the sampling model and priors for these models. Recall you can
    find the posterior here analytically without using MCMC. Defend your
    prior choice.

$$FTM_i \sim Binomial(FTA_i, \theta_i)$$ where $i$ denotes the $i^{th}$
player and $\theta_i$ is the free throw shooting percentage for player
i.

$\theta_i \sim Beta(25.5,4.5)$ for all i. This corresponds to roughly
85% free throw shooting and is weighted equivalently to taking 30 shots
(and making 25.5).

- Beal (50 / 61 = 0.82) Posterior mean is 0.83 HPD is (0.75, 0.9)

- Curry (103/ 114 = 0.9) Posterior mean is 0.89 HPD is (0.84, 0.94 )

- Oladipo (6/ 6 = 1) Posterior mean is 0.88 HPD is (0.77, 0.97)

------------------------------------------------------------------------

4.  Reflect on the differences/similarities in the credible intervals
    between the two different priors as well as the frequentist model.
    If you were going to bet on the players shooting percentages for the
    next season, which would you prefer?

**The interval for Oladipo is obviously problematic in the frequentist
case. Interestingly the typical solution can be thought of as a Bayesian
solution with a uniform prior. The differences are not as large as I’d
expect, particularly for Oladipo. Some of that is that the uniform prior
effectively shrinks toward .5 and the point estimate ends up being
$\frac{6+1}{6 + 1 +1} = \frac{7}{8}$, which is about .88 and quite close
to my prior mean. I do think I prefer the informative priors in this
case.**

------------------------------------------------------------------------

5.  Now consider a hierarchical model where

$\theta_i \sim Beta(\omega*(\kappa-2)+1, (1-\omega)*(\kappa-2)+1)$
$\omega \sim Beta(2.55, .45)$ $\kappa-2 \sim Gamma(.01, .01)$

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

``` r
# Data
guards <- playoff.free.throws %>% filter(position == 'G')
Nsubj <- nrow(guards)
dataList = list(z=guards$FTM, N=guards$FTA,Nsubj = Nsubj)

# RUN JAGS
jags.hier <- jags.model( file = "HierModel.txt", 
                         data = dataList, 
                         n.chains = 3, 
                         n.adapt = 50000)
```

    Compiling model graph
       Resolving undeclared variables
       Allocating nodes
    Graph information:
       Observed stochastic nodes: 9
       Unobserved stochastic nodes: 11
       Total graph size: 42

    Initializing model

``` r
update(jags.hier, 10000)

num.mcmc <- 10000
codaSamples <- coda.samples( jags.hier, 
                             variable.names = c('omega', 'kappa','theta'), 
                             n.iter = num.mcmc)
```

|            | player  | position | FTA | FTM | FT.PCT | post.mean | lower | upper |
|:-----------|:--------|:---------|----:|----:|-------:|----------:|------:|------:|
| theta\[1\] | Beal    | G        |  61 |  50 |   0.82 |      0.85 |  0.77 |  0.92 |
| theta\[2\] | Curry   | G        | 114 | 103 |   0.90 |      0.90 |  0.85 |  0.94 |
| theta\[3\] | Harden  | G        | 115 | 101 |   0.88 |      0.88 |  0.83 |  0.93 |
| theta\[4\] | Irving  | G        |  84 |  76 |   0.90 |      0.90 |  0.85 |  0.95 |
| theta\[5\] | Leonard | G        | 102 |  95 |   0.93 |      0.92 |  0.87 |  0.96 |
| theta\[6\] | Oladipo | G        |   6 |   6 |   1.00 |      0.90 |  0.82 |  0.99 |
| theta\[7\] | Parker  | G        |  14 |  14 |   1.00 |      0.91 |  0.84 |  1.00 |
| theta\[8\] | Paul    | G        |  33 |  29 |   0.88 |      0.88 |  0.81 |  0.95 |
| theta\[9\] | Wall    | G        |  93 |  78 |   0.84 |      0.86 |  0.79 |  0.91 |

Also summarize your estimates for $\omega$ and $\kappa$

The mean and HPD values for omega and kappa are

``` r
HPDinterval(combine.mcmc(codaSamples))[1:2,]
```

              lower       upper
    kappa 3.6261259 204.7529010
    omega 0.8543268   0.9866612

``` r
colMeans((combine.mcmc(codaSamples))[,c(1:2)])
```

        kappa     omega 
    72.386086  0.909626 

------------------------------------------------------------------------

6.  Finally, discuss the differences in all your estimation approaches.
    If you had to choose one analysis: HM, Bayes with uniform priors,
    Bayes with informative priors, or the frequentist approach which
    would you choose and why?
