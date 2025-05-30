---
layout: page
title: "All hail the ELBO: Why you should care about the evidence lower bound"
permalink: /elbo/
---
#### May 30, 2025

The [evidence lower bound](https://en.wikipedia.org/wiki/Evidence_lower_bound) (ELBO) comes up in a broad range of technical fields. Like entropy or convolutions or Markov processes or convex duality, it's a highly leveraged concept. If you understand it, you have insights into many different areas more or less for free—it's just a matter of understanding how the components of the ELBO map to the particular application at hand. Remarkably, diffusion models in generative AI, variational statistical mechanics, and the free-energy model of the brain all share a common mathematical structure. 

This note explains the ELBO and how it applies, in a [somewhat eclectic selection of examples](#what-does-this-look-like-in-practice) from statistics, machine learning, statistical physics, computational biology, and neuroscience.

## Problem statement
We have observed random variables $x$ and latent random variables $z$. We have a statistical model $p_\theta(x,z)$, and $(x,z)$ are distributed according to $p_{\theta_0}(x,z)$, for some true value $\theta_0$. We want to calculate the marginal likelihood $p_\theta(x) = \int p_\theta(x,z) \\ dz$ or the conditional distribution $p_\theta(z \mid x) = \frac{p_\theta(x,z)}{\int p_\theta(x,z) \\ dz} $, or both. Why we might want to do this will become clear in the applications below. But evaluating the integral $\int p_\theta(x,z) \\ dz$ may be very expensive. How can we quickly approximate these quantities, instead of slowly computing them exactly?

## What is the ELBO?
Let's specify an additional model for the conditional distribution, $q_\phi(z \mid x)$, which will be used to approximate the true conditional $p_{\theta_0}(z \mid x)$.

**Definition.** The _evidence lower bound_, $L(\phi, \theta; x)$, is

$$
L(\phi, \theta; x) = E_{z\sim q_\phi(\cdot \mid x)} \left[ \ln\frac{p_\theta(x,  z)}{q_\phi(z\mid x)} \right].
$$

This may seem a bit mysterious. Two equivalent perspectives shed more light on this definition.

**Relation to marginal likelihood.** We can write 

$$
\begin{align*}
L(\phi, \theta; x)
&= E_{z\sim q_\phi(\cdot \mid x)} \left[ \ln\frac{p_\theta(x) p_\theta(z \mid x)}{q_\phi(z \mid x)} \right]  \\
&= \ln p_\theta(x) + E_{z\sim q_\phi(\cdot \mid x)} \left[ \ln\frac{ p_\theta(z \mid x)}{q_\phi(z \mid x)} \right] \\
&= \ln p_\theta(x) - D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z \mid x))
\end{align*}
$$

Because [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence) is always non-negative, the ELBO $L(\phi, \theta; x)$ is indeed a lower bound on the log evidence, $\ln p_\theta(x)$.[^1] Maximizing the ELBO with respect to $(\phi, \theta)$ encourages both a large marginal likelihood $p_\theta(x)$ and conditional distributions $q_\phi(z \mid x), p_\theta(z \mid x)$ which are close to each other.


**Reconstruction plus regularization.** Equivalently we can write

$$
\begin{align*}
L(\phi, \theta; x) 
&= E_{z\sim q_\phi(\cdot \mid x)} \left[ \ln\frac{p_\theta(z) p_\theta(x \mid z)}{q_\phi(z \mid x)} \right]  \\
&= E_{z\sim q_\phi(\cdot \mid x)}[\ln p_\theta(x\mid z)] - D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z)).
\end{align*}
$$

Let's again consider what happens when we maximize the ELBO over $(\phi, \theta)$. The first term on the RHS—the _reconstruction_ term—encourages the model to learn a $q_\phi(z \mid x)$ and a $p_\theta(x\mid z)$ so that the latent $z$ capture enough information about $x$ to allow $p_\theta(x\mid z)$ to generate something very similar to $x$. The second term—the _regularization_ term—rewards a $q_\phi(z \mid x)$ which is close to the prior $p_\theta(z)$, and in particular does not vary wildly with $x$.

