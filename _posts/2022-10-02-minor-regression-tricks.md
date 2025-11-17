---
layout: post
title: Small Tricks in Linear Regression
date: 2022-10-02
description: Practical techniques for linear regression analysis, covering estimates and standard errors for sample means, and running multiple regressions simultaneously
tags: regression econometrics code
categories: notes
---

# Estimates and SE of Some Sample Mean

Sometimes we are interested in statistics $$\theta$$ that can be estimated by the sample mean of some function

$$
\hat\theta = \frac1n \sum_i f(W_i)
$$

For example, $$\hat\theta$$ is the MSE of a predictor $$\hat{g}(X_i)$$ of $$\mathbb{E}[Y_i|X_i]$$

$$
\hat\theta = \frac1n \sum_i (Y_i - \hat{g}(X_i))^2
$$

with $$f(W_i) = (Y_i - \hat{g}(X_i))^2$$.

Calculating $$\hat\theta$$ is simple but obtaining the standard error of $$\hat\theta$$ requires extra efforts. In this case, we can utilize the convenient linear regression functions in statistical softwares. The sample mean of some function $$f(W_i)$$ is equivalent to regressing $$f(W_i)$$ on a vector of $$1$$.

```r
set.seed(1)

n <- 1000
X <- rnorm(n)
Y <- 1 + 2 * X + rnorm(n)

mod <- lm(Y ~ X)
gX <- mod$fitted.values

# MSE by sample mean
mse <- mean((Y - gX)^2)
cat("The MSE of gX is", mse)

# MSE by linear regression
mse.lm <- lm((Y - gX)^2 ~ 1) |> summary()
cat("The MSE of gX by linear regression is", mse.lm$coef[1], "with SE", mse.lm$coef[2])
```

We can easily obtain the standard error of the MSE but notice that by doing this, the sum of squared in the MSE is divided by $$n$$. In small samples, we need to change the denominator to $$n - 2$$ (lost two degrees of freedom in $$\hat\beta_0$$ and $$\hat\beta_1$$ in $$gX$$) by multiplying $$n/(n - 2)$$ and also adjust the standard error accordingly. After adjustment, the MSE is the same as in the anova table.

```r
mse.lm$coefficients[1] * (n / (n - 2))
anova(mod)['Residuals', 'Mean Sq']
```

# Multiple Regressions at One Time

Sometimes, we may want to compare coefficients in two different regressions. In this case, a formal hypothesis test is hard to implement because we need a variance-covariance matrix of the coefficients in different regressions. One easy solution is to trick statistical softwares to run two regression at one time by stacking the two regressions.

Suppose we want to compare the coefficients $$\alpha$$ and $$\beta$$ in regressions

$$
Y = X\alpha + \epsilon, \quad \quad Z = D\beta + \nu
$$

We can stack the two samples and obtain a long response variable $$(Y', Z')'$$ and a large design matrix

$$
\begin{pmatrix}
X & 0 \\
0 & D
\end{pmatrix}
$$

so that we have

$$
\begin{pmatrix} Y \\ Z \end{pmatrix} = \begin{pmatrix} X & 0 \\ 0 & D \end{pmatrix} \begin{pmatrix} \alpha \\ \beta \end{pmatrix} + \begin{pmatrix} \epsilon \\ \nu \end{pmatrix}
$$

By running this stacked regression, we are able to estimate $$\alpha$$ and $$\beta$$ at the same time and obtain their variance-covariance matrix. **Note clustered standard error is recommended for the stacked regression**. This trick is similar to Seemingly Unrelated Regression (SUR), but the benefit is that this method can be extended to IV regressions (or a combination of IV and OLS) by regarding OLS as a special case of IV where the regressors are instrumented by themselves. Below is an example using the dataset of High School and Beyond survey.

```r
hsb2 <- foreign::read.dta("https://stats.idre.ucla.edu/stat/stata/notes/hsb2.dta")
hsb2$female <- ifelse(hsb2$female == "female", 1, 0)
hsb2$ses <- as.numeric(hsb2$ses)

# Separate regressions
summary(lm(read ~ female + ses + socst, data = hsb2))$coef[2, 1:2]
summary(lm(math ~ female + ses + science, data = hsb2))$coef[2, 1:2]

# Stacked regression
read.math <- with(hsb2, c(read, math))
X <- cbind(hsb2 |> select("female", "ses", "socst"), matrix(rep(0, nrow(hsb2) * 3), nrow(hsb2), 3))
D <- cbind(matrix(rep(0, nrow(hsb2) * 3), nrow(hsb2), 3), hsb2 |> select("female", "ses", "science"))
df <- data.frame(read.math, rbind(as.matrix(X), as.matrix(D))) |>
  setNames(c("rm", "female.r", "ses.r", "socst", "female.m", "ses.m", "science"))
df$sid <- c(rep(1, nrow(hsb2)), rep(0, nrow(hsb2)))

stacked <- lm(rm ~ ., data = df)
coef(stacked)[c(2, 5)]
vcov(stacked, type = "const", cluster = c(rep(1, nrow(hsb2)), rep(0, nrow(hsb2))))[c(2, 5), c(2, 5)]
```
