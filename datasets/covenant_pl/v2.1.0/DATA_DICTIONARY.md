# Data Dictionary: covenant_pl v2.1.0

The following datasets support the analytical work required to evaluate
Project Vault. Students will use these data to build a propensity model,
assess fraud detection performance under label uncertainty, stress test
the Air Lock under adverse market conditions, and construct a pilot P&L.

Data Stored: https://drive.google.com/drive/folders/1h1SkyktZBZ_j1SJTxFF0ASUfo0x-O-ZL?usp=sharing

---

## Dataset 1: customers.csv

Customer-level static attributes for propensity modeling and cohort
selection. This is the foundation for Proof Point 1.

Records: 8,500 customers | Grain: One row per customer

| Field | Type | Description |
|---|---|---|
| customer_id | string | Primary key. Unique customer identifier. |
| account_open_date | date | Date customer opened first Covenant account. |
| age_band | string | Age range. Values: 18-24, 25-34, 35-44, 45-54, 55-64, 65+ |
| income_band | string | Household income range. Values: Under 50K, 50-100K, 100-150K, 150-200K, 200K+ |
| state | string | State of primary residence. Two-letter abbreviation. |
| segment | string | Customer segment. Values: Retail, Affluent, Wealth Management |
| advisor_assigned | boolean | True if customer has an assigned relationship manager or advisor. |
| churn_risk_score | float | Internal churn propensity score. 0.0 to 1.0. |

Notes:
- Product holdings, balances, and engagement metrics are time-varying
  and stored separately in customer_snapshot_monthly.csv (Dataset 2).
  Join on customer_id.
- Students should engineer features by joining customers.csv with
  customer_snapshot_monthly.csv and outflows.csv

---

## Dataset 2: customer_snapshot_monthly.csv

Monthly snapshots of customer product holdings, balances, and engagement
metrics. One record per customer per snapshot month.

Records: One row per customer per month
Grain: One row per customer per snapshot month
Snapshot range: January 2024 – February 2025

| Field | Type | Description |
|---|---|---|
| customer_id | string | Foreign key to customers.csv. |
| snapshot_month | date | First day of snapshot month. Format: YYYY-MM-DD. |
| product_count | integer | Number of Covenant products currently held. |
| has_checking | boolean | True if customer has checking account. |
| has_savings | boolean | True if customer has savings account. |
| has_investment | boolean | True if customer has investment account. |
| has_mortgage | boolean | True if customer has mortgage with Covenant. |
| has_credit_card | boolean | True if customer has Covenant credit card. |
| total_deposits_usd | float | Total deposit balances in USD. |
| total_investments_usd | float | Total investment AUM in USD. |
| monthly_login_count | float | Average logins per month over the snapshot period. |
| mobile_login_pct | float | Percentage of logins via mobile app. 0.0 to 1.0. |
| days_since_last_login | integer | Days since most recent digital login. |
| avg_monthly_transactions | float | Average transaction count per month. |
| volatile_equity_flag | boolean | True if customer holds a concentrated position in volatile equities. Risk tolerance proxy. |
| relationship_tenure_months | integer | Months since account_open_date as of snapshot_month. |

Notes:
- For propensity modeling, use the merged history table
  (customer_snapshot_monthly_with_history_pre2024.csv) which extends
  snapshots back before 2024 to support 12-month behavioral features
- The base file covers January 2024 – February 2025
- customer_snapshot_monthly_history_pre2024.csv contains synthetic
  snapshots for months prior to 2024 — same schema, pipeline input only
- customer_snapshot_monthly_with_history_pre2024.csv is the merged
  table used to build the Model 1 modeling dataset. Students building
  the propensity model from scratch should use this merged table.
  Students using the pre-built model1_dataset_history12m.csv can
  ignore both history files.

---

## Dataset 3: bank_txn_summary_monthly.csv

Monthly aggregated bank transaction summaries per customer. Supports
behavioral feature engineering for the propensity model alongside
customer_snapshot_monthly.csv.

Records: One row per customer per month
Grain: One row per customer per snapshot month
Time Period: January 2024 – February 2025

| Field | Type | Description |
|---|---|---|
| customer_id | string | Foreign key to customers.csv. |
| snapshot_month | date | First day of snapshot month. Format: YYYY-MM-DD. |
| ach_outflow_count | integer | Number of ACH outflows in the month. |
| wire_outflow_count | integer | Number of wire outflows in the month. |
| external_outflow_count | integer | Number of external outflows (e.g., to crypto exchanges) in the month. |
| ach_outflow_amount_usd | float | Total ACH outflow amount in USD. |
| wire_outflow_amount_usd | float | Total wire outflow amount in USD. |
| external_outflow_amount_usd | float | Total external outflow amount in USD. |
| inbound_transfer_count | integer | Number of inbound transfers in the month. |
| inbound_transfer_amount_usd | float | Total inbound transfer amount in USD. |
| total_txn_count | integer | Total transaction count for the month. |

