# End-to-End Aerospace Supply Chain Planning

A five-phase supply chain analytics project covering data foundation, demand planning, inventory management, supplier risk, and integrated business planning (IBP) scenario simulation for an aerospace parts network.

## Overview

An aerospace MRO / spares network operates across multiple sites, sourcing ~300 parts from 40 suppliers. This project builds an end-to-end analytical view of that network: it cleans and joins four raw data sources into a single analysis-ready layer, measures demand predictability and forecast accuracy, evaluates current inventory health against a proper safety-stock policy, scores supplier delivery and quality risk, and finally stress-tests the whole position against four plausible disruption scenarios.

**Data used:** ~281K weekly part/site inventory-and-consumption records, ~29.7K purchase order lines, 368 quality incidents, and a 300-part master file, spanning January 2022 - December 2024.

**Methods applied:** data validation and joins with pandas, OTIF/lead-time analysis, demand-pattern classification (ADI/CV² typology), WAPE/bias forecast accuracy and Forecast Value Added (FVA) analysis, safety-stock and reorder-point modeling, a composite supplier risk score, and scenario-based inventory stress-testing.

**What the project found:** the network sustains high service levels (>97% across all criticality classes) primarily through excess inventory rather than reliable supply — overall supplier OTIF is only 39.7%. A class-differentiated safety-stock policy would free up roughly **$56M** in inventory value, and scenario testing confirms that a phased, risk-aware reduction (not a uniform cut) can be made safely provided supplier risk is managed in parallel.

## Business Problem

Before this analysis, planning could see that service levels looked healthy but had no unified way to answer:
- Is the current inventory position efficient, or is it masking supply reliability problems?
- Is the demand forecast — and the manual review step applied to it — actually accurate?
- Which suppliers are the real source of delivery and quality risk, and how exposed are the most critical parts?
- How resilient is the current position to a demand spike, a lead-time blowout, or a supplier quality failure?

## Analytical Questions Answered

1. What does a clean, validated, joined view of parts / inventory history / purchase orders / quality incidents look like? *(Phase 1)*
2. How predictable is demand, and how accurate is the current forecast — including the value added by manual forecast adjustment? *(Phase 2)*
3. How much inventory is actually needed to protect service, and how much is excess? *(Phase 3)*
4. Which suppliers carry the most delivery and quality risk, and how exposed are Class A parts? *(Phase 4)*
5. How resilient is the current inventory position to demand, lead-time, and supplier-quality shocks? *(Phase 5)*

## Dataset

| File | Rows | Granularity | Description |
|---|---|---|---|
| `data/raw/parts_master.csv` | 300 | 1 row / part | Part family, criticality class (A/B/C), unit cost, lead time, primary supplier, repairability, shelf life |
| `data/raw/supply_chain_history.csv` | 280,800 | Part x Site x Week | Weekly consumption, on-hand, backorder, blocked quantity, and forecast (Jan 2022 - Dec 2024) |
| `data/raw/purchase_orders.csv` | 29,666 | 1 row / PO line | Order/promised/receipt dates and ordered vs. received quantity |
| `data/raw/quality_incidents.csv` | 368 | 1 row / incident | Defect severity/type and scrap quantity by supplier/part |

Data is included in this repository as it is a bounded, non-sensitive, project-scale dataset (~17 MB total) suitable for GitHub.

## Methodology

```
Raw CSVs (data/raw/)
        |
Phase 1 - Data Foundation: clean, type, validate, join
        |
Phase 2 - Demand Planning: demand classification, WAPE/bias, Forecast Value Added
        |
Phase 3 - Inventory & Warehouse: days of coverage, safety stock, excess/shortage
        |
Phase 4 - Supplier Performance & Risk: OTIF, quality incidents, composite risk score
        |
Phase 5 - IBP Simulation: demand / lead-time / supplier-quality shock scenarios
        |
Key Findings & Recommendations
```

Each phase notebook loads only the processed output of the phase(s) it depends on (see `data/processed/`), so the pipeline can be re-run from Phase 1 through Phase 5 in order.

## Key Findings

**1. High service levels are being bought with excess inventory, not reliable supply.**
Service level exceeds 97% for every criticality class, but supplier OTIF is only 39.7% — and inventory sits at several multiples of calculated safety stock across the board. *(Phases 1, 3)*

**2. There is a ~$56M inventory reduction opportunity, concentrated but not uniform.**
121 of 300 parts (40%) are classified as Excess. Moving to a class-differentiated safety-stock policy (45/30/21 target days for A/B/C) would release an estimated **$56.4M** in inventory value while keeping tailored protection on the most critical parts. *(Phase 3)*

**3. Forecast accuracy is adequate, but the manual "Adjusted" forecast step is not adding value.**
Overall WAPE is 30.3% at the part-week level (64-66% at the line level), with a small over-forecasting bias. The Adjusted forecast's WAPE is *worse* than the statistical Baseline (Forecast Value Added of **-1.25%**), and accuracy is materially weaker for unplanned-maintenance-driven demand (70% WAPE) than planned (56% WAPE). *(Phase 2)*

