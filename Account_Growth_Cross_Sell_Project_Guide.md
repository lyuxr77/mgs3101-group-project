# Account growth and cross-sell: project guide

## Business scenario

A fictional technology company sells IaaS, SaaS, and professional services.
Its account managers want to understand which products existing customers
already purchase and identify reasonable opportunities to discuss additional
offerings.

**Business question:** Which existing accounts have product gaps, and what
account characteristics are associated with buying a new product?

All accounts, products, prices, and outcomes are synthetic. These are
Salesforce-style classroom files, not real Salesforce exports, import-ready
templates, or real-world recommendations. This is a separate scenario from
the other project datasets.

## Files and relationships

| File | One row per | Key |
|---|---|---|
| `Account_Growth_Accounts_2025.csv` | Existing fictional account at June 30, 2025 | `Account_ID` |
| `Account_Growth_Current_Holdings_2025.csv` | Product SKU held by an account at the snapshot | `Account_ID` + `SKU` |
| `Account_Growth_Cross_Sell_Outcomes_2025.csv` | Account outcome during the following period | `Account_ID` |
| `Account_Growth_Product_Catalog.csv` | Available SKU and offering category | `SKU` |
| `generate_account_growth_project_data.py` | Reproducible generator | Uses Python standard library and refuses to overwrite output files |

Join holdings to the catalog using `SKU`, and to accounts using `Account_ID`.
There are multiple holdings per account. Aggregate holdings to one row per
account before joining account-level attributes or outcomes, so account counts
and values are not multiplied. An account can own products in more than one
offering type; counts across types will overlap.

## Timing and outcome definition

- The account and holdings files describe what is known on **June 30, 2025**.
- The outcome window is **July 1 through December 31, 2025**, inclusive.
- `Cross_Sell_Purchased=1` means the account bought one additional SKU not
  already held at the snapshot during the window. `0` means no such purchase
  is recorded in this window; it does not mean the account will never buy.
- `New_SKU`, `New_SKU_Quantity`, `Purchase_Date`, and
  `New_Annual_Contract_Value_USD` describe that later purchase and are blank
  or zero when none occurred. They are **outcomes**, not snapshot predictors.
- This simplified simulation permits at most one new SKU per account in the
  outcome window. It does not model sales outreach, offers, declines, renewals,
  upgrades to existing products, or multiple later purchases.

## Field notes

- `Customer_Tenure_Months` is the account's age at the snapshot.
- `Product_Adoption_Percent` is a synthetic account-level adoption score, not
  usage of a specific SKU. `Customer_Health_Score` is partly derived from
  adoption and open support cases; these measures are therefore correlated.
- `Quantity` and `Annual_Contract_Value_USD` in the holdings file describe
  current contracted SKU units and annual values. The amount equals catalog
  annual price per unit times quantity. `New_Annual_Contract_Value_USD` uses
  the same rule for the later new SKU.
- Professional services are simulated annual contract values, **not recurring
  revenue**. Do not label totals that include services as ARR.
- IDs are join keys, not meaningful numeric predictors. Company names are
  fictional identifiers.

## Suggested analyses

1. Describe account counts, holdings, and current contract value by company
   size, industry, region, and offering type.
2. Identify accounts that hold one offering type but not another. Explain why
   a product gap is a **candidate for investigation**, not proof of need.
3. Calculate six-month cross-sell rates by account segment and current product
   mix, reporting both counts and rates. Compare which newly purchased SKUs
   appear in the outcomes.
4. Build a logistic regression to predict `Cross_Sell_Purchased` using only
   snapshot attributes and aggregated snapshot holdings. Compare against a
   baseline and evaluate a confusion matrix, precision, and recall on held-out
   accounts. For a real deployment, require data from multiple historical
   snapshots and evaluate on a later period.
5. Optionally fit linear regression to current recurring contract value using
   account attributes, excluding SKU prices, quantities, and totals that
   directly construct the target. Compare held-out MAE/RMSE with a baseline.
6. Recommend an account segment for further discovery and describe what
   customer needs, consent, product fit, and costs should be checked first.

## Leakage and interpretation

Do not use `New_SKU`, `Purchase_Date`, `New_Annual_Contract_Value_USD`, or
other outcome-window fields as features when forecasting at the snapshot.
Use one row per account, split before fitting preprocessing, and fit scaling,
imputation, and category encoding only on training data.

This dataset has a **single common snapshot** and one observation per
account, so it cannot support a genuine earlier-snapshot/later-snapshot
backtest. A held-out account split estimates performance on similar accounts
from this simulated cohort, not on future customers or other periods.

Relationships are generated for teaching and do not establish that adoption
or any intervention causes a purchase. Different accounts have different
available unowned SKUs, and a cross-sell purchase is not necessarily a
response to a sales campaign.