Notes:
- Join to customers.csv on customer_id and to
  customer_snapshot_monthly.csv on customer_id + snapshot_month
  for a full behavioral feature set
- bank_txn_summary_monthly_history_pre2024.csv contains synthetic
  transaction history for months prior to 2024 — same schema,
  pipeline input only
- bank_txn_summary_monthly_with_history_pre2024.csv is the merged
  table used to build the Model 1 modeling dataset. Use this merged
  table when building the propensity model from scratch.

---

## Dataset 4: labels.csv

Outflow labels for customers who transferred assets to external crypto
exchanges. This is the label source for the propensity model in
Proof Point 1.

Records: 2,805 customers | Grain: One row per customer who has outflowed

| Field | Type | Description |
|---|---|---|
| customer_id | string | Foreign key to customers.csv. |
| first_outflow_date | date | Date of first confirmed outflow to a crypto exchange. |
| count_outflows_total | integer | Total number of outflow transactions recorded for this customer. |

Notes:
- Only customers with at least one confirmed outflow appear in this
  table — it is not a complete customer list
- first_outflow_date is the label anchor for Model 1. A customer is
  labeled y=1 at snapshot month T if their first_outflow_date falls
  within the 120-day window after T
- Label source is outflows.csv transaction dates

---

## Dataset 5: outflows.csv

Transaction-level detail of ACH and wire transfers to known
cryptocurrency exchanges. Supports feature engineering for the
propensity model.

Records: 65,778 transactions
Grain: One row per outflow transaction
Time Period: January 2024 through February 2025

| Field | Type | Description |
|---|---|---|
| outflow_id | string | Primary key. Unique transaction identifier. |
| customer_id | string | Foreign key to customers.csv. |
| transaction_date | date | Date of transfer. |
| amount_usd | float | Transfer amount in USD. |
| descriptor | string | Payment descriptor. Values: COINBASE, KRAKEN, GEMINI, BINANCE.US, CRYPTO.COM, OTHER_EXCHANGE |
| transaction_type | string | Transfer method. Values: ACH, WIRE |

Notes:
- Descriptors are standardized from raw payment data
- OTHER_EXCHANGE captures smaller platforms
- Students can aggregate to customer level for feature engineering

---

## Dataset 6: shadow_transactions.csv

Shadow production transaction log from February 15 through March 14,
2025. Contains model scores, decisions, and partial fraud labels.
This is the core dataset for Proof Point 2 and P&L construction.

Records: 45,000 transactions
Grain: One row per crypto transaction attempt
Time Period: 30 days of shadow production

| Field | Type | Description |
|---|---|---|
| transaction_id | string | Primary key. Blockchain transaction hash. |
| customer_id | string | Foreign key to customers.csv. |
| timestamp | datetime | Transaction timestamp in UTC. |
| asset_type | string | Cryptocurrency. Values: BTC, ETH, LTC |
| direction | string | Transaction direction. Values: buy, sell |
| amount_usd | float | Transaction value in USD. |
| quantity | float | Cryptocurrency quantity. |
| execution_price | float | Price per unit at execution in USD. |
| base_spread_bps | float | Base spread charged in basis points. |
| dynamic_spread_bps | float | Additional spread due to volatility regime. |
| total_spread_bps | float | Total spread charged. base_spread_bps + dynamic_spread_bps. |
| dest_wallet | string | Destination wallet address. Foreign key to wallet_scores.csv. |
| device_fingerprint | string | Hashed device identifier. |
| device_type | string | Device category. Values: mobile_ios, mobile_android, desktop_web, desktop_app |
| is_new_device | boolean | True if device not previously seen for this customer. |
| session_duration_sec | integer | Length of session in seconds. |
| ip_country | string | Two-letter country code from IP geolocation. |
| model_score | float | Fraud model probability score. 0.0 to 1.0. |
| model_decision | string | Air Lock decision. Values: approve, review, block |
| volatility_regime | string | Market regime at transaction time. Values: normal, elevated, high, critical |
| fraud_label | boolean or null | True if confirmed fraud. False if confirmed legitimate. Null if not yet determined. |
| label_date | date or null | Date fraud label was confirmed. Null if label unavailable. |
| fraud_type | string or null | If fraud, the category. Values: account_takeover, synthetic_identity, transaction_laundering, authorized_push_payment, null |
| loss_amount_usd | float or null | If fraud, actual loss amount. Null otherwise. |

Label Availability Logic:
- Transactions older than 60 days from March 14: approximately 75% labeled
- Transactions 30-60 days old: approximately 30% labeled
- Transactions less than 30 days old: approximately 5% labeled

Notes:
- This structure forces students to grapple with the label availability
  problem described in the case
