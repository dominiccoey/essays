---
layout: page
title: "All hail the ELBO: Why you should care about the evidence lower bound"
permalink: /elbo/
---
#### May 23, 2025

The [evidence lower bound](https://en.wikipedia.org/wiki/Evidence_lower_bound) (ELBO) pops up in a broad range of technical fields. Like entropy or convolutions or Markov processes or convex duality, it's a highly leveraged concept. If you understand it, you have insights into a variety of different areas more or less for free—it's just a matter of understanding how the components of the ELBO map to the particular application at hand. Some applications include statistics (the EM algorithm, variational Bayes), ML (diffusion models, variational autoencoders), statistical physics (the principle of minimum energy), computational biology (), neuroscience (Friston’s free-energy principle), and electrical engineering (). 

This note explains the ELBO and how it applies in the examples above, not assuming much more than a basic statistics background.

## What is it?
**Setup.** We have observed variables $X$ and latent variables $Z$. We have a parametric model $p_\theta(x,z)$, and $(X,Z)$ are distributed according to $p_{\theta_0}(x,z)$, for some true value $\theta_0$. We also have a model for the conditional distribution $q_\phi(z \mid x)$, the role of which will become clear shortly. 

**Definition.** The evidence lower bound is $L(\phi, \theta; x)$ is defined as

$$
L(\phi, \theta; x) = \mathbb E_{z\sim q_\phi(\cdot | x)} \left[ \ln\frac{p_\theta(x,  z)}{q_\phi(z|x)} \right].
$$

**Relation to marginal likelihood.** We can write 

$$
\begin{align}
L(\phi, \theta; x) &= \mathbb E_{z\sim q_\phi(\cdot | x)} \left[ \ln\frac{p_\theta(x,  z)}{q_\phi(z|x)} \right] \\
&= \mathbb E_{z\sim q_\phi(\cdot | x)} \left[ \ln\frac{p_\theta(x) p_\theta(z \mid x)}{q_\phi(z|x)} \right]  \\
&= \ln p_\theta(x) + \mathbb E_{z\sim q_\phi(\cdot | x)} \left[ \ln\frac{ p_\theta(z \mid x)}{q_\phi(z|x)} \right] \\
&= \ln p_\theta(x) - D_{KL}(q_\phi(z | x) \parallel p_\theta(z \mid x))
\end{align}
$$

So the marginal likelihood is $\ln p_\theta(x) = L(\phi, \theta; x) +  D_{KL}( q_\phi(z|x) \parallel p_\theta(z|x) )$. And [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) is always positive, so we know that the ELBO $L(\phi, \theta; x)$ is indeed a lower bound on the "evidence", $\ln p_\theta(x)$.[^1] Ok—but what's the point of defining this quantity?

[^1]: Why is $\ln p_\theta(x)$ called the evidence?

## Why is it important?
**ELBO helps us approximately maximize intractable likelihood functions.** As additional benefits, from maximizing ELBO we'll also obtain a conditional distribution $q_\phi(z \mid x)$ which approximates the true conditional distribution $p_{\theta_0}(z \mid x)$, and we'll be able to easily draw iid samples of new data $X$ similar to the training data. This former point is important in Bayesian statistics, where $Z$'s are the parameters so these conditional distributions are in fact our posteriors given the data. The latter point is important in generative AI, where e.g. we want to generate new images similar to the training images.

### How does ELBO deliver these benefits?
Let's assume our data is sufficiently complicated that it's hard to write down a reasonable parametric statistical model for it with a density $p_\theta(x)$ that is easy to evaluate.[^2] How can we fit this model to the data (in the sense of maximizing the likelihood), if it's prohibitively expensive to even evaluate the density?

Here's where ELBO helps. If 
1. we specify some latent variables $Z$ such that $p_\theta(x,z)$ is easy to compute, and
2. we specify some reasonably flexible model $q_\phi(z \mid x)$ which is also easy to compute and to sample from

then we can use the ELBO in place of the marginal likelihood as a surrogate objective. Maximizing the ELBO will approximately maximize the marginal likelihood.

The fact that the ELBO is will be easier to evaluate than the marginal likelihood is a consequence of 1 and 2. 

### An example. 
 
 extra flexiblity bough because the mean and variance functions may be complex neural networks...


[^2]: Normalizing flows..

### How 



### Couldn't we just have evaluated the marginal likelihood by MC too?

need to evaluate a marginal likelihood, or a posterior, and get the benefits of a generative model

## Applications
