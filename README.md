# Causal Discovery on Recidivism Data: Experimental Framework

## Overview

This repository contains the complete experimental pipeline for applying causal discovery algorithms to the NIJ recidivism dataset and a set of causal benchmarks (CausalPitfalls). The work investigates whether causal structure discovery can identify meaningful causal relationships in criminal justice recidivism outcomes, with attention to algorithmic performance across synthetic (ground-truth) and real-world (unlabelled) scenarios.

**Key research questions:**
- How do causal discovery algorithms perform on mixed-type criminal justice data?
- Which causal structures are robust across multiple algorithms and bootstrap resamples?

---

## Repository Structure

```
recidivism-causal/
├── algorithms/                          # Third-party algorithm implementations (submodules/forks)
│   ├── causalexplain/                  # ReX algorithm
│   ├── dagslam/                        # DAGSLAM (mixed-type causal discovery)
│   └── lingam/                         # LiM (linear non-Gaussian models)
│
├── code/                                # Experimental pipeline and utilities
│   ├── experimental_code_clean.py      # Consolidated utility functions + main experiments
│   ├── preprocessing/                  # Data loading, cleaning, encoding
│   └── wrappers/                       # Algorithm-specific input/output standardization
│
├── data/
│   ├── raw/
│   │   ├── CausalPitfallsData/        # Benchmark datasets with ground truth DAGs
│   │   └── NIJ_Georgia_recidivism/    # Original NIJ dataset
│   └── processed/
│       ├── NIJ_lean_compact_onehot.csv         # Processed: one-hot encoded, dimension-reduced
│       ├── NIJ_lean_compact_adjacency_gt.csv   # (If available: manual expert adjacency)
│       └── metadata/                           # Variable definitions, encoding schemes
│
├── results/
│   ├── graphs/                         # Learned DAG visualizations
│   │   ├── causal_pitfalls/
│   │   │   ├── graphs_DAGBagM/
│   │   │   ├── graphs_DAGSLAM/
│   │   │   ├── graphs_fci/
│   │   │   ├── graphs_HC/
│   │   │   ├── graphs_LiM/
│   │   │   └── graphs_ReX/
│   │   └── NIJ/
│   │       ├── graphs_DAGBagM/
│   │       ├── graphs_DAGSLAM/
│   │       ├── graphs_fci/
│   │       ├── graphs_HC/
│   │       ├── graphs_LiM/
│   │       ├── graphs_ReX/
│   │       └── NIJ_graph_DAGSLAM_PRUNED.png  # Final Markov blanket of recidivism
│   │
│   ├── adjacency_matrices/             # Learned adjacency matrices (CSV)
│   │   ├── causal_pitfalls/
│   │   └── NIJ/
│   │
│   ├── metrics/
│   │   ├── scores_*.csv                # SHD, TPR, FPR, FDR per algorithm × scenario
│   │   └── edge_stability_*.csv        # Bootstrap resampling results
│   │
│   └── stability_analysis/
│       ├── NIJ_DAGSLAM_edge_stability.csv     # Edge frequency across bootstrap samples
│       └── incompatibility_scores_*.csv       # Self-consistency metrics
│
└── reports/
    ├── CS344_Project_Specification.pdf
    ├── CS344_Progress_Report.pdf
    ├── CS344_Final_Project_Management_Report.pdf
    └── CS344_Final_Report.pdf
```

---

## Algorithms

This work compares six causal discovery algorithms, each chosen for specific strengths:

### 1. **DAGSLAM** (Zhao & Jin, 2025)
- **Specialization:** Mixed-type data (continuous + discrete variables)
- **Method:** Constrained optimization with mixed loss functions
- **Implementation:** `code/dagslam/` (forked from https://github.com/yuanyuan-zhao-pku/DAGSLAM)
- **Reference:** Zhao, Y. & Jinzhu, J. (2025). DAGSLAM: Causal Bayesian Network Structure Learning of Mixed Type Data.
- **Wrapper:** `run_dagslam(df, lambda1=0.03, max_iter=100, w_threshold=0.25)`

### 2. **HC** (Hill Climbing via bnlearn (Scutari, 2010))
- **Specialization:** Discrete and mixed data; Bayesian network structure learning
- **Method:** Greedy local search with BIC or conditional Gaussian scores
- **Implementation:** R package `bnlearn` (called via rpy2)
- **Score selection:** Auto-detects mixed types and applies `bic-cg` (conditional Gaussian)
- **Reference:** Scutari, M. (2010). Learning Bayesian networks with the bnlearn R package. *Journal of Statistical Software*, 35(3), 1–22.
- **Wrapper:** `run_hc(df, seed=1)`

### 3. **LiM** (Linear Non-Gaussian Models via lingam (Zeng et al. 2022))
- **Specialization:** Continuous data with non-Gaussian noise
- **Method:** Independent component analysis (ICA) based causal discovery
- **Implementation:** Python package `lingam` (v0.7.x)
- **Reference:** Zeng, Y., Shimizu, S., Matsui, H., & Sun, F. (2022). Causal discovery for linear mixed data. _CLeaR 2022_, PMLR 177, 994–1009.
- **Wrapper:** `run_lim(df, seed=1)`

### 4. **DAGBagM** (Bayesian Networks with Mixed Variables (Chowdhury, Wang, Yu et al. 2022))
- **Specialization:** Mixed discrete/continuous data; Bayesian scoring
- **Method:** Hill climbing with multiple random restarts
- **Implementation:** R package `dagbagM`
- **Reference:** Chowdhury, S., Wang, R., Yu, Q. et al. DAGBagM: learning directed acyclic graphs of mixed variables with an application to identify protein biomarkers for treatment response in ovarian cancer. BMC Bioinformatics 23, 321 (2022). https://doi.org/10.1186/s12859-022-04864-y
- **Wrapper:** `run_dagbagm(df, seed=1)`

### 5. **FCI** (Fast Causal Inference via Causal Discovery Toolbox (Kalainathan, Goudet 2019))
- **Specialization:** Continuous data; handles hidden confounders (PDAGs)
- **Method:** Constraint-based approach using partial correlations (Fisher-Z test)
- **Implementation:** Python package `causaldag` / `cdt`
- **Output:** Partial DAG (PDAG) with directed and undirected edges
- **Reference:** Kalainathan, D., & Goudet, O. (2019). Causal discovery toolbox: Uncover causal relationships in python. arXiv preprint arXiv:1903.02278
- **Wrapper:** N/A for empirical dataset

### 6. **ReX** (Renero, 2024)
- **Specialization:** Mixed-type data; symbolic regression based
- **Method:** Causal model discovery via symbolic relationships
- **Implementation:** Python package `causalexplain` (v0.9.1)
- **Reference:** https://github.com/renero/causalexplain
- **Wrapper:** `run_rex(df, experiment_name="rex_exp")`

---

## Data

### Raw Data

#### CausalPitfalls Benchmark (with Ground Truth)
- **Source:** Causal Pitfalls dataset suite (public benchmark)
- **Location:** `data/raw/CausalPitfallsData/`
- **Contents:** Multiple scenarios with synthetic data and ground-truth DAGs
- **Use:** Algorithm benchmarking with known causal structures
- **Scenarios included:**
  - `casual_effect/`
  - `causal_direction_iv/`
  - `counterfactual_reasoning/`
  - `dag_structure_markequi/`
  - `domain_shift/`
  - `mediation_outcome_confounder/`
  - `moderation_effect/`
  - `necessity_sufficiency/`
  - `sequential_mediator/`
  - `treatment_mediator/`

#### NIJ Georgia Recidivism Dataset (Real-World)
- **Source:** National Institute of Justice (NIJ), Georgia Department of Corrections
- **Location:** `data/raw/NIJ_Georgia_recidivism/`
- **Format:** CSV with individuals as rows, variables as columns
- **Outcome:** Recidivism within 3 years (binary)
- **N observations:** ~15,000 individuals
- **Variables:** Demographic, criminal history, release conditions, outcome
- **Ethics:** De-identified, publicly available for research
- **Use:** Real-world causal discovery (no ground truth DAG available)

### Processed Data

#### `NIJ_lean_compact_onehot.csv`
- **Transformations applied:**
  1. **Encoding:** Categorical variables converted to one-hot encoding
  2. **Feature engineering:** Age bucketed into age ranges (age_at_release_18-25, age_at_release_26-35, etc.)
  3. **Dimensionality reduction:** Perfectly correlated features removed; multicollinearity addressed
  4. **Subset selection:** Variables with <5% variance or missing data >50% excluded
  5. **Standardization:** Continuous variables standardized (μ=0, σ=1)
  
- **Final dimensions:** ~35 variables × 15,000 observations
- **Variable list:** See `data/processed/metadata/NIJ_variables.txt`

---

## Experimental Pipeline

### Phase 1: CausalPitfalls Benchmarking
**Purpose:** Validate algorithms on synthetic data with known causal structures

**Procedure:**
1. Load each scenario from `CausalPitfallsData/`
2. Run all six algorithms with default hyperparameters
3. Compute structural metrics: SHD, TPR, FPR, FDR (vs. ground truth)
4. Output: `results/metrics/scores_causal_pitfalls.csv`

**Expected outputs:**
```
scenario | dataset | algo | TP | FP | FN | TN | SHD | TPR | FPR | FDR
```

### Phase 2: NIJ Real-World Causal Discovery
**Purpose:** Identify meaningful causal structures in recidivism data (no ground truth)

**Procedure:**
1. Load `NIJ_lean_compact_onehot.csv`
2. Run all six algorithms
3. Extract learned adjacency matrices
4. Visualize causal graphs (saved as PNG)
5. Compute self-consistency metrics (incompatibility score, edge stability)

**Key analyses:**
- **Edge stability via bootstrap resampling (n=20 resamples):**
  ```
  For each algorithm:
    For b in 1..20:
      Sample data with replacement from original dataset
      Run algorithm on bootstrap sample
      Count edge appearances
  Output: edge_frequency matrix (values 0-1, threshold 0.5+ = stable)
  ```

- **Incompatibility score (self-consistency test):**
  ```
  For k in [5, 10, 15]:
    For n_subsets in range(50):
      Randomly sample k variables from full set
      Run algorithm on subset
      Compare subset adjacency to full DAG restricted to k variables
      Compute SHD (structural Hamming distance)
  Output: avg SHD per subset size → % disagreement
  ```

### Phase 3: "Markov Blanket" Extraction
**Purpose:** Identify direct causes of recidivism (minimal sufficient set)

**Procedure (NIJ-specific):**
1. Identify target node: `Recidivism_Within_3years`
2. Extract parents of target in learned DAG
3. Post-hoc processing:
   - Merge one-hot age buckets using majority vote on edge direction
   - Prune graph to Markov blanket (target + parents only)
4. Visualize pruned graph
5. Output path example: `results/NIJ/graphs_DAGSLAM_PRUNED.png`

---

## Key Utility Functions

### Algorithm Wrappers
All follow the pattern: `df (pandas) → W (numpy array of shape d × d)`

```python
# DAGSLAM on mixed-type data
W_est = run_dagslam(df, lambda1=0.03, max_iter=100, w_threshold=0.25)

# Hill climbing with automatic score selection
adj, col_names = run_hc(df, seed=1)

# Linear non-Gaussian models
adj, cols = run_lim(df, seed=1)

# DAGBagM (R-based)
adj, cols = run_dagbagm(df, seed=1)

# ReX
adj, nodes = run_rex(df, experiment_name="exp_name")
```

### Evaluation Functions

```python
# Score learned DAG against ground truth (for synthetic data)
metrics = score_graph(W_est, W_true, labels_est, labels_true)
# Returns: dict with TP, FP, FN, TN, SHD, TPR, FPR, FDR

# Score PDAG (for FCI output)
metrics = score_pdag(W_pdag, W_true, labels_pdag, labels_true)
# Returns: skeleton metrics + orientation metrics

# Edge stability (bootstrap resampling)
edge_freq = bootstrap_edge_stability(df, B=20, seed=0)
# Returns: edge frequency matrix (d × d, values in [0,1])

# Self-consistency check (incompatibility score)
incompat = incompatibility_score(W_full, k=5, n_subsets=50, seed=0)
goodness_pct = goodness(incompat, k=5)  # % disagreement
```

### Visualization Functions

```python
# Draw networkx DiGraph as PNG using Graphviz
draw_graphviz_from_nx(G, out_path, engine="dot")

# Draw adjacency matrix as DAG
draw_graphviz_dag(W, out_path, node_labels=None, threshold=0.0)

# Draw using matplotlib/networkx
draw_graph(W, output_path, node_labels=None, threshold=0)
```

### Data Preprocessing

```python
# Encode non-numeric columns using categorical codes
df_enc = encode_mixed_df(df)

# Infer variable types for algorithm-specific loss functions
loss_type, m_vec = infer_type(df)  # For DAGSLAM
node_types = infer_type(df)        # For DAGBagM (returns R StrVector)
flags = infer_type(df)             # For LiM (returns flag array)
```

---

## Running Experiments

### Prerequisites

**Python environment:**
```bash
pip install pandas numpy scipy networkx matplotlib graphviz
pip install lingam causalexplain causaldag
```

**R environment (for HC, DAGBagM, FCI):**
```r
install.packages("bnlearn")
install.packages("dagbagM")
install.packages("causaldag")
```

**Python-R bridge:**
```bash
pip install rpy2
```

### Quick Start

#### 1. Benchmark on CausalPitfalls (Synthetic Data)
```python
import pandas as pd
from experimental_code_clean import run_dagslam, run_hc, score_graph

# Load a scenario with ground truth, e.g
data_path = "data/raw/CausalPitfallsData/casual_effect/device_failure_data.csv"
truth_path = "data/raw/CausalPitfallsData/casual_effect/device_failure_data_truth.csv"

df = pd.read_csv(data_path)
W_truth = pd.read_csv(truth_path, index_col=0).to_numpy()

# Run algorithms
W_dagslam = run_dagslam(df)
W_hc, _ = run_hc(df, seed=42)

# Evaluate the algorithms against ground truth
metrics_dagslam = score_graph(W_dagslam, W_truth, df.columns, df.columns)
metrics_hc = score_graph(W_hc, W_truth, df.columns, df.columns)

print(f"DAGSLAM SHD: {metrics_dagslam['SHD']}")
print(f"HC SHD: {metrics_hc['SHD']}")
```

#### 2. Causal Discovery on NIJ Data (Real-World)
```python
# Load processed NIJ data
df = pd.read_csv("data/processed/NIJ_lean_compact_onehot.csv")

# Run DAGSLAM (best for mixed types)
W_est = run_dagslam(df)

# Visualize
from experimental_code_clean import draw_graphviz_dag
draw_graphviz_dag(W_est, "results/NIJ/NIJ_DAGSLAM_graph.png", 
                  node_labels=list(df.columns))

# Bootstrap edge stability
from experimental_code_clean import bootstrap_edge_stability
edge_freq = bootstrap_edge_stability(df, B=20, seed=42)
print(f"Edges stable (freq >= 0.5): {(edge_freq >= 0.5).sum()}")
```

#### 3. Extract Markov Blanket (Target = Recidivism)
```python
import networkx as nx
from experimental_code_clean import draw_graphviz_from_nx

# Build DAG from adjacency matrix
G = nx.DiGraph()
for i, col_i in enumerate(df.columns):
    for j, col_j in enumerate(df.columns):
        if W_est[i, j] != 0:
            G.add_edge(col_i, col_j)

# Extract parents of target
target = "Recidivism_Within_3years"
parents = set(G.predecessors(target))
markov_blanket = parents | {target}

# Prune
G_mb = G.subgraph(markov_blanket).copy()
draw_graphviz_from_nx(G_mb, "results/NIJ/markov_blanket.png")

print(f"Recidivism parents: {parents}")
```

---

## Results & Outputs

### Metric Definitions

| Metric | Formula | Interpretation |
|--------|---------|-----------------|
| **SHD** | False Positives + False Negatives | Lower = closer to ground truth |
| **TPR** | TP / (TP + FN) | Sensitivity; recall of true edges |
| **FPR** | FP / (FP + TN) | False positive rate |
| **FDR** | FP / (TP + FP) | False discovery rate; precision |
| **Edge Frequency** | num resamples where edge appears / B | Robustness [0, 1] |
| **Incompatibility Score** | Avg SHD across random k-node subsets | Self-consistency; lower = better |

### Example Output Files

**`results/metrics/scores_causal_pitfalls.csv`**
```
scenario,dataset,algo,TP,FP,FN,TN,SHD,TPR,FPR,FDR
casual_effect,device_failure_data,DAGSLAM,5,2,1,42,3,0.833,0.045,0.286
casual_effect,device_failure_data,HC,4,3,2,42,5,0.667,0.068,0.429
```

**`results/NIJ/NIJ_DAGSLAM_edge_stability.csv`**
```
,Age_at_Release,Prior_Arrests,Employment_Status,...,Recidivism_Within_3years
Age_at_Release,0.0,0.0,0.15,...,0.85
Prior_Arrests,0.0,0.0,0.30,...,0.90
Employment_Status,0.20,0.0,0.0,...,0.65
```
(Values = proportion of 20 bootstrap resamples where edge appeared)

---

## Limitations & Caveats

### Algorithmic Limitations
1. **No ground truth for NIJ:** Cannot compute SHD, TPR, FPR; relying on edge stability and self-consistency metrics instead
2. **Causal sufficiency assumption:** Assumes no hidden confounders (violated in real criminal justice data)
3. **Acyclicity constraint:** Assumes DAG structure; may not capture feedback loops
4. **Sample size vs. dimensionality:** 15,000 observations across ~35 variables; moderate for structure learning

### Data Limitations
1. **Observational data only:** No interventional experiments; causal claims are associative + parametric assumptions
2. **De-identification:** Original variable names obscured; interpretation requires domain expertise
3. **Missing data handling:** Rows with NA dropped; potential bias if missingness is informative
4. **Temporal ordering:** No explicit time-based causal ordering (all variables treated as contemporaneous)

### Methodological Choices
1. **Encoding strategy:** One-hot encoding increases dimensionality; alternative encoding could yield different structures
2. **Markov blanket extraction:** Age bucketing merged buckets via majority vote; alternative aggregation rules were possible here

---

## Reproducibility

### Seeds
All randomized algorithms are seeded. To reproduce:
- DAGSLAM: Uses L-BFGS-B optimizer (deterministic after seed)
- HC (R): Set via `set.seed(seed)` before calling
- LiM: Uses `numpy.random.default_rng(seed)`
- Bootstrap resampling: Uses fixed seeds (42, 7, 12)

---

## References & Citations

**Algorithms:**
- Zhao, Y. & Jin, J. (2025). DAGSLAM: Causal Bayesian Network Structure Learning of Mixed Type Data.
- Scutari, M. (2010). Learning Bayesian networks with the bnlearn R package. *Journal of Statistical Software*, 35(3), 1–22.
- Zeng, Y., Shimizu, S., Matsui, H., & Sun, F. (2022). Causal discovery for linear mixed data. *CLeaR 2022*, PMLR 177, 994–1009.
- Chowdhury, S., Wang, R., Yu, Q. et al. DAGBagM: learning directed acyclic graphs of mixed variables with an application to identify protein biomarkers for treatment response in ovarian cancer. BMC Bioinformatics 23, 321 (2022). https://doi.org/10.1186/s12859-022-04864-y
- Kalainathan, D., & Goudet, O. (2019). Causal discovery toolbox: Uncover causal relationships in python. arXiv preprint arXiv:1903.02278
- Renero, J. (2024). CausalExplain: Explainable AI for causal discovery. https://github.com/renero/causalexplain

**Data:**
NIJ Dataset, available publicly here:
https://nij.ojp.gov/funding/recidivism-forecasting-challenge#2-0

**Benchmarks:**
- CausalPitfalls dataset suite (public benchmark) available from Kaggle here:
https://www.kaggle.com/datasets/cloverchen/causalpitfalls-benchmark-causal-data-neurips-2025

---

## Questions

For questions about specific experiments, algorithm implementations, or data preprocessing, please refer to:
- Project Specification: `reports/CS344_Project_Specification.pdf`
- Progress Report: `reports/CS344_Progress_Report.pdf`
- Code comments in `code/experimental_code_clean.py`

---

**Last updated:** April 2026  
