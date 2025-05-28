---
layout: page
title: "All hail the ELBO: Why you should care about the evidence lower bound"
permalink: /elbo/
---
#### May 23, 2025

The [evidence lower bound](https://en.wikipedia.org/wiki/Evidence_lower_bound) (ELBO) pops up in a broad range of technical fields. Like entropy or convolutions or Markov processes or convex duality, it's a highly leveraged concept. If you understand it, you have insights into a variety of different areas more or less for free—it's just a matter of understanding how the components of the ELBO map to the particular application at hand. Application areas include statistics (the EM algorithm, variational Bayes), ML (variational autoencoders, diffusion models), statistical physics (variational statistical mechanics), computational biology (modeling single-cell gene expression), neuroscience (the free-energy principle), and information theory (iterative decoding). 

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

So the marginal likelihood is $\ln p_\theta(x) = L(\phi, \theta; x) +  D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z \mid x))$. And [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) is always positive, so we know that the ELBO $L(\phi, \theta; x)$ is indeed a lower bound on the "evidence", $\ln p_\theta(x)$.[^1] 


**Reconstruction plus consistency.** Another formulation is 

$$
L(\phi, \theta; x) = E_{z\sim q_\phi(\cdot \mid x)}[\ln p_\theta(x\mid z)] - D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z)).
$$

[^1]: Why is $\ln p_\theta(x)$ called the evidence?

Ok—but what's the point of defining this quantity?

## Why is it important?
**ELBO helps us approximately maximize intractable likelihood functions.** As additional benefits, by maximizing ELBO we'll also obtain a conditional distribution $q_\phi(z \mid x)$ which approximates the true conditional distribution $p_{\theta_0}(z \mid x)$, and (for an appropriately chosen latent variable structure) we'll be able to easily draw iid samples of new data $X$ similar to the training data. This former point is important in Bayesian statistics, where $Z$'s are the parameters so these conditional distributions are in fact our posteriors given the data. The latter point is important in generative AI, where we want to generate new images similar to the training images.

## How does it work?
Let's assume our data is sufficiently complicated that it's hard to write down a reasonable parametric statistical model for it with a density $p_\theta(x)$ that is easy to evaluate.[^2] How can we fit this model to the data (in the sense of maximizing the likelihood), if it's prohibitively expensive to even evaluate the density?

Here's where ELBO helps. If 
1. we specify some latent variables $Z$ such that $p_\theta(x,z)$ is easy to compute, and
2. we specify some reasonably flexible model $q_\phi(z \mid x)$ which is also easy to compute and to sample from

then we can use the ELBO in place of the marginal likelihood as an easier to evaluate, surrogate objective. Unpacking this claim:
- _Easier to evaluate_: This is a consequence of 1 and 2. The ratio term in the ELBO, $\frac{p_\theta(x,y)}{q_\phi(z \mid x)}$, is easy to compute by assumption. And to find the expectation with respect to $q_\phi(\cdot \mid x)$, we can use Monte Carlo integration—just sample a bunch of iid $Z_i$'s from that distribution (again, easy by assumption), and then average the term in square brackets in the ELBO over the samples.[^3]
- _Surrogate objective_: The ELBO is a reasonable surrogate, because maximizing the ELBO approximately maximizes the marginal likelihood. Recall $L(\phi, \theta; x) = \ln p_\theta(x) - D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z \mid x))$.  Our gradient steps for $\theta$ will tend to increase the marginal likelihood. This will change $p_\theta(z \mid x)$, and our gradient step for $\phi$ will tighten the KL gap so $q_\phi(z \mid x)$ better tracks $p_\theta(z\mid x)$. This is a bit hand-wavy, but there are formal convergence guarantees [[CITE HERE]], and it seems to work pretty well in practice.


**[edit] The first term on the RHS—the _reconstruction term_—encourages the model to learn a $q_\phi(z \mid x)$ and a $p_\theta(x\mid z)$ so that the latent $z$ capture enough information about $x$ to allow $p_\theta(x\mid z)$ to generate something very similar to $x$. The second term acts as a regularizer, and encourages $q_\phi(z \mid x)$ to be close to the prior $p_\theta(z)$ and not vary wildly with $x$.**


[^2]: Normalizing flows..

