# Option Pricing & Risk Management via Linear Additive Models

**Final Project — Financial Engineering (Group 2b)**  
Politecnico di Milano · A.Y. 2024/2025

---

## Overview

This project implements a complete quantitative finance pipeline for the **crude oil options market**, covering model calibration, Monte Carlo simulation, exotic option pricing, and Greeks-based risk management. Three **linear additive processes** are compared throughout:

| Model | Name | Key parameters |
|-------|------|----------------|
| **AB** | Additive Bachelier | η (volatility level), κ (mean-reversion speed) |
| **MA** | Minimal Additive | α, β (shape parameters) |
| **GL** | Generalized Logistic | α, β (shape parameters) |

Additive processes generalise Lévy processes by allowing **time-inhomogeneous increments**, which makes them better suited to capture the term structure of implied volatility observed in commodity markets.

The valuation date is **2 June 2020**, in the middle of the COVID-19 oil price shock.

---

## Repository Structure

```
RunFinalProject2b.m          ← Main entry point (runs the full pipeline)
src/
├── data/                    ← Market data loading and pre-processing
├── calibration/             ← Discount curve, forward curve, implied-vol surface
├── models/                  ← Characteristic functions and PDFs (AB / MA / GL)
├── simulation/              ← Monte Carlo increment simulation
├── pricing/                 ← Exotic option pricing (two methods)
└── risk_management/         ← Delta / Vega hedging and P&L analysis
Data/
├── datacalls/               ← Market call prices (Jun 2020 – Dec 2022, 9 maturities)
└── dataputs/                ← Market put prices (same maturities)
Biblio/                      ← Reference papers
```

---

## Pipeline (Points 0 – 6)

### Point 0 — Model PDF comparison
Visual comparison of the probability density functions of AB, MA, and GL under a common parametrisation, to build intuition on the tail behaviour of each process.

### Point 1 — Market curve calibration
Bootstrap of the **discount curve** and the **forward curve** (including absolute dividends) from futures prices and option put-call parity. Outputs discount factors B(t) and forward prices F(T) for all available maturities.

### Point 2 — Implied volatility surface calibration
Constructs the forward-OTM surface, estimates **ATM normal (Bachelier) volatility** by maturity, and calibrates each model's parameters by minimising the pricing error on the full surface. Option prices are computed via the **Lewis-FFT** formula using the model's characteristic function.

### Point 3 — Simulation and forward-start option pricing
- Simulates **T₁→T₂ increments** (5 million paths) for each model using a fast FFT-based inversion scheme.
- Performs tail analysis: estimated vs asymptotic tail coefficients (λ⁻, λ⁺).
- Reconstructs the CDF via Lewis-FFT and validates against the simulated empirical distribution.
- Prices a **forward-start option** (FSO) by Monte Carlo for all three models, and cross-checks against the closed-form MA result.

### Point 4 — Exotic option pricing
Two path-dependent options are priced under each model with **two numerical methods**:

| Option | Description |
|--------|-------------|
| **CoC** (Cash or Call) | Pays max(S(T₁) − K₁, 0) or a fixed cash amount at T₂ |
| **PoP** (Put or Pay) | Pays max(K₁ − S(T₁), 0) or a fixed payment at T₂ |
| **Chooser** | Holder chooses at T₁ between the CoC and the PoP |

The two pricing methods are:
- **Stochastic mesh (grid)** — regression-based conditional expectation on a grid of intermediate points.
- **No-grid** — direct Monte Carlo payoff averaging without intermediate grid.

Timing and speedup comparisons are reported. For the MA model, Monte Carlo prices are cross-validated against **closed-form analytical formulas**.

### Point 5 — Closed-form vs numerical validation (MA model)
Compares the analytical pricing formula for CoC, PoP, and Chooser under the MA model against the stochastic mesh MC prices, to verify numerical accuracy.

### Point 6 — Risk Management (AB model)
Implements a **Delta–Vega neutral hedging strategy** on a portfolio of exotic options using vanilla ATM calls (6M and 12M maturities) as hedging instruments.

- **Delta** and **Vega** computed via finite-difference bumps (absolute bumps in price/vol units, consistent with the normal-volatility convention).
- Hedge positions in the two vanilla options are determined by solving a 2×2 Greek system.
- P&L analysis and cash-flow summary are reported at the valuation date.

---

## How to Run

1. Open MATLAB and set the working directory to the project root.
2. Run:
   ```matlab
   RunFinalProject2b
   ```
   The script automatically adds all `src/` subfolders to the path and executes the full pipeline.

**Requirements:** MATLAB R2021b or later (uses `datetime`, `table`, and standard optimisation/FFT routines — no additional toolboxes required beyond the Optimization Toolbox).

---

## Key References

- Baviera & Manzoni (2026) — *Fast Generalized Monte Carlo for additive processes* (FGMC)
- Carr & Torricelli (2021) — *Additive logistic processes in option pricing*
- Azzone & Baviera (2022) — *Synthetic forwards and cost of funding in the equity derivative market*
- Baviera (2023) — *The additive Bachelier model with an application to the oil option market in the Covid period*

---

## Authors

Group 2b — Financial Engineering, Politecnico di Milano