**4. Supplier risk is concentrated, and it reaches Class A parts.**
A composite risk score (OTIF, delay, quality incidents, Class A exposure) identifies **SUP033** as by far the highest-risk supplier (OTIF 3.0%, risk index 74), followed by SUP013 and SUP038. Of the 24 suppliers holding Class A parts, 11 have OTIF below 40% and 11 have logged a critical quality incident. *(Phase 4)*

**5. The excess-inventory buffer is doing real protective work under stress.**
Scenario testing shows a 20% Class A demand spike causes no shortage risk; a 30% lead-time extension pushes 15 parts (mostly Class C) into shortage; a SUP033 quality event affects $6.3M in inventory value but causes no shortage; and the combined demand + lead-time shock — the worst case tested — puts only $0.96M (about 1%) of the $85M inventory position at risk. *(Phase 5)*

## Business Recommendations

| Finding | Operational Issue | Recommended Action | Expected Benefit |
|---|---|---|---|
| 40% of parts are Excess vs. a proper safety-stock policy | ~$56M tied up in working capital | Phase a drawdown of excess stock, prioritizing low-criticality, high-excess parts first | Releases working capital without materially increasing shortage risk (per Phase 5 stress test) |
| Lead-time risk (not demand risk) drives the most shortage exposure, concentrated in Class C | A uniform inventory cut would raise shortage risk exactly where the network is already weakest | Retain a lead-time risk buffer for Class C parts; pair inventory reduction with lead-time-reduction initiatives | Avoids trading working-capital savings for stockout risk |
| Manual forecast adjustment shows negative Forecast Value Added | Planner time is spent on a step that doesn't improve accuracy | Review or retire the current manual adjustment process; redirect effort to the Erratic-demand subset (18% of parts) | Frees planner capacity, likely improves overall forecast accuracy |
| Supplier risk is concentrated in SUP033, SUP013, SUP038 — several holding Class A parts | Single-source exposure on critical parts to underperforming suppliers | Prioritize dual-sourcing or a formal supplier development plan for these three suppliers, starting with Class A parts | Reduces the single largest source of measured supply risk in the network |

## Repository Structure

```
End-to-End-Aerospace-Supply-Chain-Planning/
|
├── README.md
├── requirements.txt
├── .gitignore
|
├── data/
│   ├── raw/                                    # source CSVs
│   │   ├── parts_master.csv
│   │   ├── supply_chain_history.csv
│   │   ├── purchase_orders.csv
│   │   └── quality_incidents.csv
│   └── processed/                              # intermediate outputs, written by the notebooks
│       ├── phase1/ ... phase5/
|
├── notebooks/
│   ├── 01_data_foundation.ipynb
│   ├── 02_demand_planning.ipynb
│   ├── 03_inventory_warehouse_management.ipynb
│   ├── 04_supplier_performance_risk.ipynb
│   └── 05_ibp_scenario_simulation.ipynb
|
└── images/                                     # exported dashboard charts (also embedded in the notebooks)
    ├── phase1_kpi_dashboard.png
    ├── phase2_demand_dashboard.png
    ├── phase3_inventory_dashboard.png
    ├── phase4_supplier_risk_dashboard.png
    └── phase5_ibp_scenario_dashboard.png
```

## How to Run

```bash
git clone <this-repo-url>
cd End-to-End-Aerospace-Supply-Chain-Planning
python -m venv .venv && source .venv/bin/activate   # optional but recommended
pip install -r requirements.txt
jupyter notebook notebooks/
```

Run the notebooks in order (01 -> 05); each phase reads the processed output of the phase(s) before it from `data/processed/`, which is already populated in this repository but will be regenerated if you re-run from Phase 1.

## Tools & Technologies

- Python
- pandas / NumPy
- Matplotlib / Seaborn
- Jupyter Notebook
- pyarrow (Parquet I/O for intermediate data)

## Limitations

- Safety stock and reorder-point calculations use a simplified formula (Z x σ x √lead time) rather than a full multi-echelon inventory model.
- Demand classification (ADI/CV²) and forecast accuracy are computed after aggregating consumption across sites to the part-week level; site-level dynamics are not modeled separately.
- Scenario simulation (Phase 5) applies stylized, uniform shocks (e.g., "+20% demand," "50% blocked") rather than modeling shock propagation dynamically over time.
- The supplier risk index is a simple weighted composite, not a statistically fitted or validated risk model.

## Future Improvements

- Replace the simplified safety-stock formula with a full multi-echelon or simulation-based inventory model.
- Model scenario shocks as time-series events (ramp-up/decay) rather than static one-period adjustments.
- Extend supplier risk scoring with a proper weighted/validated model (e.g., logistic regression against historical delivery failures).
- Add a lightweight interactive dashboard (e.g., Streamlit) on top of the Phase 3/4/5 outputs for non-technical stakeholders.