[^1]: Why is $p_\theta(x)$ called the evidence? For Bayesian model selection, we often compare the ratio of marginal likelihoods under different models (the [Bayes factor](https://en.wikipedia.org/wiki/Bayes_factor)). In this sense, marginal likelihood is a measure of the strength of support in the data for a particular model.

## How does it help?
**The ELBO lets us replace an intractable marginal likelihood with a tractable surrogate.** How? Let's assume our data is sufficiently complicated that it's hard to write down a reasonable statistical model for it with a marginal likelihood $p_\theta(x)$ that is easy to evaluate.[^2] If 

1. we specify some latent variables $z$ such that $p_\theta(x,z)$ is easy to compute, and
2. we specify some reasonably flexible model $q_\phi(z \mid x)$ which is also easy to compute and to sample from

then we can use the ELBO in place of the marginal likelihood as an easier to evaluate, surrogate objective. Unpacking this claim:
- _Easier to evaluate_: This is a consequence of 1 and 2. The ratio term in the ELBO, $\frac{p_\theta(x,z)}{q_\phi(z \mid x)}$, is easy to compute by assumption. And to find the expectation with respect to $q_\phi(\cdot \mid x)$, we can use Monte Carlo integration—just sample a bunch of iid $z_i$'s from that distribution (again, easy by assumption), and then average the term in square brackets in the ELBO over the samples.[^3]
- _Surrogate objective_: The ELBO is often a reasonable surrogate. Recall $L(\phi, \theta; x) = \ln p_\theta(x) - D_{KL}(q_\phi(z \mid x) \parallel p_\theta(z \mid x))$. If we choose a rich class of approximating posteriors $q_\phi(z \mid x)$, the KL divergence term will tend to be small, and maximizing ELBO approximately maximizes the marginal likelihood.

## What does this look like in practice?
Below are some methodological and empirical applications of the general idea of ELBO maximization. It's far from comprehensive or representative (I'm omitting the [famous LDA paper](https://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf)), but should give a sense of how to take the basic ideas above further. This is rather compact, and each of these applications is described in more detail [in the section below](#elbos-everywhere).

|  Application | Goal |  $x$  | $z$ | $\theta$ | $\phi$
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| [_EM algorithm_](https://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm)    | estimate parameters $\theta$ | depends on use-case  | depends on use-case   | parameterizes joint distribution of $(x,z)$ | none, posterior $z \mid x$ known exactly given $\theta$ |
| [_Variational Bayes_](https://en.wikipedia.org/wiki/Variational_Bayesian_methods) | approximate the posterior $z \mid x$ | depends on use-case | parameters of interest | none | describes the nonparametric approximate posterior | 
| [_Empirical Bayes_](https://en.wikipedia.org/wiki/Empirical_Bayes_method)  | estimate hyperparameters $\theta$ | depends on use-case  | parameters of interest | hyperparameters of joint distribution of $(x,z)$  | parameterizes approximating posterior |
| [_VAEs_](https://en.wikipedia.org/wiki/Variational_autoencoder) | create a generative model of the input data | depends on use-case, often images |  lower-dimensional input representation | parameterizes decoder neural network | parameterizes encoder neural network |
| [_Diffusion models_](https://en.wikipedia.org/wiki/Diffusion_model) | create a generative model of the input data | depends on use-case, often images |  sequence of increasingly noisy versions of $x$ | parameterizes decoder neural network | none, encoder known |
| [_Variational statistical mechanics_](https://en.wikipedia.org/wiki/Helmholtz_free_energy)  | approximate the partition function | none | physical states | none | describes the nonparametric approximate state distribution |
| [_Single-cell gene expression_](https://www.nature.com/articles/s41592-018-0229-2)  | approximate the posterior $z \mid x$  | cell-level gene expression  | lower-dimensional cell representation | parameterizes distribution of $x \mid z$ | parameterizes approximate posterior |
| [_Free-energy theory of the brain_](https://www.nature.com/articles/nrn2787)  | approximate the posterior $z \mid x$  | sensory data  | state of the world  | parameterizes distribution of $x \mid z$ | parameterizes approximate posterior |


[^2]: [Normalizing flows](https://en.wikipedia.org/wiki/Flow-based_generative_model) are a different approach, which doesn't compromise on exact likelihood evaluation. They seek to specify a class of distributions such that this is still possible, but which is large and flexible enough to model the data well.

[^3]: Why not just evaluate the marginal density itself directly by Monte Carlo? If we can evaluate $p_\theta(x \mid z)$ easily, we can get an unbiased estimate of $p_\theta(x)$ by taking iid samples $z_1,\ldots, z_n$ from $p(z)$, and calculating $\tfrac{1}{n}\sum_{i = 1}^n p_\theta(x \mid z_i)$. Unfortunately this may be too high variance. Consider image data—almost no latent variables $z$ will generate anything close to a given image $x$, so for every $x$ $p_\theta(x \mid z_i)$ will be close to zero with extremely high probability. ELBO avoids this, by averaging over the conditional distribution $q_\phi(z \mid x)$. This concentrates probability mass on $z$ values most likely to have generated $x$, so we won't be averaging over a bunch of terms almost all of which are zero.



## ELBOs everywhere!
### Statistics
#### The EM algorithm
What if we can easily calculate $p_\theta(z \mid x)$, and we don't need an auxiliary model $q_\phi(z \mid x)$? This kind of situation, where it's easy to evaluate $p\theta(x,z)$ and $p_\theta(z \mid x)$ but not $p_\theta(x)$, arises in e.g. [Gaussian mixture models](https://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm#Gaussian_mixture). In this case ELBO maximization effectively reduces to the EM algorithm, which iteratively computes 

$$\theta_{t+1} = \arg\max_{\theta} E_{z \sim p_{\theta_t}(\cdot \mid x)} \ln p_{\theta}(x, z).$$

This procedure is identical to iteratively maximizing the ELBO with respect to $\theta$ with $\phi$ fixed, and with respect to $\phi$ with $\theta$ fixed. The latter step boils down to to directly calculating the conditional distribution $p_\theta(z \mid x)$.

#### Variational Bayes
ELBO provides an alternative to finding the posterior by MCMC. The latent variables $z$ are the unknown parameters of interest. The distribution $p(x,z)$ is _not_ parameterized by some unknown $\theta$ we're trying to estimate—in a fully Bayesian model, we specify the prior over all unknown parameters $p(z)$, as well as the likelihood $p(x \mid z)$. We'd like to know the posterior $p(z \mid x)$. What are our options?

MCMC methods allow us, at least in the limit, to draw exact samples from the posterior. But they may be quite computationally intensive. Variational methods deliver an approximate posterior, typically at much less computational cost. They work as follows: we specify approximating posteriors $q( z \mid x)$ where $q$ belongs to some family of densities $Q$, and we maximize the ELBO with respect to $q \in Q$. We _don't_ want $Q$  be the set of all possible densities over $z. This would be equivalent to finding the true posterior, and we wouldn't have simplified the problem. Instead we optimize over some restricted class of densities. 

An example of such a restriction when $z$ is vector-valued is that $Q$ is a _mean-field_ family. Write $z = (z^{(1)},\ldots,z^{(k)})$. Then the mean-field assumption is just that the components are independent, and $q$ factors as $q(z \mid x) = \prod_{i = 1}^k q_i( z^{(i)} \mid x)$. [This paper](https://arxiv.org/pdf/1601.00670) is an excellent introduction to these ideas.

#### Empirical Bayes Hyperparameter Selection
In empirical Bayes, as in variational Bayes, the $z$ are parameters of interest. Unlike in variational Bayes, we _do_ have some unknown hyperparameters in the density $p_\theta(x, z)$. The classic approach would be to choose them by maximizing the marginal likelihood. And if that's hard to compute? Maximizing ELBO is a natural alternative. The argument is that if we choose a rich enough posterior family, we'll get close to the true posterior, and maximizing ELBO will be close to maximizing the marginal likelihood.

### Machine Learning
#### Variational Autoencoders
In a variational autoencoder (VAE) we specify an encoder (which turns the data $x$ into a latent variable $z$) and a decoder (the reverse operation), each of which have unknown parameters we estimate by maximizing the ELBO. 
- **Encoder**, $x \to z$: We model $q_\phi(z \mid x)$ as $N(m(x;\phi), V(x;\phi))$ for some neural networks $m,V$ for the conditional mean and variance. 
- **Decoder**, $z \to x$: The latent variable $z$ is $N(0,I)$ distributed, and $x \mid z$ is $N(\mu(z;\theta), \Sigma(z;\theta))$. The $\mu, \Sigma$ functions are neural networks, so this is a rich class of distributions which could adequately model, e.g., complex high-dimensional image data.

This structure is well-suited to the ELBO approach. In general the marginal density $p_\theta(x) = \int p_\theta(x,z) \\ dz$ is expensive to evaluate accurately, but the joint density $p_\theta(x,z)$ is easy to evaluate, since it is the product of the normal densities $p_\theta(x \mid z)$, $p(z)$. Because $z$ is $N(0,I)$, it is not parameterized by $\theta$.

Because of the latent variable structure we can easily generate new samples from the model we fit—simply draw $z$ from $N(0,I)$, and then $x \mid z$ from $N(\mu(z;\theta), \Sigma(z;\theta))$. 

#### Diffusion Models
Unlike VAEs, the encoding process has no unknown parameters, and involves iteratively adding noise with known variance to the input data. This means that $q_\phi ( z \mid x)$ can be treated as fixed throughout, with $\phi$ empty. The goal is instead to learn the decoder, which turns $z$ into $x$. In more detail:

- **Encoder**, $x \to z$: the latent variable $z$ is a _sequence_ $z_1,\ldots,z_T$, which adds progressively more noise to the starting data $x$. We start with $z_1 = \sqrt{1 - \beta_1} x + \sqrt{\beta_1} \varepsilon_1$ and continue iterating $z_t = \sqrt{1 - \beta_t} z_{t - 1} + \sqrt{\beta_t} \varepsilon_t$, for $t>1$ and a known sequence $\beta_1,\ldots,\beta_T$. The noise terms $\varepsilon_t$ are independent $N(0,I)$. As $T \to \infty$, all this added noise means $z_T$'s distribution becomes close to $N(0,I)$. 

- **Decoder**, $z \to x$: we model $p_\theta(z_{t-1} \mid z_t)$ as normally distributed. Then we rewrite $p_\theta(x, z)$ in the ELBO as $p(z_T) \left[ \prod_{t = 2}^T p_\theta(z_{t-1} \mid z_t) \right] p_\theta(x \mid z_1)$, and maximize the ELBO with respect to $\theta$.

As with VAEs, generating new samples is easy once the decoder is learned, as we have the distributions of $z$ and $x \mid z$.

### Statistical Physics
#### Variational Statistical Mechanics
[Some physical systems](https://en.wikipedia.org/wiki/Canonical_ensemble) are well described by the [Boltzmann distribution](https://en.wikipedia.org/wiki/Boltzmann_distribution), which specifies the probability of a state $z_i$ for $i \in \\{1,\ldots,M\\}$ as

$$
p_i=\frac{1}{Z} \exp\left(-  \varepsilon_i \right)  = \frac{ \exp\left(-  \varepsilon_i \right) }{ \displaystyle \sum_{j=1}^{M} \exp\left(- \varepsilon_j \right) }.
$$

The normalizing constant $Z$ is known as the "partition function", and $\varepsilon_i$ is the energy of state $z_i$.[^4] Energy in any given state $z_i$ is a known system-specific function $E$, called the Hamiltonian: $\varepsilon_i = E(z_i)$. The partition function plays a critical role in determining the physical behavior of the system, but may take impractically long to compute if the number of system states $M$ is enormous.

This is where ELBO comes in. We write the log joint "density" of the ELBO as $\log p(x,z) = -E(z)$. Note for this application there are no $x$'s. Then the "marginal likelihood", obtained from summing over states $z_i$, is exactly the partition function! With a tractable model $q(z)$ of the distribution of system states, we can find a lower bound on the log partition function by maximizing the ELBO. The optimization will often involve variational methods. The negative ELBO is the called the variational free energy, and the negative log partition function is called the Helmholtz free energy.  As in the [variational Bayes](#variational-bayes) case above, if $z$ is vector-valued, a common approach is to make $q$ tractable via the _mean-field_ assumption, according to which the component elements are independent. 

See [here](https://ml4physicalsciences.github.io/2019/files/NeurIPS_ML4PS_2019_92.pdf) for more on the equivalence between the ELBO and variational methods in physics.

[^4]: Implicitly the [thermodynamic beta](https://en.wikipedia.org/wiki/Thermodynamic_beta) is set to one here—we can think of this as rescaling energies so that $\beta = 1$.

### Computational Biology
#### Modeling Single-Cell Gene Expression
Variational methods appear to be becoming more common in biology. One nice application is ["Deep generative modeling for single-cell transcriptomics"](https://www.nature.com/articles/s41592-018-0229-2). Here the data $x$ describes, for each cell, the degree to which it expresses one of many genes. The $z$ is the latent structure, which is much lower-dimensional than the number of genes: a cell is a 10-dimensional vector in latent space, but expresses 100s - 10,000s genes in these datasets. The modeling is similar to the variational autoencoder network example above, except the likelihood $p_\theta(x \mid z)$ is zero-inflated negative binomial, to fit the count data. Learning the latent structure allows us cluster or classify cells. Especially intriguing is cell _generation_—we can use this model to generate new synthetic cells, just as a VAE trained on image data allows us to generate new images! 

### Neuroscience 
#### Free-Energy Model of the Brain
An influential theory in neuroscience is Karl Friston's [Free-Energy Principle](https://www.nature.com/articles/nrn2787). It exemplifies the Bayesian brain hypothesis, which posits that the brain has a built-in generative model of the world which it uses to construct posterior beliefs. Friston's theory has the brain effectively performing variational inference to understand the world.

Here the latent variables $z$ are physical events and features in the world, which then generate sensory input $x$  (e.g. light on the retina or cochlear vibration). The brain is assumed to have some understanding of the kinds of $z$ out in the world, and how they map to $x$—in other words, a generative model $p(z)$ and $p_\theta(x \mid z)$. We wish to form an accurate understanding of what's going on in the physical world given the sense data, i.e. an approximate posterior $q_\phi(z \mid x)$, called the "recognition density" here. The parameters $(\theta, \phi)$ correspond to synaptic strengths in the brain. 

We have all the elements necessary to apply variational inference. Instead of maximizing the ELBO, we talk of "minimizing free energy" or "minimizing surprise", although this is just a point of terminology—the mathematics is the same.

