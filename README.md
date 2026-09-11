# Quantum-hardware data

This repository contains the quantum-circuit submission script and the archived measurement data associated with Fig. 3 of the manuscript *Quantum synchronization and chimera states in a programmable quantum many-body system*.

Figure 3 examines quantum chimera dynamics on a 156-qubit heavy-hex lattice using the IBM Quantum processor `ibm_kobe`. The experiment measures the local transverse magnetization at successive stroboscopic times and uses the resulting time series to characterize synchronized and desynchronized regions.

## Repository contents

| Path | Description |
| --- | --- |
| [`code/run_circuit_fig3.py`](code/run_circuit_fig3.py) | Constructs, transpiles, and submits the 156-qubit circuits. It saves backend information, shot count, final-circuit gate counts, and IBM Runtime job IDs. |
| [`result/fig3.json`](result/fig3.json) | Shot-derived local-magnetization estimates and uncertainties for 41 stroboscopic times and 156 qubits. |

## Data format

`result/fig3.json` contains three arrays, each with shape `41 x 156`:

| Key | Meaning |
| --- | --- |
| `mean_values` | Local $X$-basis expectation estimates. Element `[n][j]` corresponds to stroboscopic step $n=0,\ldots,40$ and qubit $j=0,\ldots,155$. |
| `standard_deviations` | Single-shot standard deviations, $\sqrt{1-\langle X_j\rangle^2}$. |
| `standard_errors` | Standard errors of the means, equal to `standard_deviations / sqrt(N_shots)`. |

The JSON file contains finite-shot measurement results. The normalization-based error mitigation, synchronization analysis, and plotting steps described in the manuscript are not implemented in the files currently included in this repository.

## Circuit configuration

The checked-in script uses the following configuration:

- Number of qubits: 156
- Heavy-hex couplings: 176
- Parallel coupling layers: 59, 59, and 58 edges
- Interaction angles: $\theta_{XX}=\theta_{YY}=\theta_{ZZ}=-0.25\pi$
- Longitudinal-field angle: $\theta_Z=0.25\pi$
- Stroboscopic steps: $n=0,\ldots,40$
- Shots per circuit: $N_{shots}=2^{13}=8192$
- Initial-phase random seed: `19483`
- Initial phases in the checked-in script: sampled uniformly from $[0,\pi]$
- Transpiler optimization level: 0
- Measurement basis: local $X$ basis

The 176 heavy-hex couplings are unique, cover all 156 qubits, form one connected graph, and are partitioned into three conflict-free layers.

## Requirements

The circuit-submission script requires Python and the following packages:

```text
numpy
qiskit
qiskit-ibm-runtime
```

IBM Quantum Runtime interfaces can change between releases. A version-pinned `requirements.txt` or environment file is recommended for software-level reproducibility.

## Running the hardware experiment

The target backend is read from the `QISKIT_BACKEND_NAME` environment variable:

```bash
export QISKIT_BACKEND_NAME=ibm_kobe
python code/run_circuit_fig3.py
```

The script writes an execution-metadata JSON file to the current working directory. It does not download job results or generate `result/fig3.json`; those steps require a separate retrieval and processing workflow.

## Important consistency check

The manuscript describes the Fig. 3 initial state using strong phase randomness, $\phi^{\max}=2\pi$.

To make the submitted script correspond to the stated Fig. 3 protocol, verify the provenance of `result/fig3.json` and change this value to `2.0 * pi` if the archived data were obtained with $\phi^{\max}=2\pi$.

The manuscript also states that the Fig. 3 magnetizations are error mitigated by normalization to a reference circuit. The reference data and the code that performs this normalization and propagates its uncertainty should be archived if the repository is intended to reproduce the final plotted values.

## Reproducibility note

Re-executing the circuits verifies the experimental procedure but will not reproduce the archived numerical values exactly because device noise and calibration vary with time. The archived data should therefore be used to reproduce the reported analysis, whereas a new hardware run constitutes an independent realization of the experiment.

## Citation

If you use this code or data, please cite the associated manuscript:

> K. Shinjo *et al.*, “Quantum synchronization and chimera states in a programmable quantum many-body system.”