[^3]: Why not just evaluate the marginal density itself directly by Monte Carlo? If we can evaluate $p_\theta(x \mid z)$ easily, we can get an unbiased estimate of $p_\theta(x)$ by taking iid samples $z_1,\ldots, z_n$ from $N(0,I)$, and calculating $\sum_{i = 1}^n p_\theta(x \mid z_i)$. Unfortunately this may be too high variance. Consider image data—almost no latent variables $z$ will generate anything close to a given image $x$, so for every $x$ $p_\theta(x \mid z_i)$ will be close to zero with extremely high probability. ELBO avoids this, by averaging over the conditional distribution $q_\phi(z \mid x)$. This concentrates probability mass on $z$'s which are likely to have generated $x$, so we won't be averaging over a bunch of terms almost all of which are zero.


## Applications
### Statistics
#### The EM algorithm
What if we can easily calculate $p_\theta(z \mid x)$, and we don't need an auxilary model $q_\phi(z \mid x)$? This effectively reduces to the EM algorithm, which iteratively computes 

$$\theta_{t+1} = \arg\max_{\theta} E_{z \sim p_{\theta_t}(\cdot \mid x)} \ln p_{\theta}(x, z).$$

This procedure is identical to iteratively maximizing the ELBO with respect to $\theta$ with $\phi$ fixed, and with respect to $\phi$ with $\theta$ fixed.

