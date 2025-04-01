# Activity 12: key

On Thursday we will use historical data to predict NCAA basketball
games. For this activity we will generate (simulate) synthetic data.

### 1.

Simulate synthetic data that represents historical NCAA games. In
particular, let’s consider the model

$$pointdiff = \beta_0 + \beta_1 x_{seeddiff} + \epsilon; \epsilon \sim N(0, \sigma^2)$$

where:

- $\beta_0$ = 0
- $\beta_1$ = 2
- $\sigma = 12$

You can assume that $x_{seeddiff}$ is a non-negative integer valued
variable between 0 and 15. Ten years of data would be approximately ~
635 games.

``` r
set.seed(04032023)
num_games <- 960 # approximately 15 years
seeddiff <- rep(0:15, each = 60)


beta1 <- 2
sigma <- 12

pointdiff <- rnorm(num_games, mean = beta1 * seeddiff, sd = sigma)

data_out <- tibble(seeddiff = seeddiff,
                   pointdiff = pointdiff) 
```

### 2.

Interpret the three coefficients in the model specified in part 1.

- $\beta_0$ = expected point differential for teams with the same seed
- $\beta_1$ = multiplicative coefficient for point differential as a
  function of seed difference
- $\sigma = 12$ = standard deviation in point differential. Most games
  would be $\pm$ 2 standard deviations (or 24 points) from the mean.

### 3.

Create a visualization of your point differential versus seed
differential.

Bonus points: also checkout the [ggridges
package](https://cran.r-project.org/web/packages/ggridges/vignettes/introduction.html)

``` r
library(ggridges)

data_out %>% ggplot(aes(y = pointdiff, x = seeddiff), method = 'loess', formula = 'y ~ x') + geom_point() + geom_smooth() + theme_bw() 
```

    `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

![](Activity12_key_files/figure-commonmark/unnamed-chunk-2-1.png)

``` r
ggplot(data_out, aes(y = factor(seeddiff), x = pointdiff)) +
  geom_density_ridges(quantile_lines = T, jittered_points = TRUE) + 
  coord_flip() +
  theme_bw() 
```

    Picking joint bandwidth of 4.44

![](Activity12_key_files/figure-commonmark/unnamed-chunk-2-2.png)

### 4.

Specify prior distributions on your parameters. For this exercise, avoid
a uniform prior on sigma and consider an inverse gamma prior.

- $\beta_0 \sim$ Normal(0,$.001^2$)
- $\beta_1 \sim$ Normal(1, $3^2$)
- $\sigma \sim$ InvGamma(2, 20)

``` r
library(invgamma)
x <- seq(0, 100, by = .1)

tibble(x = x) |>
  mutate(y = dinvgamma(x, 2, 20 )) |>
  ggplot(aes(y = y, x= x)) +
  geom_line() +
  theme_bw()
```

    Warning: Removed 1 row containing missing values or values outside the scale range
    (`geom_line()`).

![](Activity12_key_files/figure-commonmark/unnamed-chunk-3-1.png)

### 5.

Write JAGS code to fit this model. Output the results.

``` r
#Prior parameters
M0 <- 0
S0 <- .001
M1 <- 1
S1 <- 3
a <- 2
b <- 20

# Store data
dataList = list(y = data_out$pointdiff, 
                x = data_out$seeddiff,
                N = nrow(data_out), 
                M0 = M0, S0 = S0,
                M1 = M1, S1 = S1,
                a = a, b = b)

# Model String
modelString = "model {
  for ( i in 1:N ) {
    y[i] ~ dnorm(beta0 + beta1 * x[i], 1/sigma^2) # sampling model
  }
  beta0 ~ dnorm(M0,1/S0^2)
  beta1 ~ dnorm(M1, 1 / S1^2)
  phi ~ dgamma(a,b)
  sigma <- 1/phi
} "
writeLines( modelString, con='NORMmodel.txt')


# Runs JAGS Model
jags.norm <- jags.model( file = "NORMmodel.txt", data = dataList,
                         n.chains = 2, n.adapt = 1000)
```

    Compiling model graph
       Resolving undeclared variables
       Allocating nodes
    Graph information:
       Observed stochastic nodes: 960
       Unobserved stochastic nodes: 3
       Total graph size: 1971

    Initializing model

``` r
update(jags.norm, n.iter = 1000)

coda.norm <- coda.samples( jags.norm, variable.names = c('beta0','beta1', 'sigma'), n.iter = 5000)

