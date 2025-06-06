# Coarse-Grained Chromatin–Lamin Simulation

This repository describes the setup and simulation details for a coarse-grained model of chromatin and lamin interactions, inspired by experimental data from the mouse **C2C12** cell line (GRCm38/mm10).

---

## 🧬 Coarse-Grained Model of Chromatin and Lamins

### Chromatin Model

- A 10.7 Mbp region of **mouse chromosome 18** was modeled.
- Chromatin is represented as a **heteropolymer chain** with:
  - **2140 beads**
  - Each bead corresponds to **5 kbp** of DNA (~25 nucleosomes)
  - Bead diameter: `σ`
  - Beads are classified as:
    - **Euchromatin (EC)**
    - **Heterochromatin (HC)**

Classification is based on **H3K9me3 ChIP-seq** enrichment [[Beyer et al., 2016]](https://doi.org/10.1016/j.celrep.2016.06.081).

#### HC Fraction Calculation (`f_HC`)
- Chromosome segments were binned into **2140 × 5kb** windows.
- A bin is HC if ≥ 50% of it is enriched in H3K9me3.
- `f_HC` = fraction of HC beads out of 2140.

### Random Block Copolymer Control

- Segment = block of 20 consecutive beads.
- Beads randomly labeled EC or HC based on `f_HC`.

### Lamin Model

- Lamins are modeled as coarse-grained particles with:
  - Diameter: `σ_L = σ / 2`
  - Total count: **2000 lamin particles**

---

## 🧱 Simulation Box

- Dimensions: `20σ × 20σ × 35σ` (x, y, z)
- **Periodic** boundary in `x`, `y`; **fixed** boundary in `z`
  - Simulates confinement near the **nuclear envelope**

### Volume Fractions

Chromatin:
```
V_chromatin / V_box = (2140 × 4πσ³/24) / (14000σ³) ≈ 0.08
```

Lamin:
```
V_lamin / V_box = (2000 × 4πσ³/192) / (14000σ³) ≈ 0.01
```

---

## ⚙️ Interaction Potentials

The **total energy** of the system is:

```
U = Σ_{i=1}^{N-1} U_spring(|r_{i+1}-r_i|) + Σ_{i=1}^{N-1} Σ_{j=i+1}^{N} U_pair(|r_j - r_i|)
```

### 1. FENE Spring Potential

Used for polymer backbone connectivity:
```
U_spring(r) = -½ k R₀² ln[1 - (r / R₀)²]
```
- `k = 30 k_B T / σ²`
- `R₀ = 1.6σ`

### 2. Nonbonded Interactions

Depending on the context, either **Lennard-Jones (LJ)** or **WCA** (purely repulsive LJ) potential is used.

#### Lennard-Jones (LJ)

```
U_LJ(r) =
  4ε[(σ/r)^12 - (σ/r)^6],   r < 1.8σ
  0,                       r ≥ 1.8σ
```

#### Weeks-Chandler-Andersen (WCA)

```
U_WCA(r) =
  4ε[(σ/r)^12 - (σ/r)^6] + E_cut,   r < 1.12σ
  0,                               r ≥ 1.12σ
```

Where the shift:
```
E_cut = -4ε[(σ/r_c)^12 - (σ/r_c)^6],  r_c = 1.12σ
```

---

## 🧲 Boundary Interactions

- `z = +17.5σ` (upper): **Attractive LJ** interaction (nuclear membrane with lamin)
- `z = -17.5σ` (lower): **Completely repulsive** for all particles

---

## 📋 Table of Interaction Parameters

| Interaction Type        | Potential Used | Notes |
|-------------------------|----------------|-------|
| Chromatin–Chromatin     | LJ / WCA       | EC–EC, HC–HC, HC–EC types |
| Lamin–Lamin             | LJ / WCA       | Short-range attraction |
| Chromatin–Lamin         | LJ / WCA       | HC-specific attractive interaction |
| Wall (Top)–Lamin        | LJ             | Simulates membrane |
| Wall (Bottom)–All       | WCA            | No penetration |

---

## 📚 Reference

Beyer, T. A., et al. (2016). *Canonical histone marks are predictive of chromatin architecture*. Cell Reports, 17(2), 307-320. https://doi.org/10.1016/j.celrep.2016.06.081
