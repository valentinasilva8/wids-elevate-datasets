# WiDS Elevate — Covenant National Bank Dataset

This repository contains the datasets for the WiDS Elevate P&L case
study: Project Vault at Covenant National Bank.

## What's Inside
datasets/covenant_pl/v2.1.0/

├── DATA_DICTIONARY.md       Full schema documentation for all datasets

├── raw_tables/              Source data tables (9 datasets)

└── prebuilt_datasets/       Pre-built modeling datasets for the hackathon

## Setup

### Option 1 — Clone this repo
```bash
git clone https://github.com/valentinasilva8/wids-elevate-datasets.git
cd wids-elevate-datasets/datasets/covenant_pl/v2.1.0/
```

### Option 2 — Download directly
Click the green **Code** button above → **Download ZIP**

### Loading in Google Colab
```python
import pandas as pd

# Raw tables
customers = pd.read_csv('raw_tables/customers.csv')
shadow_transactions = pd.read_csv('raw_tables/shadow_transactions.csv')
wallet_scores = pd.read_csv('raw_tables/wallet_scores.csv')

# Pre-built modeling datasets
model1_data = pd.read_csv('prebuilt_datasets/model1_dataset_history12m.csv')
model2_data = pd.read_csv('prebuilt_datasets/model_dataset_v4.csv')
```

## Data Dictionary
See `DATA_DICTIONARY.md` for full schema documentation including
field types, descriptions, joins, and the P&L framework.

## Case Study Materials
Case study PDF and model notebooks are distributed separately
via Google Drive: https://drive.google.com/drive/folders/1h1SkyktZBZ_j1SJTxFF0ASUfo0x-O-ZL

## Version
v2.1.0 — Student distribution. April 2026.
