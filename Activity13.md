# Activity 13

Now reconsider the willow tit dataset and consider modeling not just the
presence / absence of birds, but directly modeling the number of birds
observed in each spatial region.

``` r
birds <- read.csv('http://math.montana.edu/ahoegh/teaching/stat491/data/willowtit2013_count.csv')
head(birds) |> 
  kable()
```

| siteID | elev | rlength | forest | bird.count | searchDuration |
|:-------|-----:|--------:|-------:|-----------:|---------------:|
| Q001   |  450 |     6.4 |      3 |          0 |            160 |
| Q002   |  450 |     5.5 |     21 |          0 |            190 |
| Q003   | 1050 |     4.3 |     32 |          3 |            150 |
| Q004   |  950 |     4.5 |      9 |          0 |            180 |
| Q005   | 1150 |     5.4 |     35 |          0 |            200 |
| Q006   |  550 |     3.6 |      2 |          0 |            115 |

This dataset contains 242 sites and 6 variables:

    - siteID, a unique identifier for the site, some were not sampled during this period 
    - elev, mean elevation of the quadrant in meters 
    - rlength, the length of the route walked by the birdwatcher, in kilometers 
    - forest, percent forest cover 
    - bird.count, number of birds identified 
    - searchDuration, time birdwatcher spent searching the site, in minutes

#### 1. Data Visualization

Create two figures that explore `bird.count` as a function of forest
cover percentage (`forest`) and elevation (`elev`).

#### 2. Model Specification

Using a Poisson regression model, clearly write out the model to
understand how forest cover and elevation impact bird count.

#### 3. Priors

Describe and justify the necessary priors for this model.

#### 4. Fit MCMC

Fit the JAGS code for this model. You will have to put this together
following the specification in the previous examples, but the following
statement can be used for the sampling model portion.

``` r
model.string <- 'model {
  for (i in 1:Ntotal) {
    y[i] ~ dpois(mu[i])
    mu[i] <- exp(beta0 + sum( beta[1:Nx] * x[i,1:Nx] ))
  }
  # priors 
  beta0 ~ dnorm(0,1/5^2)
  for (j in 1:Nx){
    beta[j] ~ dnorm(0, 1/100^2)
  }
}'
```

#### 5. Use posterior predictive distribution for model checking

- First extract posterior samples.

- Conditional on a single ‘X’ value, the 113th row in the dataset

- Conditional on all data

#### 6. Summarize inferences from model

Talk about the model and discuss which and how predictor variables
influence the observed bird count.