summary(coda.norm)
```


    Iterations = 2001:7000
    Thinning interval = 1 
    Number of chains = 2 
    Sample size per chain = 5000 

    1. Empirical mean and standard deviation for each variable,
       plus standard error of the mean:

                Mean        SD  Naive SE Time-series SE
    beta0 -4.690e-06 0.0009932 9.932e-06      9.992e-06
    beta1  2.067e+00 0.0429811 4.298e-04      4.221e-04
    sigma  1.183e+01 0.2716487 2.716e-03      3.526e-03

    2. Quantiles for each variable:

               2.5%        25%        50%       75%     97.5%
    beta0 -0.001969 -0.0006721 -2.033e-05 6.566e-04  0.001961
    beta1  1.982192  2.0381419  2.067e+00 2.096e+00  2.151223
    sigma 11.309635 11.6468022  1.183e+01 1.201e+01 12.384740

### 6.

Compare these results to those from a classical regression model using
`lm()`.

``` r
summary(lm(pointdiff~seeddiff, data = data_out))
```


    Call:
    lm(formula = pointdiff ~ seeddiff, data = data_out)

    Residuals:
        Min      1Q  Median      3Q     Max 
    -35.584  -7.793  -0.119   8.015  35.603 

    Coefficients:
                Estimate Std. Error t value Pr(>|t|)    
    (Intercept)  0.72954    0.72915   1.001    0.317    
    seeddiff     1.99638    0.08283  24.103   <2e-16 ***
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Residual standard error: 11.83 on 958 degrees of freedom
    Multiple R-squared:  0.3775,    Adjusted R-squared:  0.3769 
    F-statistic:   581 on 1 and 958 DF,  p-value: < 2.2e-16

``` r
summary(lm(pointdiff~ -1 + seeddiff, data = data_out))
```


    Call:
    lm(formula = pointdiff ~ -1 + seeddiff, data = data_out)

    Residuals:
        Min      1Q  Median      3Q     Max 
    -35.702  -7.634   0.124   8.241  35.415 

    Coefficients:
             Estimate Std. Error t value Pr(>|t|)    
    seeddiff  2.06698    0.04337   47.66   <2e-16 ***
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Residual standard error: 11.83 on 959 degrees of freedom
    Multiple R-squared:  0.7031,    Adjusted R-squared:  0.7028 
    F-statistic:  2271 on 1 and 959 DF,  p-value: < 2.2e-16

### 7.

Summarize your findings. Write a paragraph so that a college basketball
coach could understand.

\*\*For each unit difference in seed the expected point differential
would increase by about 2 points. For teams of the same seed, the
expected point differential would be 0 points. As you know with the NCAA
tournament there is the potential for upsets. This is illustrated by the
$\sigma$ term in our model. For a given point spread, it would not be
unlikely to see differences of about 20 points, either way, from the
expected point spread.

### 8.

Now we will construct a posterior predictive distribution. This will
enable us answers questions like “What is the probability that a one
seed is upset by a 16 seed (15 point difference in seeds.) In
particular, construct a posterior predictive distribution, conditional
on the following scenarios:

- Seed difference = 15
- Seed difference = 7 (most commonly 1 vs. 8)
- Seed difference = 1

Use those distributions to compute the probability of an upset occuring.

``` r
# Store data
dataList = list(y = data_out$pointdiff, 
                x = data_out$seeddiff,
                N = nrow(data_out), 
                M0 = M0, S0 = S0,
                M1 = M1, S1 = S1,
                a = a, b = b)

# Model String
modelString_pp = "model {
  for ( i in 1:N ) {
    y[i] ~ dnorm(beta0 + beta1 * x[i], 1/sigma^2) # sampling model
  }
 
  beta0 ~ dnorm(M0,1/S0^2)
  beta1 ~ dnorm(M1, 1 / S1^2)
  phi ~ dgamma(a,b)
  sigma <- 1/phi
  
  pp1 ~ dnorm(beta0 + beta1, 1/sigma^2)
  pp7 ~ dnorm(beta0 + beta1 * 7, 1/sigma^2)
  pp15 ~ dnorm(beta0 + beta1 * 15, 1/sigma^2)
} "
writeLines( modelString_pp, con='NORMmodel.txt')


# Runs JAGS Model
jags.norm <- jags.model( file = "NORMmodel.txt", data = dataList,
                         n.chains = 2, n.adapt = 1000)
```

    Compiling model graph
       Resolving undeclared variables
       Allocating nodes
    Graph information:
       Observed stochastic nodes: 960
       Unobserved stochastic nodes: 6
       Total graph size: 1977

    Initializing model

``` r
update(jags.norm, n.iter = 1000)

coda.norm <- coda.samples( jags.norm, variable.names = c('beta0','beta1', 'sigma','pp1','pp7','pp15'), n.iter = 5000)

pp_out <- tibble(vals = c(coda.norm[[1]][,'pp1'],
                          coda.norm[[1]][,'pp7'],
                          coda.norm[[1]][,'pp15']),
                 type = rep(c('seed diff 1', 
                              'seed diff 7', 
                              'seed diff 15'),
                            each = 5000))

pp_out %>% group_by(type) %>% 
  summarize(`upset prob` = mean(vals < 0)) %>%
  kable(digits = 3)
```

| type         | upset prob |
|:-------------|-----------:|
| seed diff 1  |      0.424 |
| seed diff 15 |      0.003 |
| seed diff 7  |      0.111 |

``` r
pp_out %>% ggplot(aes(x = vals)) +
  geom_histogram() + facet_grid(type~.)
```

    `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Activity12_key_files/figure-commonmark/unnamed-chunk-7-1.png)
