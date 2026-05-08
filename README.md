# XG Cargo — M-Series Klaviyo Welcome Email (10% off)

Welcome email for the M-Series Magnetic Pod System sign-up flow. Triggered when an email subscribes through the on-site Klaviyo form (`Trt9Lj`) embedded on [m-series-magnetic-pod-system](https://xgcargo.com/pages/m-series-magnetic-pod-system) and joins list **`TnfDx6` — M Series Pods**.

Each recipient gets a **unique 10% off discount code** (Klaviyo pulls from a Shopify-generated pool, so codes can't be screenshot-shared).

## What's in this repo

| File | Purpose |
|------|---------|
| [`email.html`](email.html) | The actual email — table-based, dark theme, Klaviyo merge tags inline. **Copy this entire file's contents into Klaviyo's HTML source view.** |
| [`index.html`](index.html) | Vercel preview wrapper. Renders `email.html` in a faux email-client viewport with desktop/mobile toggle and a "Copy HTML" button. |
| [`vercel.json`](vercel.json) | Vercel static-deploy config. |

## Live preview

Vercel preview URL is on the GitHub repo's "deployments" tab once the first deploy lands.

## Setup checklist

The four pieces that need to exist for the flow to work end-to-end:

1. **Shopify discount code POOL** (so Klaviyo can hand out unique codes) — *Shopify side*
2. **Klaviyo "Coupon"** named exactly `Mpod` — *Klaviyo side*
3. **Klaviyo email template** (paste the HTML from `email.html`) — *Klaviyo side*
4. **Klaviyo flow** triggered on "Added to List → M Series Pods" — *Klaviyo side*

Total ~20 minutes if you have all four tabs open.

### Step 1 — Shopify discount pool (5 min)

1. Shopify admin → **Discounts → Create discount → Amount off products**
2. **Method:** Discount code
3. **Discount code:** leave the field empty for now — we'll generate the pool from Klaviyo. (Or pre-create one master code like `MSERIES10TEST` for verifying the link-format — it'll be replaced by the unique pool.)
4. **Type:** Percentage → **10%**
5. **Applies to:** Specific products → add **M5 Magnetic Pod**, **M7 Magnetic Pod**, **M12 Magnetic Tool Roll**, **Magnetic Pod System (Bundle)**
6. **Minimum purchase:** None
7. **Customer eligibility:** All customers (or "First-time customers" if you want to gate it harder)
8. **Maximum discount uses:**
   - ☑️ Limit to one use per customer
   - ☐ leave the total-uses limit unchecked — Klaviyo will create unique codes per recipient
9. **Active dates:** Start today, no end date (per-code expiry happens via Klaviyo's setting in step 2)
10. **Save**

> **Why not change the M5/M7/M12 prices to apply the discount automatically?** Per house rule, Shopify product prices are read-only — the 10% is delivered through a discount code, not by editing the variant price.

### Step 2 — Klaviyo coupon (5 min)

1. Klaviyo dashboard → **Coupons → Create coupon**
2. **Coupon name:** `Mpod` *(must match exactly — it's referenced 8× in the email HTML)*
3. **Integration:** Shopify → select your store
4. **Code prefix:** `MSERIES-` *(so codes look like `MSERIES-A3K9X2`)*
5. **Code length:** 6 characters
6. **Code type:** Alphanumeric
7. **Expiration:** **30 days** after the code is sent
8. **Max uses per customer:** 1
9. **Save**

Klaviyo will now create a code on demand each time the merge tag is rendered for a recipient.

### Step 3 — Email template (3 min)

1. Klaviyo dashboard → **Email Templates → Create template → HTML editor** (not drag-and-drop)
2. **Template name:** `M-Series Welcome — 10% off`
3. Open the live preview at the Vercel URL → click **Copy HTML** in the top bar
4. Paste into Klaviyo's HTML editor → **Save**
5. *(Optional)* In Klaviyo's preview, paste a sample profile to verify the coupon merge tag renders a real code

### Step 4 — Flow (5 min)

1. Klaviyo dashboard → **Flows → Create flow → Create from scratch**
2. **Flow name:** `M-Series — Welcome 10% off`
3. **Trigger:** *List* → **M Series Pods** (`TnfDx6`)
4. *(Recommended)* Add a **Filter:** `What someone has done (or not done) → Has not received this email in the last 90 days` — prevents re-sending if someone re-subs
5. *(Recommended)* Add a **Time Delay:** 5 minutes (lets the form-fill confirmation feel intentional, not robotic; bump higher if you want to soften the cadence)
6. Add an **Email** action:
   - **From:** XG Cargo `<hello@xgcargo.com>` *(or your sending address)*
   - **Subject:** `Your 10% off M-Series code is here ↓`
   - **Preview text:** `One-time code, expires in 30 days.`
   - **Template:** select `M-Series Welcome — 10% off`
   - **Smart Sending:** ON
7. Toggle the flow status to **Live**

### Step 5 — Verify end-to-end (2 min)

1. Open [m-series-magnetic-pod-system](https://xgcargo.com/pages/m-series-magnetic-pod-system) in an incognito window
2. Submit the form with a fresh test email
3. Confirm:
   - [ ] Email arrives within ~5 min (or whatever delay you set)
   - [ ] Coupon code in the email is a real `MSERIES-XXXXXX` value, not the literal `{% coupon_code 'Mpod' %}` placeholder
   - [ ] Clicking **Apply & Shop the Bundle** opens `xgcargo.com/discount/MSERIES-XXXXXX?redirect=/products/m-series-triple-bundle` and lands on the bundle PDP with the code applied
   - [ ] Adding the bundle to cart shows 10% off in the cart total
   - [ ] Code can't be used twice from the same customer

If the coupon shows as the literal merge-tag text, the coupon name in Klaviyo doesn't match `Mpod` exactly — fix it in the coupon settings, not the email HTML.

## Klaviyo merge tags used

| Tag | Where | Notes |
|-----|-------|-------|
| `{{ first_name|default:'there' }}` | Hero greeting | Falls back to "there" if no first name |
| `{% coupon_code 'Mpod' %}` | Code reveal block + every CTA URL | The unique code. Coupon name **must** match Klaviyo exactly |
| `{% web_view %}` | "View in browser" link | Klaviyo auto-generates |
| `{% unsubscribe_link %}` | Footer | Required for CAN-SPAM compliance |
| `{% manage_preferences_link %}` | Footer | Optional, recommended |

## Brand tokens (mirrors the M-Series landing page)

| Token | Value |
|-------|-------|
| Background | `#0a0a0a` |
| Surface | `#161616` |
| Body text | `#f5f5f7` / dim `#a1a1a6` |
| Accent | `#615a45` (button), `#d4c8a8` (text/headings) |
| Border | `#262626` |
| Display font | Helvetica Neue 900 / Arial Black uppercase + 0.02em tracking *(email-safe fallback for Monument Extended)* |

## Local preview

```bash
# from the repo root
python3 -m http.server 8080
# then open http://localhost:8080
```

## Image hosting

All product imagery loads directly from Shopify's CDN (`cdn.shopify.com/s/files/1/1982/0005/...`) so prices/photos stay live with the catalog and the repo stays small. If a product image gets re-versioned in Shopify, swap the `?v=` query param in `email.html`.
