# Storefront Analyzer

**Live:** https://storefront-analyzer.quick.shopify.io

A Quick-hosted internal tool for pulling key merchant metrics from BigQuery by Storefront ID. Built for fast merchant analysis without needing to write SQL.

---

## What it does

Enter a Shopify Storefront ID and the tool runs a set of parallel BigQuery queries to return:

### Shop Header
- Merchant name, country, currency, subscription plan
- Shopify Payments active/inactive badge
- Permanent domain (`X.myshopify.com`) and custom domain links
- Shop ID

### Core Metrics *(last 365 days)*
| Metric | Source |
|---|---|
| Total Turnover | `merchant_sales.order_sales_summary` |
| % Reaching Checkout | `buyer_activity.storefront_sessions_summary_v4` |
| % Converting | `buyer_activity.storefront_sessions_summary_v4` |
| AOV | `merchant_sales.order_sales_summary` |
| Total Sessions | `buyer_activity.storefront_sessions_summary_v4` |
| Total Orders | `merchant_sales.order_sales_summary` |

Also includes **5% and 10% conversion + AOV uplift scenarios** showing additional orders and revenue.

### Shopify Payments — Presentment Currencies
Breakdown of SP orders and volume by presentment currency. Flags if orders were checked out in a non-storefront currency (multi-currency indicator).

### Shopify Payments — Geographic Breakdown
SP volume split by:
- **Domestic** — card country matches shop country
- **EEA** — card country is in the European Economic Area
- **International / Amex** — all other cards and American Express

### All Payment Providers
Volume and order count per payment gateway with:
- **SP processing rates** by segment (Domestic / EEA / Int/Amex)
- **3P Shopify transaction fees** derived from plan and SP status:
  - Plus + SP active → 0% for all 3P
  - Plus + SP inactive → 0.2% for all 3P
  - Non-Plus + SP active → 0% for PayPal, standard plan rate for others
  - Non-Plus + SP inactive → standard plan rate for all 3P
- Reconciliation check against total turnover

### Multi-Currency Payouts
- Whether multi-currency payouts are enabled
- Active payout currencies (from payout destination countries)
- Current bank, bank country, SP account, payout-to-balance status

---

## Data sources

| Table | Used for |
|---|---|
| `accounts_and_administration.shop_profile_current` | Shop name, domain, currency, country |
| `accounts_and_administration.shop_billing_info_current` | Subscription plan |
| `merchant_sales.order_sales_summary` | GMV, orders, AOV |
| `buyer_activity.storefront_sessions_summary_v4` | Sessions, checkout funnel |
| `money_products.order_transactions_payments_summary` | SP presentment currencies, geo breakdown |
| `finance.gross_merchandise_volume` | All payment providers |
| `finance.currency_rate_daily_snapshot` | FX conversion to local currency |
| `money_products.shopify_payments_payout_destinations_summary_v1` | Payout destinations |
| `money_products.shopify_payments_fee_rates_current` | SP processing rates |
| `finance.transaction_billing_summary` | 3P transaction fee billing actuals |

All monetary values are converted to the shop's local currency via the most recent USD FX rate.

---

## Deploying

```bash
quick deploy /Users/fingordon/storefront-analyzer-quick storefront-analyzer -f
```

Requires the Quick CLI and `gcloud` authenticated as a Shopify employee.

---

## Files

```
storefront-analyzer-quick/
├── index.html   # Single-file app — all HTML, CSS, and JS
└── README.md    # This file
```
