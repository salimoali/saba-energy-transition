# saba-energy-transition

Integrated energy system modelling for Saba Island’s 100% renewable electricity transition by 2035.

The project covers the full modelling chain: stochastic demand forecasting (RAMP) → renewable resource data (Renewables.ninja) → energy system optimisation (Calliope) → financial investment analysis.

-----

## Files

|File                          |Description                          |
|------------------------------|-------------------------------------|
|`saba_2035_demand_model.ipynb`|RAMP stochastic demand model         |
|`saba_demand_median.csv`      |Median demand profile (8,760 hours)  |
|`saba_demand_all_runs.csv`    |All 50 Monte Carlo run outputs       |
|`modelsaba.yaml`              |Calliope model configuration         |
|`saba_model_final.py`         |Run script + results visualisation   |
|`renewables2.csv`             |Renewables.ninja capacity factor data|

-----

## Part 1 — RAMP Stochastic Demand Model

**Tool:** [RAMP](https://github.com/RAMP-project/RAMP)
**Notebook:** `demand/saba_2035_demand_model.ipynb`

### Why RAMP

Saba has limited public metering infrastructure. A deterministic demand profile would be unreliable at this scale. RAMP is designed for bottom-up stochastic modelling in data-scarce environments, building load profiles from appliance-level usage patterns rather than aggregated historical data.

### User categories modelled

|Category                               |Units       |
|---------------------------------------|------------|
|Basic households                       |500         |
|Higher-consumption households (with AC)|200         |
|Tourist guestrooms                     |70          |
|Commercial units                       |30          |
|Public lighting                        |925 fixtures|

EVs excluded — not yet deployed on Saba.

### Key modelling decisions

- **Monte Carlo runs:** 50 — produces reliable P10–P90 uncertainty band
- **Seasonal tourism factor:** 85% occupancy Nov–Apr, 40% May–Oct
- **AC coincidence factor:** 0.45 to prevent unrealistic simultaneous switching
- **Output:** 8,760-hour annual profile; median used as central estimate; P10/P90 used as ±20% sensitivity band

### QA validation results

|Metric          |Value       |Target      |Status|
|----------------|------------|------------|------|
|Annual energy   |9,003 MWh/yr|8,500–10,500|✓ PASS|
|Peak demand     |2.21 MW     |1.5–2.5 MW  |✓ PASS|
|Load factor     |0.315       |0.25–0.45   |✓ PASS|
|Monte Carlo runs|50          |≥ 30        |✓ PASS|

### Solar–demand gap

Solar generation peaks 12:00–15:00; demand peaks 18:00–22:00. This 3–7 hour gap is the primary technical argument for battery storage sizing and was the key output handed to the Calliope model.

-----

## Part 2 — Calliope Energy System Optimisation

**Tool:** [Calliope](https://calliope.readthedocs.io/)
**Config:** `calliope/modelsaba.yaml`
**Script:** `calliope/saba_model_final.py`

### Inputs

|File                    |Source                                      |Content                                                                            |
|------------------------|--------------------------------------------|-----------------------------------------------------------------------------------|
|`renewables2.csv`       |Renewables.ninja (Lat 17.6320, Lon -63.2733)|Hourly capacity factors for wind and solar at Saba                                 |
|`saba_demand_median.csv`|RAMP model                                  |Median 8,760-hour demand profile                                                   |
|`locations.yaml`        |Manual                                      |Installed capacities and battery specs                                             |
|`techs.yaml`            |Manual                                      |Technology definitions: solar PV, wind, battery, diesel, load shedding, curtailment|

### Optimised system capacities (2035 scenario)

|Technology     |Installed capacity|Annual generation     |
|---------------|------------------|----------------------|
|Solar PV       |4.720 MW          |8,082.7 MWh           |
|Wind           |0.577 MW          |2,034.9 MWh           |
|Battery storage|1.684 MW / 9.8 MWh|6,942.4 MWh dispatched|
|Load shedding  |0.907 MW          |45.6 MWh              |


> Wind capacity was installed but not dispatched in the normal scenario — solar and battery were sufficient at lower cost. Wind activates in the hurricane stress-test scenario.

### Scenarios

**Scenario 1 — Normal year (2035 optimised):** Full demand met by solar + battery. Batteries store surplus midday solar and dispatch at night. Small residual load shedding (45.6 MWh) is cost-optimal vs. oversizing battery bank.

**Scenario 2 — Hurricane stress-test:** 5-day solar blackout. Solar generation marginalised; system falls back on wind + battery + diesel. Retaining 54 MWh/year of emergency diesel costs only €12,432/year and drops IRR by just 0.21%.

### Configuration fixes applied

- Demand CSV column naming corrected to match Calliope’s `sink_use_equals` variable
- Timestep index updated from 2019 baseline to 2035 scenario year
- Capacity values scaled from 2019 baseline to 2035 optimised targets
- Deprecated YAML keys updated to Calliope 0.6.x API
- Financial model cross-checked against Calliope outputs — capacity mismatch identified and corrected

-----

## Part 3 — Investment Analysis (summary)

|Metric                        |Baseline   |
|------------------------------|-----------|
|Total CAPEX                   |€8,480,060 |
|Battery share of CAPEX        |45.2%      |
|Battery replacement at Year 15|€3.84M     |
|Annual avoided diesel revenue |€1,400,000 |
|LCOE                          |75.10 €/MWh|
|Payback period                |9 years    |
|IRR                           |12.31%     |
|DSCR                          |4.47×      |

Worst-case scenario (+20% CAPEX + annual hurricane): payback 10 years, IRR >9.2%, NPV positive.

-----

## Requirements

```
ramp-mobility
calliope>=0.6
pandas
numpy
matplotlib
scipy
jupyter
```

```bash
pip install ramp-mobility calliope pandas numpy matplotlib scipy jupyter
```

-----

## Running the models

```bash
# RAMP demand model
jupyter notebook demand/saba_2035_demand_model.ipynb

# Calliope optimisation
conda activate calliope
cd calliope/
python saba_model_final.py
```