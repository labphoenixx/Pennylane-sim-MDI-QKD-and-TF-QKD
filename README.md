# Noise Tolerant Limits for Higher-Dimensional MDI-QKD and TF-QKD Protocols

Simulation code for the paper "Noise-Tolerant Limits for Higher-Dimensional MDI-QKD and TF-QKD Protocols" (S. Dharshana and T. S. L. Radhika, 2026 International Conference on Intelligent Computing and Secure Communication Systems (ICICSCS)).

The notebooks simulate two protocols of Quantum Key distribution (QKD) : Measurement Device Independent QKD (MDI-QKD) and Twin Field QKD (TF-QKD) with qudits of dimension **d = 2 to 7** under realistic channel noise in [PennyLane](https://pennylane.ai) They measure how the noise tolerance, secret key rate and ability to detect an eavesdropper (Eve) change with the encoding dimension.

---

## Key findings

- **Noise tolerance rises with dimension, with diminishing returns.** The MDI-QKD zero-rate error threshold e\*(d) goes from 11.00% at d = 2 to 23.72% at d = 7, but the gain per added dimension shrinks from 4.94 to 1.20 percentage points.
- **The rate-optimal dimension depends on the noise level.** Because of the 1/d Bell-measurement penalty, the best MDI-QKD dimension _rises_ with channel noise. TF-QKD has no such penalty, so its best dimension _falls_ as thermal noise increases.
- **d = 6 is rate-optimal only within a narrow noise band.** In MDI-QKD, d = 6 gives the highest key rate for an operating error rate of **9.87%–12.19%**. In TF-QKD it is optimal for thermal noise n̄ ≈ 0.13–0.16, which corresponds to an observed QSER of **about 7.1%–8.3%**.
- **An eavesdropper is always detectable.** Intercept-resend errors stay well above e\*(d) for every d tested. The detection margin peaks at **d = 6** in MDI-QKD and stops growing around d = 6 in TF-QKD.

| d   | e\*(d) (%) | Δe\* (pp) | MDI crossover Q\*(d→d+1) (%) | Q_Eve (%) | Detection margin (pp) |
| --- | ---------- | --------- | ---------------------------- | --------- | --------------------- |
| 2   | 11.00      | –         | –                            | 25.00     | 14.00                 |
| 3   | 15.95      | 4.94      | 2.96                         | 33.33     | 17.39                 |
| 4   | 18.93      | 2.98      | 6.88                         | 37.50     | 18.57                 |
| 5   | 20.99      | 2.06      | 9.87                         | 40.00     | 19.01                 |
| 6   | 22.52      | 1.53      | 12.19                        | 41.67     | 19.15                 |
| 7   | 23.72      | 1.20      | 14.03                        | 42.86     | 19.13                 |

_MDI-QKD values. The simulation covers d = 2–7; the notebook also evaluates d = 8, 9 from the same equations._

---

## Repo contents

```
.
├── mdi_qkd_d2_to_d7.ipynb    # MDI-QKD, d = 2–7, depolarizing channel noise
├── tf_qkd_d2_to_d7.ipynb     # TF-QKD (sending-or-not-sending), d = 2–7, thermal loss + phase drift
├── requirements.txt
└── README.md
```

---

### `mdi_qkd_d2_to_d7.ipynb`

| Section | Content                                                                                                                          |
| ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| 0       | Shared formulas: entropies, key rates, thresholds, Fourier basis, generalised Bell states, entanglement witness                  |
| 1       | Concept figures: MUB overlap heatmaps, Fourier phases, Bell-basis Gram matrices, BSM outcome maps, dual-MUB entanglement witness |
| 2       | d = 2 (qubit): BB84 states, l = 0 Bell measurement, intercept-resend, noise sweep                                                |
| 3       | d = 3 (qutrit): padded two-qubit encoding, Sheridan–Scarani key rate                                                             |
| 4       | d = 4 (ququart): gate-level two-qubit encoding, Fourier states from H/Z/S gates                                                  |
| 5       | d = 5, 6, 7: padded three-qubit registers, idealised d-ary channel vs physical depolarizing channel, leakage into padding states |
| 6       | Cross-dimension analysis: total key rate, crossovers, threshold increments, rate-optimal dimension, detection margin             |

### `tf_qkd_d2_to_d7.ipynb`

| Section | Content                                                                                                         |
| ------- | --------------------------------------------------------------------------------------------------------------- |
| 1–2     | Parameters; Gaussian thermal-loss channel on `default.gaussian` with transfer-matrix validation and heatmaps    |
| 3       | Analytical QSER from photon-number discrimination; key rate and thresholds; photon-statistics and QSER heatmaps |
| 4–6     | Monte Carlo engine, thermal and phase-drift sweeps (20 trials × 150,000 rounds per point), result tables        |
| 7       | Figures: QSER and key rate vs thermal noise and phase drift, intercept-resend, diminishing returns              |
| 8       | Rate-optimal dimension and security margin (analytical, d = 3–12)                                               |

---

## Method

**MDI-QKD.** Alice and Bob prepare qudits in the computational or discrete-Fourier basis, a pair of mutually unbiased bases. Noise is applied with `qml.DepolarizingChannel` on the physical qubits after preparation, on PennyLane's `default.mixed` density-matrix device. The untrusted relay Charlie performs a generalised Bell-state measurement that resolves the d Bell states with l = 0, so it succeeds with probability 1/d. Bob corrects his symbol using Charlie's announced shift m: b_corr = (b − m) mod d.

- Key rate per sifted symbol: r_d(Q) = log₂d − 2H_d(Q)
- Total key rate per signal: R = (1/2)(1/d) r_d(Q)

**TF-QKD.** This is a qudit generalisation of sending-or-not-sending. In each sending round a weak coherent pulse goes into one of d − 1 time-bin modes. The channel combines thermal loss (a thermal mode mixed in at a beamsplitter) with random phase drift. A symbol error occurs when one of the d − 2 thermally populated competing modes registers at least as many photons as the signal mode.

- The total rate is R = p_click · r_d(Q), with no 1/d penalty.

**Attack model.** Intercept-resend attack on Alice's channel.

---

## Getting started

```bash
git clone https://github.com/labphoenixx/Pennylane-sim-MDI-QKD-and-TF-QKD.git
cd Pennylane-sim-MDI-QKD-and-TF-QKD
pip install -r requirements.txt
jupyter notebook
```

All random seeds are fixed, so the outputs are reproducible. To trade accuracy for speed, adjust the round counts at the top of each section (`N_D2`, `N_D3`, `N_D4`, `N_QUDIT`, `N_CHANNEL`, `N_ROUNDS`).

---

## Limitations

- The results are simulation-based and use asymptotic key rates. Finite-key effects, error-correction inefficiency and detector imperfections are not modelled.
- Security is evaluated against intercept-resend only. The key-rate expression assumes collective attacks.
- The qudit TF-QKD model is a modelling construction. Strict twin-field QKD has been shown not to generalise to higher dimensions. The model has no error mechanism at d = 2, so d = 2 is excluded from TF-QKD key-rate comparisons.

---

## Paper

> S. Dharshana and T. S. L. Radhika, "Noise-Tolerant Limits for Higher-Dimensional MDI-QKD and TF-QKD
> Protocols," presented at the _2026 Int. Conf. Intelligent Computing and Secure Communication Systems
> (ICICSCS)_, Oct. 2026. (To appear in IEEE Xplore.)

### Citation

```bibtex
@inproceedings{dharshana2026noise,
  author    = {Dharshana, S. and Radhika, T. S. L.},
  title     = {Noise-Tolerant Limits for Higher-Dimensional {MDI-QKD} and {TF-QKD} Protocols},
  booktitle = {Proc. 2026 Int. Conf. Intelligent Computing and Secure Communication Systems (ICICSCS)},
  year      = {2026},
  month     = oct,
  note      = {To appear}
}
```

---

## Authors

- **S. Dharshana**, Dept. of Physics and Computer Science Engineering, BITS Pilani, Hyderabad Campus · sdharshana26@gmail.com or · f20231208@hyderabad.bits-pilani.ac.in
- **T. S. L. Radhika**, Associate Professor, Dept. of Mathematics, BITS Pilani, Hyderabad Campus · radhikatsl@hyderabad.bits-pilani.ac.in

## License

Code released under the MIT License (see `LICENSE`). The paper is © 2026 IEEE.
