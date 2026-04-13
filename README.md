# AI-VFC Simulation Project

**An AI-Integrated Vehicular Fog Computing Simulation**
Based on the paper: *"An AI-Integrated Conceptual Framework for Vehicular Fog Computing: Architecture, Challenges and Future Directions"*

---

## Project Structure

```
ai_vfc_project/
├── config/
│   └── settings.py          # All tunable parameters (weights, thresholds, paths)
├── data/
│   └── ai_vfc_offline_extended.csv   # Simulation dataset
├── core/
│   ├── cost_function.py     # C = αL + βE + γW per-layer cost computation
│   ├── decision_engine.py   # DRL-inspired greedy target selection
│   └── simulator.py         # Dataset loader + simulation orchestrator
├── evaluation/
│   └── metrics.py           # Seven KPI metrics computed from simulation output
├── utils/
│   └── io_utils.py          # Report printer, file savers, plot generator
├── results/                 # ← EMPTY before first run; populated by main.py
├── main.py                  # Entry point
└── README.md
```

---

## Prerequisites

```bash
pip install pandas numpy matplotlib
```

Python 3.8+ required.

---

## How to Run

```bash
cd ai_vfc_project
python main.py
```

The `results/` folder will be **created and populated** only after running the script.

---

## Outputs (after running)

| File | Description |
|------|-------------|
| `results/metrics.txt` | Formatted performance report |
| `results/metrics.json` | Machine-readable metrics |
| `results/simulation_output.csv` | Full per-task simulation output |
| `results/plot_target_distribution.png` | Vehicle / Fog / Cloud task counts |
| `results/plot_latency_distribution.png` | Effective latency histograms by layer |
| `results/plot_energy_distribution.png` | Effective energy histograms by layer |
| `results/plot_metrics_summary.png` | All 7 KPIs in one bar chart |
| `results/plot_queue_length.png` | Rolling queue length over time |
| `results/plot_utilisation_by_priority.png` | Resource utilisation by priority class |

---

## Core Algorithm

### Cost Function
```
C = αL + βE + γW
```
- **L** = effective latency (dataset latency × layer factor)
- **E** = effective energy (dataset energy × layer factor)
- **W** = workload (with over-capacity penalty per layer)
- **α, β, γ** = adaptive weights driven by `Vehicle_Priority`

### Layer Characteristics

| Layer   | Latency Factor | Energy Factor | Workload Cap |
|---------|---------------|--------------|-------------|
| Vehicle | 0.55          | 0.45         | 0.45        |
| Fog     | 0.90          | 0.80         | 0.80        |
| Cloud   | 1.55          | 1.30         | 1.00        |

- Cloud is **excluded** when `offline_mode == 1`.
- All eligible layers are evaluated; **minimum cost wins** (no bias).

### Adaptive Weights

| Priority | α (Latency) | β (Energy) | γ (Workload) |
|----------|------------|-----------|-------------|
| 1 (low)  | balanced   | +0.10     | base        |
| 2 (med)  | +0.10      | base      | base        |
| 3 (high) | +0.25      | base      | base        |

---

## Integrity Guarantees

- ✅ No hardcoded results
- ✅ No pre-generated outputs
- ✅ No label (`execution_target`) used for decisions
- ✅ Cloud is always evaluated when online
- ✅ All metrics computed from simulation data
- ✅ Re-run produces identical results (deterministic cost function)
- ✅ Different dataset → different results

---

## Tuning Parameters

All parameters are in `config/settings.py`:

- `ALPHA_BASE`, `BETA_BASE`, `GAMMA_BASE` — base cost weights
- `LAYER_PARAMS` — per-layer latency/energy/workload multipliers
- `SUCCESS_COST_THRESHOLD` — offload-success percentile threshold
- `TARGET_RANGES` — PASS/FAIL thresholds for each metric
