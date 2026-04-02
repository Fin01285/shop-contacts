# Storefront Contacts — Claude Instructions

## Project overview
Single-file Quick app (`index.html`) deployed at https://storefront-contacts.quick.shopify.io. Looks up store staff/contacts from BigQuery by Storefront ID. All HTML, CSS, and JavaScript lives in `index.html`.

## Architecture
- **Platform:** Quick (Shopify's internal static hosting). Uses `quick.js` client for BigQuery access via `quick.dw.querySync()`.
- **No build step.** Edit `index.html` directly and deploy.
- **No backend.** All queries run client-side through the Quick BigQuery API.

## Deploying
```bash
quick deploy /Users/fingordon/storefront-contacts-quick storefront-contacts -f
```

## BigQuery tables
| Table | Purpose |
|---|---|
| `accounts_and_administration.admin_shop_users_history` | Store staff — account owner and contacts |
| `accounts_and_administration.shop_profile_current` | Shop name (`name` column) |

### Columns used
| Column | Description |
|---|---|
| `shop_id` | Storefront / shop ID (filter key) |
| `first_name` | First name (policy-tagged, requires SDP-PII permit) |
| `last_name` | Last name (policy-tagged, requires SDP-PII permit) |
| `email` | Email address (policy-tagged, requires SDP-PII permit) |
| `is_account_owner` | Boolean — `TRUE` identifies the account owner |
| `is_current` | Boolean — filter to current staff only |
| `user_type` | User role — e.g. regular user or collaborator |
| `last_login_at` | Timestamp of most recent login — **not accurate**, data is stale. Not displayed in the UI. |

## Permissions
This tool requires the **SDP-PII permit** to access the policy-tagged name and email columns (`first_name`, `last_name`, `email`) in `admin_shop_users_history`.
- Apply at: https://clouddo.shopify.io/permits?claim=sdp-pii
- The permit is valid for **8 hours**, after which a new one must be applied for
- Without the permit, queries fail with a 403 `OTHER_INDIRECT_IDENTIFIER` / `EMAIL_ADDRESS` policy tag error

## Logic
- Fetches shop name from `shop_profile_current` (`name` column)
- Identifies the account owner by `is_account_owner = TRUE`
- Displays up to 5 non-owner contacts ordered by `MAX(last_login_at) DESC`
- Selects `first_name`, `last_name`, `email`, `is_account_owner`, `user_type`
- Last login is not displayed — data in BQ is stale (years behind live DB)
- Each successful lookup logs a usage event to `quick.db` collection `usage_events` with `user_email`, `user_name`, `shop_id`, `ts`

## Manager view
- Accessible via the "📊 Manager view" button in the header
- Opens as an overlay showing a bar chart of total lookups per user, sorted highest to lowest
- Data comes from the `usage_events` quick.db collection