- model_decision reflects Air Lock output but was not enforced in
  shadow mode
- Students must calculate P&L using spread revenue from approved
  transactions and confirmed fraud losses

---

## Dataset 7: market_conditions.csv

Hourly market data for BTC, ETH, and LTC. Includes a 36-hour stress
event. Supports Proof Point 2 and stress testing the Air Lock.

Records: 2,016 rows
Grain: One row per asset per hour
Time Period: February 15 through March 14, 2025

| Field | Type | Description |
|---|---|---|
| timestamp | datetime | Hour timestamp in UTC. |
| asset | string | Cryptocurrency. Values: BTC, ETH, LTC |
| price_usd | float | Spot price in USD. |
| price_change_1h_pct | float | Price change over prior hour. Decimal format. |
| price_change_24h_pct | float | Price change over prior 24 hours. Decimal format. |
| volatility_1h_ann | float | Realized volatility over prior hour, annualized. Decimal format. |
| volatility_24h_ann | float | Realized volatility over prior 24 hours, annualized. Decimal format. |
| bid_ask_spread_bps | float | Bid-ask spread in basis points. |
| order_book_depth_usd | float | Total order book depth within 1% of mid price in USD. |
| liquidity_score | float | Composite liquidity score. 0 to 100. |
| avg_confirmation_time_min | float | Average block confirmation time in minutes. |
| mempool_pending_tx | integer | Number of pending transactions in mempool. BTC and LTC only. |
| gas_price_gwei | float | Gas price in gwei. ETH only. |
| regime | string | Volatility regime classification. Values: normal, elevated, high, critical |

Stress Event:
- March 7, 2025 06:00 UTC through March 8, 2025 18:00 UTC (36 hours)
- BTC drops 27% from $67,400 to $49,200
- ETH drops 31% from $3,840 to $2,650
- LTC drops 34% from $98 to $65
- Volatility spikes to 140%+ annualized
- Liquidity scores drop below 25
- Bid-ask spreads widen 4-5x
- Regime classified as "critical" for 18 hours

Notes:
- Students should use this data to evaluate Air Lock behavior during stress
- The stress event tests whether dynamic spread pricing and halt logic
  would have protected the bank

---

## Dataset 8: wallet_scores.csv

Risk scores for destination wallets appearing in shadow transactions.
Simulates Chainalysis integration.

Records: 12,400 unique wallets | Grain: One row per wallet address

| Field | Type | Description |
|---|---|---|
| wallet_address | string | Primary key. Blockchain wallet address. |
| risk_score | integer | Risk score from 0 (lowest risk) to 100 (highest risk). |
| risk_category | string | Risk classification. Values: low, medium, high, severe |
| first_seen_date | date | Date wallet first appeared in blockchain data. |
| total_transaction_count | integer | Historical transaction count for wallet. |
| known_entity | string or null | Entity attribution if known. Examples: "Coinbase Hot Wallet", "Kraken", "Unknown", "Mixer Service", "Darknet Market" |
| sanctioned_flag | boolean | True if wallet associated with sanctioned entity. |
| score_date | date | Date risk score was calculated. |

Risk Category Thresholds:
- low: 0–25
- medium: 26–50
- high: 51–75
- severe: 76–100

Notes:
- Approximately 0.3% of wallets have sanctioned_flag = true
- Approximately 2.1% of wallets have risk_category = severe
- known_entity is null for approximately 78% of wallets
- Not all destination wallets in shadow_transactions.csv appear in
  this table. Missing wallets represent novel or unscored addresses.
- 20% of wallets (2,480 of 12,400) have risk_score, risk_category,
  sanctioned_flag, and total_transaction_count intentionally set to
  null. wallet_address is preserved. Handling this missing data is
  a core exercise in Proof Point 2.

---

## Dataset 9: settlement_data.csv

Blockchain settlement telemetry. Supports Proof Point 3 on operational
differences across assets.

Records: 45,000 rows (one per shadow transaction)
Grain: One row per transaction

| Field | Type | Description |
|---|---|---|
| transaction_id | string | Primary key. Foreign key to shadow_transactions.csv. |
| asset_type | string | Cryptocurrency. Values: BTC, ETH, LTC |
| initiated_timestamp | datetime | When transaction was broadcast to network. |
| expected_confirmation_min | float | Expected confirmation time based on asset and network conditions. |
| actual_confirmation_min | float | Actual time to first confirmation. |
| confirmations_required | integer | Number of confirmations required by Covenant policy. |
| confirmations_received | integer | Number of confirmations received. |
| final_settlement_timestamp | datetime | When required confirmations were reached. |
| settlement_status | string | Final status. Values: confirmed, pending, failed |
| delay_flag | boolean | True if actual_confirmation_min exceeded expected_confirmation_min by more than 50%. |
| delay_reason | string or null | If delayed, the cause. Values: network_congestion, low_fee, block_variance, null |

