# ED-TRACE: Equity Derivatives Trade Lifecycle & Control Engine

An automated, vectorized Front-to-Back (FO/BO) trade reconciliation, Exposure at Risk (EaR) quantification, and exception-routing engine built in Python. Designed to simulate T+0 Middle Office, Product Control, and Trading Services workflows across Equity Index Options, Total Return Swaps (TRS), and Stock Loans.

## Architecture & Core Modules

### Module 1: Synthetic Trade & Break Simulator (`SyntheticTradeGenerator`)
Generates reproducible Front Office trade blotters and simulates downstream Back Office settlement feeds with controlled operational breaks across institutional counterparties:
* **Missing Trade Captures:** Simulates dropped MQ messages between FO execution and BO settlement systems.
* **Valuation / MTM Discrepancies:** Injects pricing surface/mark divergences exceeding £1,000 tolerance thresholds.
* **ISDA Accrual Basis Mismatches:** Simulates `Actual/365` vs. `30/360` day-count convention discrepancies on swap and stock loan legs.
* **Unconfirmed Counterparty Status:** Flags trades pending affirmation via MarkitWire/SWIFT.

### Module 2: Vectorized Reconciliation & Exposure at Risk (`TradeReconciler`)
Replaces legacy iterative row loops (`iterrows()`) with compiled NumPy/Pandas boolean masking and `np.select` decision trees to scale across institutional blotter volumes:
* **Outer-Join Alignment:** Reconciles FO and BO feeds on `Trade_ID` while preserving orphaned records.
* **Exposure at Risk (EaR) Quantification:**
  * *Missing / Unconfirmed Trades:* 100% of absolute Front Office MTM (`|MTM_GBP_FO|`).
  * *MTM Discrepancies:* Absolute valuation delta (`|MTM_GBP_FO - MTM_GBP_BO|`).
  * *Accrual Mismatches:* 5-day basis spread accrual proxy (`Notional * 5% * 5/365`).
* **Aging & Severity Escalation:** Automatically flags exceptions as `CRITICAL` when Exposure at Risk exceeds £50,000, a trade is unbooked in BO, or break aging exceeds 5 days.

### Module 3: Automated Triage & SLA Routing (`ResolutionRouter`)
Maps prioritized exceptions to target operational control desks, assigns resolution SLAs (2h for dropped captures, 4h for Critical, 24h for High), and exports structured audit payloads (`ed_control_exceptions_pack.csv`).

## Benchmark Control Output (100-Trade Run)

| Assigned Control Desk | Open Tickets | Critical Count | Desk Exposure at Risk (GBP) | Primary Remediation Action |
| :--- | :---: | :---: | :---: | :--- |
| **Valuation Control (IPV) & Product Control** | 5 | 5 | £451,972.56 | Compare FO vol surface / curve marks against BO EOD snap |
| **Client Confirmations & Legal Settlements** | 5 | 4 | £203,759.54 | Chase counterparty affirmation via MarkitWire / SWIFT |
| **Middle Office Trade Capture** | 2 | 2 | £159,794.61 | Re-trigger MQ trade message from FO blotter (2h SLA) |
| **Static Data & Middle Office Operations** | 5 | 1 | £11,473.62 | Align day-count convention against master ISDA/CSA |
| **Total Portfolio Exceptions** | **17** | **12** | **£827,000.33** | |

## Tech Stack & Quickstart

* **Core Stack:** Python 3, Pandas, NumPy, Jupyter
* **Execution:**
  ```bash
  python3 -m venv venv
  source venv/bin/activate
  pip install pandas numpy jupyter
  jupyter notebook ed_trace_pipeline.ipynb