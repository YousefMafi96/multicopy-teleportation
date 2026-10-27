# Multicopy teleportation and quantum program retrieval — simulation code

Simulation code accompanying the paper

> Y. Mafi, T. Ojanen, and A. G. Moghaddam,
> *Multicopy teleportation and quantum program retrieval on superconducting quantum hardware* (2026).

The protocol is the multicopy, correction-free teleportation of
F. Grosshans *et al.*, *Quantum* **10**, 2105 (2026): with $k$ copies of an unknown qubit state and a
single Bell pair, one collective measurement transfers the state exactly, without any correction at
the receiver, with success probability

$$p(d,k)=\frac{k}{d(k-1+d)}\qquad(\tfrac14,\ \tfrac13,\ \tfrac38,\ \tfrac25,\ \tfrac5{12}\ \text{for}\ d=2,\ k=1\dots5).$$

The same measurement retrieves a quantum program $\hat U$ that was stored earlier in the Bell pair.

The paper's results were measured on `ibm_kingston`. This repository reproduces every figure with
**Qiskit Aer** only: the ideal `AerSimulator` and the noisy `FakeKingston` snapshot of the same
processor. No IBM Quantum account or hardware access is needed.

## Notebooks

| Notebook | Paper section | Output | Runtime* |
|---|---|---|---|
| `00_core.ipynb` | – | shared settings and functions, loaded by all other notebooks | – |
| `01_protocol_and_implementation.ipynb` | Protocol and its implementation (Eqs. 1–3) | exactness checks, circuits for $k=2$ | < 1 min |
| `02_multicopy_teleportation.ipynb` | Multicopy teleportation | **Fig. 2** (a) $m=1$, (b) $m=2$ programs | ~ 6 min |
| `03_program_storage_retrieval.ipynb` | Quantum program storage and retrieval | **Fig. 3** | ~ 2 min |
| `04_memory_lifetime.ipynb` | Memory lifetime, Eq. (4) | **Fig. 4**, fit of $\tau_c$, DD comparison | ~ 10 min |
| `05_circuit_complexity.ipynb` | Discussion (circuit overhead) | gate counts, depth, duration, routing overhead | < 1 min |

\* single CPU core, 10 000 shots per circuit. Reduce `SHOTS` in `00_core.ipynb` for quick tests.

Each section notebook starts with `%run ./00_core.ipynb`, so the notebooks are independent of each
other and can be run in any order.

## Installation

```bash
git clone <this-repository>
cd <this-repository>
pip install -r requirements.txt
jupyter lab
```

On Google Colab, upload all notebooks to the same folder (or clone the repository), run
`!pip install -r requirements.txt`, and restart the runtime once.

## What the code does

**Measurement.** The vectors $\lvert r_i\rangle$ (Eq. 2) are built from Dicke states and a Bell pair,
and completed to a unitary $\mathcal W$ by a full QR decomposition (Eq. 3). After $\mathcal W$, an
outcome $<R=\binom{k-2+d}{k-1}$ on Alice's $k+1$ qubits heralds success.

**Two decompositions of $\mathcal W$.**

* `qsd` – the full unitary is synthesised by the quantum Shannon decomposition, $\Theta(4^k)$ two-qubit gates.
* `isometry` – only the physical columns $\lvert r_i\rangle$ are synthesised, $O(k\,2^k)$ two-qubit gates.

**Protocol.** Bell pair → stored program on Bob → optional storage delay → $k$ input copies →
measurement → herald. Teleportation is the case $\hat U=\mathbb 1$. For two-qubit programs ($m=2$) the
input is a product state, so the measurement factorises into one block per qubit and the ideal success
probability is $p(2,k)^2$.

**Figures of merit.**

* success probability: fraction of shots with a successful herald;
* fidelity ($m=1$): Bob's state is reconstructed by $X$, $Y$, $Z$ tomography on the heralded shots,
  $\mathcal F=\langle t\rvert\rho\lvert t\rangle$ with target $\lvert t\rangle=\hat U\lvert\psi\rangle$;
* fidelity ($m=2$): Bob undoes $\hat U$ and the probability of reading back the input is recorded;
* error bars: one standard deviation from shot noise.

**Layouts.** Noisy runs use fixed, nested layouts on `FakeKingston` (`LAYOUT_1`, `LAYOUT_2` in
`00_core.ipynb`): increasing $k$ adds one copy qubit and keeps all other qubits fixed.

## Simulation versus hardware

The noisy simulator reproduces the trends of the paper, but not every number, because

* the noise model contains depolarising gate errors, readout errors and $T_1/T_2$ relaxation, but no
  crosstalk, leakage or low-frequency noise;
* readout errors are **not** mitigated here (the hardware data used M3), so simulated fidelities include
  readout error;
* relaxation is applied to explicit delays (the storage delay of Fig. 4). Idle windows inside a circuit
  are only charged when `schedule="pad"` or `schedule="dd"` is passed to the run functions;
* the relaxation model is Markovian, so XY4 dynamical decoupling gives no gain in simulation
  (`04_memory_lifetime.ipynb` shows this explicitly), whereas it did on hardware;
* `FakeKingston` is a calibration snapshot and differs from the device on the day of the experiment.

## Reproducibility

Transpilation uses `seed_transpiler=SEED`, and every simulator run draws its seed from a generator
initialised with `SEED` (both set in `00_core.ipynb`). Running a notebook from top to bottom therefore
gives identical results.

## Citation

If you use this code, please cite the paper above and the theory proposal:

```bibtex
@article{Grosshans2026multicopy,
  author  = {Grosshans, Fr{\'e}d{\'e}ric and Horodecki, Micha{\l} and Murao, Mio and M{\l}ynik, Tomasz
             and Quintino, Marco T{\'u}lio and Studzi{\'n}ski, Micha{\l} and Yoshida, Satoshi},
  title   = {Multicopy quantum state teleportation with application to storage and retrieval of quantum programs},
  journal = {Quantum},
  volume  = {10},
  pages   = {2105},
  year    = {2026}
}
```
