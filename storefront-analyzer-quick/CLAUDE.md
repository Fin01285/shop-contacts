# ShopID Analyser — Claude Instructions

## Project overview
Single-file Quick app (`index.html`) deployed at https://shopid-analyser.quick.shopify.io. It queries Shopify's internal BigQuery data warehouse and displays merchant metrics by Storefront ID. All HTML, CSS, and JavaScript lives in `index.html`.

## Architecture
- **Platform:** Quick (Shopify's internal static hosting). Uses `quick.js` client for BigQuery access via `quick.dw.queryAndWait()`.
- **No build step.** Edit `index.html` directly and deploy.
- **No backend.** All queries run client-side through the Quick BigQuery API.

## Deploying
```bash
quick deploy /Users/fingordon/storefront-analyzer-quick shopid-analyser -f
```
Always deploy after making changes. Requires `gcloud` authenticated as a Shopify employee.

## Key BigQuery tables
| Table | Purpose |
|---|---|
| `accounts_and_administration.shop_profile_current` | Shop name, `domain`, `permanent_domain`, currency, country, plan |
| `accounts_and_administration.shop_billing_info_current` | Subscription plan name |
| `merchant_sales.order_sales_summary` | GMV, orders, AOV |
| `buyer_activity.storefront_sessions_summary_v4` | Sessions and checkout funnel |
| `money_products.order_transactions_payments_summary` | SP presentment currencies and geo breakdown |
| `finance.gross_merchandise_volume` | All payment providers and volume |
| `finance.currency_rate_daily_snapshot` | USD → local FX rates |
| `money_products.shopify_payments_payout_destinations_summary_v1` | Payout destinations |
| `money_products.shopify_payments_fee_rates_current` | SP processing rates by segment |
| `finance.transaction_billing_summary` | 3P transaction fee billing actuals |

All monetary values are converted to the shop's local currency using the most recent USD FX rate.

## Important domain fields
- `domain` — the merchant's custom domain (e.g. `paleobull.com`)
- `permanent_domain` — the myshopify.com URL (e.g. `paleobull.myshopify.com`) — use this when the permanent/myshopify URL is needed

## 3P transaction fee logic
- Plus + SP active → 0% for all 3P providers
- Plus + SP inactive → 0.2% for all 3P providers
- Non-Plus + SP active → 0% for PayPal only; standard plan rate for all others
- Non-Plus + SP inactive → standard plan rate for all 3P providers
- Standard rates: Basic = 2.0%, Shopify = 1.0%, Advanced = 0.5%, Plus = 0.2%

## EEA countries
Used in the geographic breakdown for SP transactions:
`AT, BE, BG, HR, CY, CZ, DK, EE, FI, FR, DE, GR, HU, IS, IE, IT, LV, LI, LT, LU, MT, NL, NO, PL, PT, RO, SK, SI, ES, SE`
