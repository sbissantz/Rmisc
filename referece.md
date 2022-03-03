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

```
# Design matrix
X <- cbind(rep(1,N), x)
beta <- solve(t(X) %*% X)%*%t(X)%*%y
```

### lm()

```
lm(y ~ x + 1)
```

### optim()
see: <https://www.magesblog.com/post/2013-03-12-how-to-use-optim-in-r/>

The basic idea is to minimize the sum of squared residuals: \(\min S(R))\). In
matrix notation: \(\min S(R) = \min(\(hat{y} = X \beta)^2\). If we only have to
predictors the expression simplifies to \(\min S(R) = \min(\(hat{y} - alpha +
beta * X)^2\). \(\mathbf{beta} \) are the parameters of the model; so lets call
them likewise (`par`). Finally, note that `optim` does always minimize a
function, so giving it \(S(R)\) yields: 

```
df <- data.frame(x,y)
SR <- function(data, par) {
    with(data, sum((par[1] + par[2] * x - y)^2))
}
optim(par=c(0,1), fn=SR, data=df)$par
```

## Estimate your ML-Estimator (optim())
see: <https://www.magesblog.com/post/2013-03-12-how-to-use-optim-in-r/>

Here we fit a passion distribution to the data. Our aim is to find the most
likely value. This is the one that maximizes the likelihood.

```
# Fake data
set.seed(123)
N <- 100
# Parameter 
x <- rpois(N, lambda=2)
```

```
lklh.poisson <- function(x, lambda) lambda^x/factorial(x) * exp(-lambda)

# Since optim minimizes by default -- maximize with "-"
log.lklh.poisson <- function(x, lambda){ 
                     -sum(x * log(lambda) - log(factorial(x)) - lambda) 
                     }
optim(par = 2, log.lklh.poisson, x = x)
# Incoporate R's advice
optim(par = 2, fn = log.lklh.poisson, x = x, method = "Brent", lower = 2, upper
= 3)
```

