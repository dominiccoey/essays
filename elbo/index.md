---
layout: page
title: "All hail the ELBO: Why you should care about the evidence lower bound"
permalink: /elbo/
---
#### May 23, 2025

The [evidence lower bound](https://en.wikipedia.org/wiki/Evidence_lower_bound) pops up in a broad range of technical fields. Like entropy or convolutions or Markov processes, it's an example of a highly leveraged concept. If you understand it, you automatically have insights into a variety of different areas—it's just a matter of understanding how the components of the ELBO map to the particular application at hand. Some applications include statistics (the EM algorithm, variational Bayes), ML (diffusion models, variational autoencoders), statistical physics (the principle of minimum energy), computational biology (), neuroscience (Friston’s free-energy principle), and electrical engineering (). 

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

So the marginal likelihood can be written as $\ln p_\theta(x) = L(\phi, \theta; x) +  D_{KL}( q_\phi(z|x) \parallel p_\theta(z|x) )$. And [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) is always positive, so we know that the ELBO $L(\phi, \theta; x)$ is indeed a lower bound on the "evidence", $\ln p_\theta(x)$.[^1] Ok—but what's the point of defining this quantity?

[^1]: Why is $\ln p_\theta(x)$ called the evidence?

## Why is it important?
In brief—if we are facing a problem where the marginal likelihood is intractable
need to evaluate a marginal likelihood, or a posterior, and get the benefits of a generative model

## Applications
