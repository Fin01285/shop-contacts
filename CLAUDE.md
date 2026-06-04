# ShopID Contacts — Claude Instructions

## Project overview
Single-file Quick app (`index.html`) deployed at https://shopid-contacts.quick.shopify.io. Looks up store staff/contacts from BigQuery by Storefront ID. All HTML, CSS, and JavaScript lives in `index.html`.

## Architecture
- **Platform:** Quick (Shopify's internal static hosting). Uses `quick.js` client for BigQuery access via `quick.dw.querySync()`.
- **No build step.** Edit `index.html` directly and deploy.
- **No backend.** All queries run client-side through the Quick BigQuery API.

## Deploying
```bash
quick deploy "/Users/fingordon/Contact-search-quick " shopid-contacts -f
```

## BigQuery tables
| Table | Purpose |
|---|---|
| `accounts_and_administration.admin_shop_users_history` | Store staff — account owner and contacts |
| `accounts_and_administration.shop_profile_current` | Shop name (`name` column) |
| `accounts_and_administration.identity_account_current` | Fresher login recency (`last_login_at`, `last_seen_at`), joined to staff by `email` |

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
- Displays up to 5 non-owner contacts. **Regular users (`user_type = 'RegularUser'`) are prioritised** — the most recently signed-in regular users fill the 5 slots first. Only if there are fewer than 5 regular users do other types (Collaborator, CollaboratorTeamMemberUser, InvitedUser, etc.) fill the remainder. Within each priority group, ordering is by login recency DESC.
- Selects `first_name`, `last_name`, `email`, `is_account_owner`, `user_type`
- **Login recency for ordering** comes from `GREATEST(admin_shop_users_history.last_login_at, identity_account_current.last_login_at, identity_account_current.last_seen_at)`. The `admin_shop_users_history.last_login_at` column alone is NULL for ~58% of current users and years stale for most of the rest, so the query LEFT JOINs `identity_account_current` (CTE `ident`, aggregated `MAX … GROUP BY LOWER(email)` to collapse the ~4% of emails mapping to multiple identities) to recover a much fresher signal — roughly doubles coverage and is current to within hours/days rather than years.
- Login recency is used only for ordering; it is **not displayed** in the UI.
- Cost note: the identity join scans ~9.4 GB per lookup (email isn't a clustering key on either table). Acceptable for light internal use. For maximum freshness, the event-level `identity_login_sessions` table exists (bridge via `identity_uuid`) but is heavier and not used.
- Each successful lookup logs a usage event to `quick.db` collection `usage_events` with `user_email`, `user_name`, `shop_id`, `ts`

## Manager view
- Accessible via the "📊 Manager view" button in the header
- Opens as an overlay showing a bar chart of total lookups per user, sorted highest to lowest
- Data comes from the `usage_events` quick.db collection