#### Variational Bayes
ELBO provides an alternative to finding the posterior by MCMC. Here the latent variables $Z$ are the unknown parameters we wish to perform inference on, and $q_\phi(z \mid x)$ is called the **variational  distribution**. In addition to fitting an approximate posterior $q_\phi(z \mid x)$, we get the marginal likelihood $p_\theta (x)$. Comparing the marginal likelihoods across different models is often used for [Bayesian model selection](https://en.wikipedia.org/wiki/Bayes_factor).


### Machine Learning
#### Variational Autoencoders
In a variational autoencoder (VAE) we specify an encoder (which turns the data $x$ into a latent variable $z$) and a decoder (the reverse operation), each of which have unknown parameters we estimate by maximizing the ELBO. 
- **Encoder**, $x \to z$: The latent variable $Z$ is $N(0,I)$, and $X \mid Z = z$ is $N(\mu(z;\theta), \Sigma(z;\theta))$. The $\mu, \Sigma$ functions are neural networks, so this is a rich class of distributions which could adequately model, e.g., complex high-dimensional image data.
- **Decoder**, $z \to x$: We model $q_\phi(z \mid x)$ as $N(m(x;\theta), V(x;\theta))$ for some neural networks $m,V$ for the conditional mean and variance. 

This structure is well-suited to the ELBO approach. In general the marginal density $p_\theta(x) = \int p_\theta(x,z) dy$ is expensive to evaluate accurately, but the joint density $p_\theta(x,z)$ is easy to evaluate, since it is the product of the normal densities $p_\theta(x \mid z)$, $p_\theta(z)$.

Because of the latent variable structure we can easily generate new samples from the model we fit—simply draw $Z$ from $N(0,I)$, and then $X \mid Z = z$ from $N(\mu(z;\theta), \Sigma(z;\theta))$. 

#### Diffusion Models
Unlike VAEs, we the encoding process has no unknown parameters, and involves iteratively adding noise with known variance to the input data. This means that $q_\phi ( z \mid x)$ can be treated as fixed throughout, with no unknown parameters. The goal is instead to learn the decoder, which turns $z$ into $x$. In more detail:

- **Encoder**, $x \to z$: the latent variable $z$ is a _sequence_ $z_1,\ldots,z_T$, which adds progressively more noise to the starting data $x$. We start with $z_1 = \sqrt{1 - \beta_1} x + \sqrt{\beta_1} \varepsilon_1$ and continue iterating $z_t = \sqrt{1 - \beta_t} z_{t - 1} + \sqrt{\beta_t} \varepsilon_t$, for $t>1$ and a known sequence $\beta_1,\ldots,\beta_T$. The noise terms $\varepsilon_t$ are independent $N(0,I)$. As $T \to \infty$, all this added noise means $z_T$'s distribution becomes close to $N(0,I)$. 

- **Decoder**, $z \to x$: we model $p_\theta(z_{t-1} \mid z_t)$ as normally distributed. Then we rewrite $p_\theta(x, z)$ in the ELBO as $p(z_T) \left[ \prod_{t = 2}^T p_\theta(z_{t-1} \mid z_t) \right] p_\theta(x \mid z_1)$, and maximize the ELBO with respect to $\theta$ (because $z_T$ is treated as being $N(0,I)$ distributed, it is not parameterized by $\theta$).

As with VAEs, generating new samples is easy once the decoder is learned, as we have the distributions of $Z$ and $X \mid Z$.

### Statistical Physics
#### Variational Statistical Mechanics
[Some physical systems](https://en.wikipedia.org/wiki/Canonical_ensemble) are well described by the [Boltzmann distribution](https://en.wikipedia.org/wiki/Boltzmann_distribution), which specifies the probability of a state $i \in \\{1,\ldots,M\\}$ as

$$
p_i=\frac{1}{Q} \exp\left(-  \varepsilon_i \right)  = \frac{ \exp\left(-  \varepsilon_i \right) }{ \displaystyle \sum_{j=1}^{M} \exp\left(- \varepsilon_j \right) }.
$$

The normalizing constant $Q$ is known as the "partition function", and $\varepsilon_i$ is the energy of state $i$.[^4] Energy in any given system state $z_i$ is a known system-specific function $E$, called the Hamiltonian: $\varepsilon_i = E(z_i)$. The partition function plays a critical role in determining the physical behavior of the system, but may take impractically long to compute if the number of system states $M$ is enormous.

This is where ELBO comes in. We write the log joint density of the ELBO as $\log p(x,z) = -E(z)$, where we consider the $x$'s to be fixed system parameters rather than a random variable. Then the "marginal likelihood", obtained from summing over states $z$, is exactly the partition function! With a tractable model $q(z)$ of the distribution of system states, we can find a lower bound on the log partition function by maximizing the ELBO. The optimization will often involve variational methods, and the negative ELBO is the called the variational free energy. If $z$ is vector-valued, for example, we may make $q$ tractable via the _mean-field_ assumption, according to which the component elements are independent. 

See [here](https://ml4physicalsciences.github.io/2019/files/NeurIPS_ML4PS_2019_92.pdf) for more on the equivalence between the ELBO and variational methods in physics.

[^4]: Implicitly the [thermodynamic beta](https://en.wikipedia.org/wiki/Thermodynamic_beta) is set to one here—we can think of this as rescaling energies so that $\beta = 1$.

### Computational Biology
#### Modeling Single-cell Gene Expression
Variational methods appear to becoming more common in biology. One nice application is ["Deep generative modeling for single-cell transcriptomics"](https://www.nature.com/articles/s41592-018-0229-2). Here the data $x$ describes, for each cell, the degree to which it expresses one of many genes. The $z$ is the latent structure, which is much lower-dimensional than the number of genes: a cell is a 10-dimensional vector in latent space, but expresses 100s - 10,000s genes in these datasets. The modeling is similar to the variational autoencoder network example above, except the likelihood $p_\theta(x \mid z)$ is zero-inflated negative binomial, to fit the count data. Learning the latent structure allows us cluster or classify cells. Especially intriguing though is cell _generation_—we can use this model to generate new synthetic cells, just as a VAE trained on image data allows us to generate new images! 

### Neuroscience 
#### Free-Energy Principle
An influential theory in neuroscience is Karl Friston's [Free-Energy Princple](https://www.nature.com/articles/nrn2787). It exemplifies the Bayesian brain hypothesis, which posits that the brain has a built-in generative model of the world which it uses to construct posterior beliefs. Friston's theory has the brain effectively performing variational inference to understand the world.

Here the latent variables $z$ are physical events and features in the world, which then generate sensory input $x$  (e.g. light on the retina or cochlear vibration). The brain is assumed to have some understanding of the kinds of $z$ out in the world, and how they map to $x$—in other words, a generative model $p_\theta(x)$ and $p_\theta(z \mid x)$. We wish to form an accurate understanding of what's going on in the physical world given the sense data, i.e. an approximate posterior $q_\phi(z \mid x)$, called the "recognition density" here. The parameters $(\theta, \phi)$ correspond to synaptic strengths in the brain. 

We have all the elements necessary to apply variational inference. Instead of maximizing the ELBO, we talk of "minimizing free energy" or "minimizing surprise", although this is just a point of terminology—the mathematics is the same.


### Information Theory
#### Iterative Decoding
infomax principle? efficient coding?
