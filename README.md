# 1
200 data from an underlying logistic regression model is generated. This is done in the same manner as importance sampling project. The number of 0s and 1s are reported respectively.
# 2
## functions
The required functions are defined. In these functions, log-likelihoods are used to avoid computational errors.


log_likelihood: takes the data (X and Y) and beta as parameters. Calculates the likelihood of observing the data given a beta.

log_posterior: Its parameters are the data, and standard deviation of the prior (in more general terms it has to be the parameters of the prior, however in this case we already
decided the prior is a gaussian with mean zero which leaves the standard deviation only. That could have been avoided as well since I kept the prior sd at 1.5 through the whole
project). This function returns loglikelihood+logprior (which is simply the log of their product)

log_prior: Its parameters are beta and standard deviation of the gaussian prior we assumed. returns the probability of beta under Gaussian prior.

Metropolis_hastings: Its parameters are the data, number of samples (iterations), the standard deviation of the Gaussian proposal, and the standard deviation of the Gaussian prior.

This function is the implementation of the Metropolis-Hastings algorithm. An initial beta named current beta is generated from the prior. At each iteration, a new beta is proposed which
is sampled from a gaussian centered at the current beta with the proposal sd, then the proposed beta is accepted with the probability of log_posterior(proposed)/log_posterior(current)
The samples array is returned in the end. This array includes the value of each beta at each of the 10000 iterations.


To find the best standard deviation to reach the desired 23.4 acceptance rate I implemented a tuning function that checks for different standard deviations between 0 and 1 
in 0.01 incerements. However I did not use this function for later parts because only using 50 samples was not enough and yielded wrong standard deviations and using more
samples was computationally expensive.

## Plots
The trace plot for each beta is presented. Another set of trace plots only for the first few hundred iterations is presented as well to make a better decision on the burn in
period. The beta values seem to converge after the first 150 iterations so the first 150 values were thrown away.

Histograms are plotted as instructed and the means and standard deviations are reported.

## Gelman Rubin Statistic
In this section we use the Gelman Rubin statistic to evaluate convergence. This statistic compares the variance of between chains with the variance within each chain.

The function's parameter is the chains. N is the number of samples which is one of the dimensions of the chain array.

R hat is then calculated as np.sqrt((1 - 1/N) + (B / (N * W))) where B is variance between chains and W is variance within each chain.

As instructed, 20 chains are produced. I let each chain run for 100 iterations. The Gelman-Rubin statistic is calculated for each iteration and then plotted. 

It is observed that the statistic approaches 1 after around 80 iterations which implies convergence. It is consistent with my visual observation, I only threw away a larger 
burn-in period for higher certainty.

# 3
The dimensionality of the problem is increases as it was in the importance sampling project. Using trial and error I observed that I need to decrease the proposal 
standard deviation to less than 0.2 which is around 35% less than the standard deviation required for the desired acceptance in 3 dimensions.

Besides this difference, the only other visible difference with the case of 3 dimensions was the later convergence shown in both the trace plots and through gelman rubin statistic.

This is rational firstly because the problem is more complex due to higher dimensions and secondly because lowering the standard deviation causes less movement at each iteration.

# 4 
metropolis_hastings_conditional function is defined. Its parameters are the data, the index (which out of 9 betas we are sampling), beta and acceptance rate. The latter two
are only for taking in the initial values in the for loop of the metropolis_within_gibbs function. I added acceptance rate to print the acceptance rate array for betas and find a proposal sd which yields around
15% acceptance rate. It was found to be 2.

In this conditional function only the specified index is updated at each iteration. The function returns either the current beta or the proposed beta based on the previously
explained metropolis hastings rules.

Metropolis_within_gibbs function takes in the data, number or samples(iterations), prior sd and proposal sd. Inside the function there is a nested for loop. At each iteration
of the outer loop, conditionals for all 9 betas are sampled using the inner for loop and the metropolis_hastings_conditional function (we cycle through all indices)

The metropolis_withing_gibbs function is then used for 10 different chains, each of them having 200 iterations.

using Gelman Rubin statistic, we see that this algorithm converges faster. It is obvious that the statistic is very close to 1 from very early on, most of the absolute differences
from 1 being less than 0.2 as early as the 50th iteration.

## 5 
This section is mostly similar to the chain section of question 3. The difference is that the metropolis_hastingsrnd takes in an array of size to as proposal sd and
at each iteration chooses randomly between them to propose a new beta. The sd's I found to yield 10% and 30% acceptance rates were 0.35 and 0.15.

If we compare the Gelman Rubin statistic plot at iteration 50 for this methods vs the method in question 3, we can see a slight improvement (decrease in the statistic) for this
method. This is due to the fact that by alternating between sd's we can benefit from the advantages of both low sd and high sd. When the sd is high, the sampler is more likely
to explore a new point and having the possibility of a low sd decreases its probability of wandering too far.
