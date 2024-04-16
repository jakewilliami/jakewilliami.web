+++
title = "Understanding a little bit about Posterior Probability"
date = 2019-10-18

[extra]
katex = true
+++

$$
  \text{Pr}(p|W,N)=\frac{\text{Pr}(W|N,p)\text{Pr}(p)}{\sum\text{Pr}(W|N,p)\text{Pr}(p)\forall p}
$$
$$
  \text{Posterior}=\frac{(\text{Prob. observed variables}\times(\text{Prior})}{\text{Normalising constant}}
$$

(Where the normalising constant standardises it).  The probability of observed variables is also called the "likelihood".

Calculating posterior:
  1. analytical approach (often impossible)
  2. Grid approximation (very intensive)
  3. Quadratic approximation (limited) (A.K.A. Lacrosse approximation)
  4. Markov chain monte carlo (intensive)

Here is some example R code to do such a thing:
```R
# dbinom(6, size=9, prob=0.5)
p_grid <- seq(from=0, to=1, length.out=1000)
prob_p <- rep(1, 1000)
prob_data <- dbinom(6, size=9, prob=p_grid)
posterior <- prob_data * prob_p
posterior <- posterior / sum(posterior) # standardise it
```
