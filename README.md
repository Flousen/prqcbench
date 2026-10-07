<p align="center"><img src="logo.svg" alt="PRQC Bench logo" width="360"></p>

# PRQC Benchmark Circuits

The Peaked Random Quantum Circuits (PRQCs) used in

> M. Brieger, F. Krötz, M. Chung, D. Kranzlmüller, *Evaluating System-Level Fidelity with Peaked Random Circuits*, [arXiv:2605.25983](https://arxiv.org/abs/2605.25983) (2026).

[![arXiv](https://img.shields.io/badge/arXiv-2605.25983-b31b1b.svg)](https://arxiv.org/abs/2605.25983)

A PRQC is a mirrored brick-wall circuit $P(\theta)^\dagger R$ of two-qubit gates. $R$ is random, and $P(\theta)$ is optimised so that measuring the circuit on $|0^n\rangle$ gives the all-zero bitstring with high probability.

## Layout

```
circuits/
├── metrics.csv          # one row per circuit
├── q2/
│   ├── prqc_q2_d2.qasm
│   ├── ...
│   └── prqc_q2_d50.qasm
├── ...
└── q20/
generate_prqc.ipynb      # how the circuits are generated
requirements.txt
```

`circuits/` holds all 931 circuits from the paper, for every combination of qubits n = 2–20 and depth d = 2–50.
`circuits/q{n}/prqc_q{n}_d{d}.qasm` is the circuit with `n` qubits and depth `d`. It is written in OpenQASM 3 with the gate set `U` + `cz` and contains no measurements.
Every circuit peaks on the all-zero bitstring `0…0`. Line 2 of each file gives this peak bitstring and its ideal probability, for example:

```
OPENQASM 3.0;
// peak bitstring: 00000000000000000000  (ideal probability 0.2414)
```

`circuits/metrics.csv` has these columns:

| column | meaning |
|---|---|
| `qubits`, `depth` | n and d |
| `peak_bitstring` | the target bitstring (always `0…0`) |
| `rand_layers`, `peak_layers` | layers in the random half R and in the peaking half P (for odd d, P has one extra layer) |
| `peak_probability` | ideal probability of measuring `0…0` |
| `n_u3`, `n_cz` | gate counts of the QASM circuit |
| `file` | path of the QASM file |

## Usage

```python
from qiskit import qasm3

qc = qasm3.load("circuits/q10/prqc_q10_d20.qasm")
qc.measure_all()
# run qc on a device; a successful run returns "0000000000" as the most frequent outcome
```

## Generating circuits

[`generate_prqc.ipynb`](generate_prqc.ipynb) generates a single PRQC for a chosen `n` and `d`:

1. build R and P as brick-walls of Haar-random two-qubit unitaries (quimb tensor networks);
2. maximise $|\langle 0^n|P(\theta)^\dagger R|0^n\rangle|^2$ with JAX (5,000 L-BFGS-B iterations, then up to 10,000 Adam iterations);
3. transpile to `u` + `cz` and export as OpenQASM 3.

Optimisation uses random initial gates, so a new run will not reproduce the published circuits exactly. It should reach a similar peak probability.

```bash
pip install -r requirements.txt
jupyter notebook generate_prqc.ipynb
```

## References

The construction follows Aaronson and Zhang [1, 2]. The circuit-generation code builds on the reference implementation released with [1]: [yuxuanzhang1995/Peaked-circuits](https://github.com/yuxuanzhang1995/Peaked-circuits).

1. S. Aaronson and Y. Zhang, "On verifiable quantum advantage with peaked circuit sampling," arXiv:2404.14493, 2024. https://doi.org/10.48550/arXiv.2404.14493
2. Y. Zhang, "Complexity and hardness of random peaked circuits," arXiv:2510.00132, 2025. https://doi.org/10.48550/arXiv.2510.00132

## Citation

If you use these circuits, please cite:

```bibtex
@misc{brieger2026prqc,
  title         = {Evaluating System-Level Fidelity with Peaked Random Circuits},
  author        = {Brieger, Martin and Kr{\"o}tz, Florian and Chung, Minh and Kranzlm{\"u}ller, Dieter},
  year          = {2026},
  eprint        = {2605.25983},
  archivePrefix = {arXiv},
  primaryClass  = {quant-ph},
  doi           = {10.48550/arXiv.2605.25983},
  url           = {https://arxiv.org/abs/2605.25983}
}
```
