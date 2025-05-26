---
layout: page
title: "All hail the ELBO: Why you should care about the evidence lower bound"
permalink: /elbo/
---
#### May 23, 2025

The [evidence lower bound](https://en.wikipedia.org/wiki/Evidence_lower_bound) (ELBO) pops up in a broad range of technical fields. Like entropy or convolutions or Markov processes or convex duality, it's a highly leveraged concept. If you understand it, you have insights into a variety of different areas more or less for free—it's just a matter of understanding how the components of the ELBO map to the particular application at hand. Application areas include ML (diffusion models, variational autoencoders), statistics (the EM algorithm, variational Bayes), statistical physics (the principle of minimum energy), computational biology (), neuroscience (Friston’s free-energy principle), and electrical engineering (). 

This note explains the ELBO and how it applies in the examples above, not assuming much more than a basic statistics background.

## What is it?
**Setup.** We have observed variables $X$ and latent variables $Z$. We have a parametric model $p_\theta(x,z)$, and $(X,Z)$ are distributed according to $p_{\theta_0}(x,z)$, for some true value $\theta_0$. We have an additional model for the conditional distribution $q_\phi(z \mid x)$, the role of which will become clear shortly. 

**Definition.** The evidence lower bound is $L(\phi, \theta; x)$ is

$$
L(\phi, \theta; x) = E_{z\sim q_\phi(\cdot \mid x)} \left[ \ln\frac{p_\theta(x,  z)}{q_\phi(z\mid x)} \right].
$$

**Relation to marginal likelihood.** We can write 

$$
\begin{align*}
L(\phi, \theta; x) &= E_{z\sim q_\phi(\cdot \mid x)} \left[ \ln\frac{p_\theta(x,  z)}{q_\phi(z \mid x)} \right] \\
&= E_{z\sim q_\phi(\cdot \mid x)} \left[ \ln\frac{p_\theta(x) p_\theta(z \mid x)}{q_\phi(z \mid x)} \right]  \\
&= \ln p_\theta(x) + E_{z\sim q_\phi(\cdot \mid x)} \left[ \ln\frac{ p_\theta(z \mid x)}{q_\phi(z \mid x)} \right] \\
&= \ln p_\theta(x) - D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z \mid x))
\end{align*}
$$



So the marginal likelihood is $\ln p_\theta(x) = L(\phi, \theta; x) +  D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z \mid x))$. And [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) is always positive, so we know that the ELBO $L(\phi, \theta; x)$ is indeed a lower bound on the "evidence", $\ln p_\theta(x)$.[^1] Ok—but what's the point of defining this quantity?

[^1]: Why is $\ln p_\theta(x)$ called the evidence?

## Why is it important?
**ELBO helps us approximately maximize intractable likelihood functions.** As additional benefits, by maximizing ELBO we'll also obtain a conditional distribution $q_\phi(z \mid x)$ which approximates the true conditional distribution $p_{\theta_0}(z \mid x)$, and (for an appropriately chosen latent variable structure) we'll be able to easily draw iid samples of new data $X$ similar to the training data. This former point is important in Bayesian statistics, where $Z$'s are the parameters so these conditional distributions are in fact our posteriors given the data. The latter point is important in generative AI, where e.g. we want to generate new images similar to the training images.

## How does it work?
Let's assume our data is sufficiently complicated that it's hard to write down a reasonable parametric statistical model for it with a density $p_\theta(x)$ that is easy to evaluate.[^2] How can we fit this model to the data (in the sense of maximizing the likelihood), if it's prohibitively expensive to even evaluate the density?

Here's where ELBO helps. If 
1. we specify some latent variables $Z$ such that $p_\theta(x,z)$ is easy to compute, and
2. we specify some reasonably flexible model $q_\phi(z \mid x)$ which is also easy to compute and to sample from

