# Genome-Informed Media Optimization for *Lactobacillus rhamnosus*  
**Adaptive DOE (1,000 conditions) + Lightweight QC Framework**

This repository contains a technical case study focused on designing a practical, data-driven media optimization strategy for *Lactobacillus rhamnosus*. The work combines 

(1) genome-informed nutrient reasoning, 

(2) a lightweight QC system for plate experiments, and 

(3) an adaptive 1,000-condition Design of Experiments (DOE) balancing exploration vs. exploitation.

---

## Goals

1. **QC framework + dashboard-ready report** to quickly answer:
   - *Is this run trustworthy?*
   - *What went wrong (if anything)?*
   - *Are there systematic plate/edge/batch effects?*

2. **Design a 1,000-condition media DOE** under lab constraints:
   - ≤ **5 discrete concentration levels** per component
   - Balanced **exploration (400)** + **exploitation (600)**

3. **Genome-informed recommendations** to prioritize likely limiting nutrients for *L. rhamnosus*.

---

## High-Level Approach

### Part 3 — QC First (Data-Informed DOE)
Before designing the DOE, we build a minimal QC system to characterize assay behavior and technical variation.

**Run-level QCs**
- Missingness / duplicates / value bounds
- OD distribution sanity checks
- Replicate precision summary (CV, disagreement rate)

**Plate-level QCs**
- Plate completeness
- Plate shift detection (median vs global median)
- Edge effect detection (edge vs interior medians)
- Spatial pattern detection (heatmaps)

**Well-level QCs**
- Replicate discordance (CV threshold + binary disagreement)
- Outliers (optional robust z-score within plate)
- Final classification: `top_candidate`, `true_no_growth`, `rerun`

---

### Part 2 — Adaptive DOE (Exploration → Exploitation)
We design a two-stage DOE:

**Exploration (N=400)**
- Discrete Latin Hypercube sampling (LHS)
- Balanced marginal distributions across component levels  
- Goal: broad coverage for learning main effects and interactions

**Exploitation (N=600)**
- Train predictive model on trustworthy historical data (and exploration results when available)
- Generate a large candidate pool and score by predicted OD
- Select high-performing conditions while enforcing diversity (greedy max–min)

---

## Media Components & Discrete Levels

All components use ≤5 concentration levels for liquid handling practicality:

| Component | Levels |
|---|---|
| K2HPO4 (Dipotassium phosphate) | 1, 2, 4 |
| Peptone (Soy) | 0, 5, 10, 20, 40 |
| Glucose | 0, 10, 20, 40, 50 |
| Ammonium citrate | 0, 1, 2, 4, 10 |
| MgSO4 (Magnesium sulfate) | 0, 0.1, 0.2, 1.0, 2.0 |
| Yeast extract | 0, 5, 10, 20, 40 |
| MnSO4·H2O (Manganese sulfate) | 0, 0.1, 0.2, 1.0, 2.0 |
| Meat extract | 0, 5, 10, 20, 40 |
| Sodium acetate (CH3COONa) | 0, 2.5, 5, 10, 15 |
| Tween 80 (ml/L) | 0, 0.5, 1, 2, 4 |

---

## Key Outputs
**QC**
- Run trustworthiness summary (PASS / CAUTION / FAIL)
- Plate shift table + plate-level boxplots
- Edge effect analysis per plate
- Example plate heatmaps (spatial clustering)
- Replicate agreement scatter + flagged reruns

**DOE**
- exploration_400.csv — balanced LHS exploration set
- exploitation_600.csv — high-performing, diversity-aware exploitation set
- doe_1000.csv — combined DOE plan

---

## Methods Notes
**Plate shift**

Detected using plate median OD relative to global median.
Optionally median-centered normalization is used for downstream modeling while preserving raw OD for traceability.

**Edge effects**

Edge wells are defined as:

- Rows A/H or columns 1/11 (adjust based on plate indexing)
Edge suppression flagged when interior median exceeds edge median by a threshold.

**Diversity-aware exploitation (greedy max–min)**

After filtering to top predicted candidates, select conditions iteratively to maximize the minimum distance from previously selected points in normalized feature space. This prevents the exploitation set from collapsing into near-duplicate media.

---

##Limitations / Future Improvements

- No explicit negative/positive controls were available in the historical dataset; QC relies on replicate concordance and spatial diagnostics as proxies.
- Future runs should include:
  - media-only blanks
  - reference condition(s) per plate
  - metadata for batch/incubation position to model batch effects directly
