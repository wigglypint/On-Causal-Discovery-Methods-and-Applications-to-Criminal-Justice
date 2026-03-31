# Causal Discovery Experiments on NIJ Recidivism Data

This repository contains a cleaned and consolidated version of the code and data used for my causal discovery experiments on the Causal Pitfall and NIJ recidivism datasets.

## Repository structure

```text
.
├── algorithms/
│   ├── causalexplain/
│   ├── dagslam/
│   └── lingam/
├── code/
├── data/
│   ├── processed/
│   └── raw/
└── results/
│   ├── graphs_DAGBagM/
│   ├── graphs_DAGSLAM/
│   ├── graphs_fci/
│   ├── graphs_HC/
│   ├── graphs_LiM/
│   ├── graphs_ReX/
│   ├── NIJ/
│   │   ├── graphs_DAGBagM/
│   │   ├── graphs_DAGSLAM/
│   │   ├── graphs_fci/
│   │   ├── graphs_HC/
│   │   ├── graphs_LiM/
│   │   └── graphs_ReX/
└───└── scores/
│
└── reports/
```

## `algorithms/ `
This folder contains forked GitHub repositories of the original implementations of causal discovery algorithms used, including:
- `causalexplain/` Renero, J. (2026). CausalExplain (Version 0.9.1). Available at: https://github.com/renero/causalexplain

- `dagslam/`
Zhao, Y. Jinzhu, J. (2025). DAGSLAM. Available at: https://github.com/yuanyuan-zhao-pku/DAGSLAM
- `lingam/`
Code authors: T. Ikeuchi, M. Ide, Y. Zeng, T. N. Maeda, and S. Shimizu. (2023) Python package for causal discovery based on LiNGAM. Journal of Machine Learning Research, 24(14): 1−8, 2023. Available at: https://github.com/cdt15/lingam.
Based on the LiM algorithm as developed by:
Y. Zeng, S. Shimizu, H. Matsui, F. Sun. (2022) Causal discovery for linear mixed data. In Proc. First Conference on Causal Learning and Reasoning (CLeaR2022). PMLR 177, pp. 994-1009, 2022.

## `code/`
This folder contains python notebooks used for data preprocessing, as well as data loading, cleaning and transformation functions. Wrappers are used to run each algorithm and to standardise inputs and outputs: data df -> estimated adjacency matrix. Experimental setups are also programmatically laid out here.

## `data/`
Contains data for experiments. `raw/` contains Causal Pitfall datasets as well as the original NIJ dataset. `processed/` contains processed versions of the NIJ dataset, after adjusting for colinearity, one hot encoding, bucketing/dimension reduction, and variable subset.

## `results/`
Stores all graphical outputs from running each causal discovery algorithm on the causal pitfall and NIJ datasets as well as experiments. E.g NIJ edge stability. `scores/` stores the SHD, FPR, TPR etc statistics for each algorithm on each scenario.

## `reports/` 
Houses all written components relevant to the dissertation submission.