then we can use the ELBO in place of the marginal likelihood as an easier to evaluate, surrogate objective. Unpacking this claim:
- _Easier to evaluate_: This is a consequence of 1 and 2. The ratio term in the ELBO, $\frac{p_\theta(x,y)}{q_\phi(z \mid x)}$, is easy to compute by assumption. And to find the expectation with respect to $q_\phi(\cdot \mid x)$, we can use Monte Carlo integration—just sample a bunch of iid $Z_i$'s from that distribution (again, easy by assumption), and then average the term in square brackets in the ELBO over the samples.[^2]
- _Surrogate objective_: The ELBO is a reasonable surrogate, because maximizing the ELBO approximately maximizes the marginal likelihood. Recall $L(\phi, \theta; x) = \ln p_\theta(x) - D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z \mid x))$.  Our gradient steps for $\theta$ will tend to increase the marginal likelihood. This will change $p_\theta(z \mid x)$, and our gradient step for $\phi$ will tighten the KL gap so $q_\phi(z \mid x)$ better tracks $p_\theta(z\mid x)$. This is a bit hand-wavy, but there are formal convergence guarantees, and it seems to work pretty well in practice.

### An example. 
 In many applications $Z$ is a simple, unparameterized, distribution like $N(0,I)$, and $X \mid Z = z$ is $N(\mu(z;\theta), \Sigma(z;\theta))$. The $\mu, \Sigma$ functions can be multilayer neural networks, so this is a rich class of distributions which could adequately model, e.g., complex high-dimensional image data.
 
While in general the marginal density $p_\theta(x) = \int p_\theta(x,z) dy$ is expensive to evaluate accurately, the joint density $p_\theta(x,z)$ is easy to evaluate, since it is the product of the normal densities $p_\theta(x \mid z)$, $p_\theta(z)$.

Similarly, for conditional $q_\phi(z \mid x)$ we can choose $N(m(x;\theta), V(x;\theta))$ for some neural networks $m,V$ for the conditional mean and variance. Thus we've satisfied 1 and 2 above. We can maximize the ELBO, and we because of the latent variable structure we can easily generate new samples—simply draw Z from $N(0,I)$, and then $X$ given $Z = z$ from $N(\mu(z;\theta), \Sigma(z;\theta))$.

[^2]: Why not just evaluate the marginal density itself directly by Monte Carlo? Given the latent variable structure we postulated, we could get an unbiased estimate of $p_\theta(x)$ by taking iid samples $z_1,\ldots, z_n$ from $N(0,I)$, and calculating $\sum_{i = 1}^n p_\theta(x \mid z_i)$. Unfortunately this may be too high variance. To continue the example of image data—almost no latent variables $z$ will generate anything close to a given image $x$, so for every $x$ $p_\theta(x \mid z_i)$ will be close to zero with extremely high probability. ELBO avoids this: We average over the conditional distribution $q_\phi(z \mid x)$. This concentrates probability mass on $z$'s which are likely to have generated $x$, so we won't be averaging over a bunch of terms almost all of which are zero.

[^3]: Normalizing flows..


## Applications
### Statistics
#### The EM algorithm
What if we can easily calculate $p_\theta(z \mid x)$, and we don't need an auxilary model $q_\phi(z \mid x)$? This effectively reduces to the EM algorithm, which iteratively computes $$\theta_{t+1} = \arg\max_{\theta} E_{z \sim p_{\theta_t}(\cdot \mid x)} \ln p_{\theta}(x, z).$$ This procedure is identical to iteratively maximizing the ELBO with respect to $\theta$ with $\phi$ fixed, and with respect to $\phi$ with $\theta$ fixed.

#### Variational Bayes
Here the latent variables $Z$ are the unknown parameters we wish to perform inference on, and $q_\phi(z \mid x)$ is called the **variational  distribution**. In addition to fitting an approximate posterior $q_\phi(z \mid x)$, we get the marginal likelihood $p_\theta (x)$. Comparing the marginal likelihoods across different models is often used for [Bayesian model selection](https://en.wikipedia.org/wiki/Bayes_factor).

### Machine Learning
#### Diffusion models


#### Variational Autoencoders



corresponds to ELBO where 


