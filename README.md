# t-center-zpl
# ⚛️ ML-Accelerated Prediction of Optical Lineshape & ZPL Fraction
### SRIC Look Within Internship | IIT Kharagpur | Prof. Sabyashachi Mishra

![Python](https://img.shields.io/badge/Python-3.11-blue?style=flat-square&logo=python)
![MACE](https://img.shields.io/badge/MACE-MP--0-purple?style=flat-square)
![Phonopy](https://img.shields.io/badge/Phonopy-2.x-orange?style=flat-square)
![ASE](https://img.shields.io/badge/ASE-3.23-blue?style=flat-square)
![QE](https://img.shields.io/badge/Quantum_ESPRESSO-7.2-darkblue?style=flat-square)

> A two-stage computational pipeline combining MACE machine-learning interatomic potentials and Quantum ESPRESSO DFT to predict the zero-phonon line fraction of T-center silicon quantum emitters — achieving 300x cost reduction over conventional DFT while outperforming published HSE06 results from ACS Nano 2026.

---

## 📌 Research Context

The **T-center** — a carbon-hydrogen co-defect (Cs–H) in silicon — is one of the most promising solid-state quantum emitters for quantum networking, operating at 1326 nm in the telecom O-band. Its key figure of merit is the **Zero-Phonon Line (ZPL) fraction**: the proportion of photons emitted without phonon loss, determined by the Huang-Rhys factor S via ZPL = e^(-S).

**The challenge:** Computing S from first principles requires the full phonon spectrum of a ~500-atom defect supercell — prohibitively expensive with conventional DFT (HSE06 level requires ~32 days of CPU time).

**This work:** A MACE ML potential replaces DFT force evaluations (4 minutes vs 18 hours), validated against experimental data and outperforming the published HSE06 benchmark from ACS Nano 2026.

---

## 🏗️ Two-Stage Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    STAGE 1: MACE Pipeline                   │
│                                                             │
│  T-Center Structure (55-atom supercell)                    │
│         │                                                   │
│         ▼                                                   │
│  MACE Foundation Model (mace-mp-0)                         │
│  → Force evaluations: 74 configurations in ~4 minutes      │
│         │                                                   │
│         ▼                                                   │
│  Phonopy 2.x                                               │
│  → Dynamical matrix → Phonon eigenvectors                  │
│  → Phonon DOS → C-Si local mode at 68.9 meV               │
│         │                                                   │
│         ▼                                                   │
│  Huang-Rhys Factor Computation                             │
│  S_i = (ω_i / 2ℏ) × (Δq_i)²                              │
│  S_total = 1.497, ZPL = 22.4%                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   STAGE 2: ΔSC F Pipeline                  │
│                                                             │
│  Quantum ESPRESSO (PBE functional, PAW pseudopotentials)   │
│         │                                                   │
│         ▼                                                   │
│  Ground State SCF → Identify T-center defect level        │
│  (Band 40, ε = +0.250 eV, gap = 0.752 eV above VBM)      │
│         │                                                   │
│         ▼                                                   │
│  Excited State: tot_charge = -1, full BFGS relaxation      │
│  → Atomic displacement ΔR extracted (PBC-corrected)        │
│         │                                                   │
│         ▼                                                   │
│  Local Atom Correction (PBE bandgap artefact removed)      │
│  S_total = 1.294, ZPL = 27.4%                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Results Summary

| Method | S_total | ZPL Fraction | Error |
|---|---|---|---|
| **MACE Pipeline (This Work)** | **1.497** | **22.4%** | **1.8%** |
| **ΔSC F Pipeline (This Work)** | **1.294** | **27.4%** | **12%** |
| Experiment (Bergeron et al., PRX Quantum 2020) | 1.47 | 23% | — |
| HSE06 DFT (Turiansky et al., ACS Nano 2026) | 1.88 | 15% | 28% |

**Key achievement:** MACE pipeline achieves 1.8% error vs experiment, outperforming the published HSE06 DFT benchmark (28% error) at 300x lower computational cost.

---

## ⚡ Computational Cost Comparison

| Method | Supercell | Force Evals | Est. Time | Accuracy |
|---|---|---|---|---|
| HSE06 DFT (Paper 1) | 512 atoms | 3,072 | ~32 days | High |
| PBE DFT (QE) | 55 atoms | 74 | ~18 hours | Moderate |
| **MACE (This Work)** | **55 atoms** | **74** | **~4 minutes** | **Comparable to PBE** |

**300x speedup over PBE DFT; ~11,000x speedup over HSE06 DFT**

---

## 🔬 Key Physics

### Huang-Rhys Factor
The ZPL fraction is determined by the Debye-Waller factor:

```
ZPL = e^(-S_total)

where S_total = Σ_i S_i = Σ_i (ω_i / 2ℏ) × (Δq_i)²

Δq_i = projection of mass-weighted displacement ΔQ onto phonon mode i
```

### Dominant Mode
The C-Si local vibrational mode at **68.9 meV** (above the Si phonon continuum at ~65 meV) contributes **45.8% of S_total** alone — an Einstein oscillator behavior explained by M_C < M_Si.

### Parseval Completeness
- MACE pipeline: 88.7% (6 modes excluded: 3 acoustic + 3 ghost)
- ΔSC F pipeline: 99.2% (3 acoustic modes excluded)

Both confirm the phonon basis captures the dominant displacement.

---

## 📁 Project Structure

```
t-center-zpl/
├── structure/
│   ├── build_supercell.py         # T-center 55-atom supercell construction
│   └── relax_mace.py              # BFGS relaxation with MACE
├── phonons/
│   ├── mace_phonopy.py            # MACE + Phonopy phonon calculation
│   ├── extract_eigenvectors.py    # Gamma-point eigenvector extraction
│   └── dos_analysis.py            # Phonon DOS visualization
├── huang_rhys/
│   ├── compute_hr.py              # Huang-Rhys factor calculation
│   ├── calibrated_pipeline.py     # MACE calibrated pipeline
│   └── luminescence_spectrum.py   # Optical lineshape computation
├── dscf/
│   ├── qe_inputs/
│   │   ├── ground_state.in        # QE ground state input
│   │   └── excited_state.in       # QE excited state (tot_charge=-1)
│   ├── extract_displacement.py    # PBC-corrected displacement extraction
│   └── local_correction.py        # PBE bandgap artefact correction
├── results/
│   ├── figures/                   # Publication-quality plots
│   └── data/                      # Numerical results
├── requirements.txt
└── README.md
```

---

## 🚀 Quick Start

```bash
# Clone
git clone https://github.com/ako-009/t-center-zpl.git
cd t-center-zpl

# Install dependencies
pip install -r requirements.txt
# Requires: ase, phonopy, mace-torch, numpy, matplotlib

# Build and relax T-center supercell
python structure/relax_mace.py

# Run MACE phonon calculation
python phonons/mace_phonopy.py

# Compute Huang-Rhys factor
python huang_rhys/calibrated_pipeline.py

# Run ΔSC F pipeline (requires Quantum ESPRESSO installation)
# See dscf/README.md for QE setup instructions
python dscf/extract_displacement.py
```

---

## 📚 References

1. Bergeron et al., *PRX Quantum* 1, 020301 (2020) — Experimental S=1.47, ZPL=23%
2. Turiansky, Lyons & Bernstein, *ACS Nano* 20, 7454 (2026) — HSE06 benchmark (S=1.88)
3. Batatia et al., *NeurIPS* 36, 11423 (2022) — MACE architecture
4. Alkauskas et al., *New J. Phys.* 16, 073026 (2014) — Huang-Rhys formalism
5. Togo & Tanaka, *Scr. Mater.* 108, 1 (2015) — Phonopy

---

## 📄 Research Report

Full internship report: [T_Center_Report_Final.pdf](reports/T_Center_Report_Final.pdf)

Supervised by **Prof. Sabyashachi Mishra**, Department of Chemistry, IIT Kharagpur
SRIC Look Within Internship — June 2026 to July 2026

---

## 👤 Author

**Abhishek Kumar Ojha**
B.S.-M.S. (5YR) | IIT Kharagpur | 22CY23003

[![GitHub](https://img.shields.io/badge/GitHub-ako--009-black?style=flat-square&logo=github)](https://github.com/ako-009)