Expected Confirmation Baselines:
- BTC: 10 minutes per confirmation, 3 confirmations required
- ETH: 15 seconds per confirmation, 12 confirmations required
- LTC: 2.5 minutes per confirmation, 6 confirmations required

Notes:
- During the stress event, confirmation times increase significantly
- Students should analyze settlement risk differences across assets
- delay_flag indicates transactions where settlement uncertainty
  created operational exposure

---

## P&L Framework

Students should construct the pilot P&L using the following logic.

Revenue Components:

| Component | Calculation |
|---|---|
| Spread Revenue | sum(amount_usd × total_spread_bps / 10000) for approved transactions only |
| Custody Fees | average_daily_AUM × custody_rate_bps × days / 36500 |

Cost Components:

| Component | Calculation |
|---|---|
| Fraud Losses | sum(loss_amount_usd) where fraud_label = true |
| Manual Review Costs | count(model_decision = 'review') × cost_per_review |
| Hedging Costs | function of volatility regime and gross position |
| Infrastructure Costs | fixed monthly cost + variable per transaction |

Suggested Assumptions:
- Custody rate: 45 bps annually
- Cost per manual review: $87 (4.2 analyst hours × $20.71 loaded hourly rate)
- Hedging cost: 15 bps of notional in normal regime, 45 bps in
  elevated, 90 bps in high, 150 bps in critical
- Infrastructure: $25,000 fixed monthly + $0.35 per transaction

Net Pilot P&L = Spread Revenue + Custody Fees − Fraud Losses −
Manual Review Costs − Hedging Costs − Infrastructure Costs

---

## Joins & Relationships
customers (customer_id) [master]
├── bank_txn_summary_monthly.customer_id
├── bank_txn_summary_monthly_with_history_pre2024.customer_id
├── customer_snapshot_monthly.customer_id
├── customer_snapshot_monthly_with_history_pre2024.customer_id
├── labels.customer_id
├── outflows.customer_id
└── shadow_transactions.customer_id
shadow_transactions (transaction_id)
└── settlement_data.transaction_id
wallet_scores (wallet_address)
└── shadow_transactions.dest_wallet
(not all dest_wallets have a matching wallet_scores row —
missing wallets are unscored addresses)

- customers.csv is the central dimension table. All customer_id
  references resolve to it.
- shadow_transactions and settlement_data share transaction_id.
  settlement_data is a subset — one row per transaction.
- wallet_scores covers a subset of destination wallets. Transactions
  to unscored wallets will produce nulls on join — handling this null
  case is a core modeling decision in Proof Point 2.

---

## Pre-Built Modeling Datasets

The following datasets are derived artifacts built from the raw tables
above. They are pre-built and included in the repository for the
hackathon format, where time constraints make constructing them from
scratch impractical. In other contexts, these are distributed
separately alongside their respective model code.

### model1_dataset_history12m.csv

Pre-built modeling dataset for the customer propensity model
(Proof Point 1).

Records: 62,924 rows
Grain: One row per eligible customer per snapshot month
Snapshot range: January 2024 – February 2025

Notes:
- Eligibility filters applied: tenure ≥ 90 days at snapshot month T,
  observability (T + 120 days ≤ 2025-02-26), at-risk status (no prior
  outflow before T), 12 months of behavioral history available
- Label: y = 1 if the customer's first crypto outflow date falls within
  the 120-day window after T. Derived from labels.csv only.
- Time splits use forward chaining — no future data leaks into training
- Students using this dataset can proceed directly to feature selection
  and model training

### model_dataset_v4.csv

Pre-built modeling dataset for the Air Lock fraud detection model
(Proof Point 2).

Records: 45,000 rows | Grain: One row per shadow transaction

Notes:
- Wallet scores are intentionally incomplete. 20% of wallets
  (2,480 of 12,400) have risk_score, risk_category, sanctioned_flag,
  and total_transaction_count set to null. wallet_address is preserved.
- Nulling is not random. Wallets were ranked by three criteria:
  appearing in a stress window transaction (March 7–8), appearing in
  a transaction above $8,000, and appearing in an ETH transaction.
  The top 20% by that ranking were nulled. This concentrates missing
  data where it matters most.
- Approximately 10,844 of 45,000 transaction rows will have null wallet
  signal columns. This is the missing-data problem students must solve.
- Students must decide how to handle null wallet scores before scoring
  transactions. The three approaches produce materially different
  outcomes — this decision is the core exercise of Proof Point 2.
- model_decision column is dropped. Students recompute decisions from
  model_score using their chosen thresholds.
- The trained model was built on full wallet coverage. No separately
  retrained model exists for the student dataset. Imputation happens
  at inference time — which is the realistic production scenario.
