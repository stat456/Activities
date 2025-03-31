# Activity 12


### Last Week Overview: t-tests & t-distributions

- t-distributions $\rightarrow$ heavier tails

- t-tests are a simple linear model

- posterior predictive distributions (more this week)

------------------------------------------------------------------------

### This Week Overview: regression (as GLMs)

- t-tests, ANOVA, regression, ANCOVA, … $\rightarrow$ all GLMs

- posterior predictive distributions

------------------------------------------------------------------------

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

### 2.

Interpret the three coefficients in the model specified in part 1.

- $\beta_0$ =
- $\beta_1$ =
- $\sigma = 12$ =

### 3.

Create a visualization of your point differential versus seed
differential.

Bonus points: also checkout the [ggridges
package](https://cran.r-project.org/web/packages/ggridges/vignettes/introduction.html)

### 4.

Specify prior distributions on your parameters. For this exercise, avoid
a uniform prior on sigma and consider an inverse gamma prior.

- $\beta_0 \sim$
- $\beta_1 \sim$
- $\sigma \sim$ InvGamma(, )

### 5.

Write JAGS code to fit this model. Output the results. Hint for an
inverse gamma model in JAGS, use the following..

    phi ~ dgamma(a,b)
    sigma <- 1/phi

### 6.

Compare these results to those from a classical regression model using
`lm()`.

### 7.

Summarize your findings. Write a paragraph so that a college basketball
coach could understand.

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

Note in JAGS you can add the following lines to the model section of
your code

    pp1 ~ dnorm(beta0 + beta1, 1/sigma^2)
    pp7 ~ dnorm(beta0 + beta1 * 7, 1/sigma^2)
    pp15 ~ dnorm(beta0 + beta1 * 15, 1/sigma^2)
