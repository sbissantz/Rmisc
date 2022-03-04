# Reference list

## Estimate your OLS-Estimator

```
# Fake data
set.seed(123)
N <- 100
# Predictor
x <- rnorm(N)
# Outcome
y <- rnorm(x)
```

### Analytical approach 

The analytical approach is 100% matrix algebra. Applying the definition,
gives us our estimate for beta:

```
# Design matrix
X <- cbind(rep(1,N), x)
beta <- solve(t(X) %*% X) %*% t(X) %*% y
```

### lm()

```
lm(y ~ x + 1)
```

### optim()

The basic idea is to minimize the sum of squared residuals using the general
optimization function in R. Remember, der residual is the difference between
the observed value and is (linear) prediction. Squaring the residuals, allows
to toss positive and negative value together. So in the end, we get only *one*
pod of deviations from the mean prediction. The last thing we need to know
is that the linear predictor is  an additive combination of parameters. So
let's call them likewise, in our snippet: 

I got the idea and the code to from Markus Gesmann; so be sure to check out his
blog: <https://www.magesblog.com/post/2013-03-12-how-to-use-optim-in-r/>. 

```
df <- data.frame(x,y)
SR <- function(data, beta) {
    with(data, sum((beta[1] + beta[2] * x - y)^2))
}
optim(par=c(1,1), fn=SR, data=df)$par
```

## ML-Estimator 

We can also use general optimization in R to construct a ML estimator.
I got the basic idea from <https://www.ime.unicamp.br/~cnaber/optim_1.pdf>.
But since I found multiple errors in their code, I adapted it hereafter.

First, let's fit a passion distribution to the data. Our aim is to find the
most likely value. This is the one that maximizes the likelihood function.

```
# Fake data
set.seed(123)
N <- 100
# Parameter 
y <- rpois(N, lambda=2)
mll.pois <- function(mu,y){ 
    n <- length(y) 
    -(sum(y) * log(mu) - n * mu)
    }
optim(par=1,mll.pois,y=y,method="BFGS")
```

Note the "-" inside the function. By default, `optim()` searches the minimum of
a function. Put a minus in front makes the function search for the minimum of
the minus log-likelihood. This is the same as search for the maximum of the
likelihood function.

The same strategy holds for the normal distribution. Our aim is to find the
most likely value. To be more precise, we try to find the one that minimizes
the minus log-likelihood.

```
y <- rnorm(100,0,1)
normal.lik1<-function(theta, y){
    mu <- theta[1] ; sigma_2 <- (theta[2])^2 
    n <- length(y)
    -(-.5*n*log(2*pi) -.5*n*log(sigma_2) - 
    (1/(2*sigma_2))*sum((y-mu)*2))
}
optim(c(0,1),normal.lik1,y=y,method="BFGS")
```

