---
jupytext:
  text_representation:
    extension: .myst
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.1
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# What is the theory behind PEC?


Probabilistic error cancellation (PEC) {cite}`Temme_2017_PRL, Sun_2021_PRAppl, Shuaining_2020_NatComm`  is a noise-aware error mitigation technique which is
based on two main ideas:

- The first idea is to express ideal gates as linear combinations of implementable noisy gates.
These linear combinations are called quasi-probability representations {cite}`Pashayan_2015_PRL`;

- The second idea is to probabilistically sample from the previous quasi-probability representations to approximate
quantum expectation values via a Monte Carlo average.

**Note:* In this section we follow the same notation of {cite}`Mari_2021_PRA`.*

## Quasi-probability representations

In PEC, each ideal gate $\mathcal G_i$ of a circuit of interest
$\mathcal U = {\mathcal G}_t \circ  \dots \circ {\mathcal G}_2 \circ {\mathcal G}_1 $
is represented as a linear combination of noisy implementable operations ${\mathcal O_{i, \alpha}}$ (i.e., operations that
can be directly applied with a noisy backend):

$$
\mathcal G_i = \sum_\alpha \eta_{i, \alpha} \mathcal O_{i, \alpha},
\quad  \eta_{i, \alpha} \in \mathbb R,
$$

where the calligraphic symbols ($\mathcal U$, $\mathcal G_i$, $\mathcal O_{i, \alpha}$) stand for super-operators acting
on the density matrix of the qubits as linear quantum channels.

The real coefficients ${\eta_{i,\alpha}}$ form a quasi-probability distribution {cite}`Pashayan_2015_PRL` with respect to the
index $\alpha$. Their sum is normalized but, differently from standard probabilities, they can take negative values:

 $$ \sum_\alpha \eta_{i,\alpha}=1,  \qquad  \gamma_i = \sum_\alpha |\eta_{i, \alpha}| \ge 1.$$

The constant $\gamma_i$ quantifies the negativity of the quasi-probability distribution which is directly related
to the error mitigation cost associated to the gate $\mathcal G_i$.

***Note:** In principle, the gate index "$i$" in the noisy operations $\mathcal O_{i, \alpha}$ could be dropped.
However, we keep it to explicitly define gate-dependent basis of implementable operations, consistently with
the  structure of the `OperationRepresentation` class discussed in [What additional options are
available in PEC?](pec-3-options.md).*


## Error cancellation

The aim of PEC is estimating the ideal expectation value of some observable $A=A^\dagger$ with respect to
the quantum state prepared by an ideal circuit of interest $\mathcal U$ acting on some initial  reference
state $\rho_0$ (typically $\rho_0= |0\dots 0 \rangle \langle 0 \dots 0 |$).

Replacing each gate $\mathcal G_i$ with its noisy representation, we can express the ideal expectation
value as a linear combination of noisy expectation values:

$$
\langle A \rangle_{\rm ideal}= {\rm tr}[A \mathcal U (\rho_0)] =
\sum_{\vec{\alpha}} \eta_{\vec{\alpha}} \langle A_{\vec{\alpha}}\rangle_{\rm noisy}
$$

where we introduced the multi-index $\vec{\alpha}=(\alpha_1, \alpha_2, \dots ,\alpha_t)$ and

$$
\eta_{\vec{\alpha}} := \prod_{i=1}^t \eta_{i, \alpha_i},
\quad  \langle A_{\vec{\alpha}}\rangle_{\rm noisy} :=  {\rm tr}[A \Phi_{\vec{\alpha}}(\rho_0)],
\quad \Phi_{\vec{\alpha}} := \mathcal O_{t, \alpha_t} \circ \dots \circ \mathcal O_{2, \alpha_2} \circ \mathcal O_{1, \alpha_1}.
$$


The coefficients $\{ \eta_{\vec{\alpha}} \}$ form a quasi-probability distribution
for the global circuit over the noisy circuits. Indeed it is easy to check that, at the level of super-operators,
we have:

