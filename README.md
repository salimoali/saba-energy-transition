# saba-energy-transition

Integrated energy system modelling for Saba Island's 100% renewable electricity transition by 2035.

The project covers the full modelling chain: stochastic demand forecasting (RAMP) → renewable resource data (Renewables.ninja) → energy system optimisation (Calliope) → financial investment analysis.

---

## Repository structure
saba-energy-transition/
├── demand/
│   ├── saba_2035_demand_model.ipynb
│   ├── saba_demand_median.csv
│   └── saba_demand_all_runs.csv
└── energy_sources/
└── renewables2.csv
## Part 1 — RAMP Stochastic Demand Model

**Tool:** RAMP
**Notebook:** `saba_2035_demand_model.ipynb`

### User categories modelled

| Category | Units |
|----------|-------|
| Basic households | 500 |
| Higher-consumption households (with AC) | 200 |
| Tourist guestrooms | 70 |
| Commercial units | 30 |
| Public lighting | 925 fixtures |

### QA validation results

| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| Annual energy | 9,003 MWh/yr | 8,500–10,500 | ✓ PASS |
| Peak demand | 2.21 MW | 1.5–2.5 MW | ✓ PASS |
| Load factor | 0.315 | 0.25–0.45 | ✓ PASS |
| Monte Carlo runs | 50 | ≥ 30 | ✓ PASS |

Solar generation peaks 12:00–15:00; demand peaks 18:00–22:00. This gap drives battery storage sizing.

---

## Part 2 — Calliope Energy System Optimisation

**Tool:** Calliope

### Optimised system capacities (2035)

| Technology | Installed capacity | Annual generation |
|------------|-------------------|-------------------|
| Solar PV | 4.720 MW | 8,082.7 MWh |
| Wind | 0.577 MW | 2,034.9 MWh |
| Battery storage | 1.684 MW / 9.8 MWh | 6,942.4 MWh |
| Load shedding | 0.907 MW | 45.6 MWh |

Two scenarios modelled: normal year and 5-day hurricane blackout stress-test.

---

## Part 3 — Investment Analysis

| Metric | Value |
|--------|-------|
| Total CAPEX | €8,480,060 |
| LCOE | 75.10 €/MWh |
| Payback period | 9 years |
| IRR | 12.31% |
| DSCR | 4.47× |

---

## Requirements
ramp-mobility
calliope>=0.6
pandas
numpy
matplotlib
jupyter
