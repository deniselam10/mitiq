---
jupytext:
  text_representation:
    extension: .myst
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.10.3
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# When should I use ZNE?

## Advantages

Zero noise extrapolation is one of the simplest error mitigation techniques and, in many practical situations, it can be applied with a relatively small sampling cost.
The main advantage of ZNE is that the technique can be applied without a detailed knowledge of
the undelying noise model. Therefore it can be a good option in situations where
tomography is impractical.


## Disadvantages

In some instances the results of the extrapolation can exhibit a large bias
{cite}`Mari_2021_PRA`. ZNE may not be helpful in cases where a low degree
polynomial curve obtained by fitting the noisy expectation values does not match the
zero-noise limit. When using circuits of less trivial depth on real devices, the
lowest error points may be too noisy for the extrapolation to show improvement over
the unmitigated result {cite}`Lowe_2021_PRR`.

## Example

For a simple example in which the application of ZNE reduces the estimation error when
compared to the unmitigated result, see:

[Zero Noise Extrapolation for mitigating errors in energy landscape of a two-qubit
variational circuit](https://mitiq.readthedocs.io/en/latest/examples/simple-landscape-cirq.html)