$$ \mathcal U =  \sum_{\vec{\alpha}} \eta_{\vec{\alpha}} \Phi_{\vec{\alpha}}. $$

The one-norm $\gamma$ of the global quasi-probability distribution is the product of those of the gates:

$$
\sum_{\vec \alpha} \eta_{\vec{\alpha}}=1,  \qquad  \gamma = \sum_{\vec{\alpha}} |\eta_{\vec \alpha}| = \prod_{i=1}^{t} \gamma_i.
$$

All the noisy expectation values $\langle A_{\vec{\alpha}}\rangle_{\rm noisy}$ can be directly measured with
a noisy backend since they only require circuits composed of implementable noisy operations.
In principle, by combining all the noisy expectation values, one could compute the ideal result $\langle A \rangle_{\rm ideal}$.
Unfortunately this approach requires executing a number of circuits which grows exponentially with the circuit depth and
which is typically unfeasible.

An important fact at the basis of PEC is that, for weak noise, only a small number of noisy expectation values actually
contribute to the linear combination because many of the coefficients $\eta_{\vec \alpha}$ are negligible.
For this reason, it is more efficient to estimate $\langle A \rangle_{\rm ideal}$ using an
 [importance-sampling](https://en.wikipedia.org/wiki/Importance_sampling) Monte Carlo approach as described in the next section.

## Monte Carlo estimation

To apply a Monte Carlo estimation, we need to replace quasi-probabilities with positive probabilities.
This can be achieved as follows:

$$ \mathcal{G_i} = \sum_{\alpha} \eta_{i, \alpha} \mathcal{O}_{i, \alpha}
= \gamma_i \sum_{\alpha} p_i(\alpha) \, {\rm sgn}(\eta_{i, \alpha})\, \mathcal{O}_{i, \alpha},$$

where $p_{i}(\alpha)=|\eta_{i, \alpha}|/\gamma_i$ is a valid probability distribution with respect to $\alpha$.

If for each gate $\mathcal G_i$ of the circuit we sample a value of $\alpha$ from $p_{i}(\alpha)$ and we apply the corresponding noisy operation
$\mathcal O_{i, \alpha}$, we are effectively sampling a noisy circuit $\Phi_{\vec{\alpha}}$ from the
global probability distribution $p(\vec{\alpha})= |\eta_{\vec{\alpha}}| / \gamma$.

Therefore, at the level of quantum channels, we have:

$$ \mathcal U = \gamma \mathbb E \left\{  {\rm sgn}(\eta_{i, \vec{\alpha}}) \Phi_{\vec{\alpha}} \right\},$$

where $\mathbb E$ is the sample average over many repetitions of the previous probabilistic procedure and
${\rm sgn}(\eta_{\vec{ \alpha}}) = \prod_i {\rm sgn}(\eta_{i, \alpha})$.
As a direct consequence, we can express the ideal expectation value as follows:

$$\langle A \rangle_{\text{ideal}} = \gamma\,
\mathbb E \left\{  {\rm sgn}(\eta_{\vec{\alpha}}) \langle A_{\vec{\alpha}}\rangle_{\rm noisy} \right\}.$$

By averaging a finite number $N$ of samples we obtain an unbiased estimate of $\langle A \rangle_{\text{ideal}}$.
Assuming a bounded observable $|A|\le 1$, the number of samples $N$
necessary to approximate $\langle A\rangle_{\text{ideal}}$ within an absolute error $\delta$,
scales as {cite}`Takagi_2020_PRR`:

$$ N \propto \frac{\gamma^2}{\delta^2}. $$

The term $\delta^2$ in the denominator is due to the stochastic nature of quantum measurements
and is present even when directly estimating an expectation value without error mitigation.
The $\gamma^2$ factor instead represents the sampling overhead associated to PEC.
For weak noise and short circuits, $\gamma$ is typically small and PEC is applicable with a reasonable
cost.
On the contrary, if a circuit is too noisy or too deep, the value of $\gamma$ can be so large that PEC becomes
unfeasible.
