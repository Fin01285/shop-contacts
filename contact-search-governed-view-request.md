# Governed data request — narrow shop → active staff contacts view (Contact-search-quick)

**Requestor:** findlay.gordon@shopify.com (Commercial / EMEA Core Cross-sell)
**Routed by:** River (seller data-access governance), 2026-07-21 — confirmed this is a
SEPARATE path from Payments (Lane 3) and from CRM MCP (Lane 1).
**Owner:** accounts_and_administration / dw-foundations-cda; help: help-data-warehouse.

---

## Why

Contact-search-quick (shopid-contacts.quick.shopify.io) resolves, for a given merchant, the
active staff users on the shop and orders them by most recent login. Under the 2026-07
narrowing its reads are denied:
- `accounts_and_administration.admin_shop_users_history` — staff-user membership / status
- `identity_account_current` — fresh last-login recency (admin_shop_users_history.last_login_at
  is mostly NULL/stale)
- `shop_profile_current` — shop context

These carry staff-user **email + last-login PII**, so per River I'm asking for a narrow
governed view rather than broad table access.

## What I'm asking for

A governed view keyed on `shop_id` returning **active (non-deactivated) staff contacts +
last_login_at** only:

- `shop_id`
- staff-user identifier + email (active/non-deactivated only)
- role / permission level on the shop (if cheaply available)
- `last_login_at` (fresh — from the identity source, not the stale history column)

## Constraints / scope

- Active staff only; exclude deactivated users.
- Readable under the Commercial seller approved-access path (same path Quick +
  personal `bq` auth use); no SMD-permit dependency.
- Single-merchant lookup use, not bulk export of the staff-user graph. Fine to rate-limit
  / require a shop_id filter (no unfiltered scan).
