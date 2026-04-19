# Deadline Signs — Full Site Audit & Rebuild Spec

**Audit date:** 2026-04-18 (Session 19)
**Audited by:** Claude Code (5 parallel deep-dive agents, 3 passes)
**Purpose:** Complete specification for rebuilding deadlinesigns.com as a custom Next.js + PostgreSQL application

---

## Executive Summary

Deadline Signs is a B2B print and sign shop in Concord, NC generating ~$50k/month (trending up) across two storefronts:

- **deadlinesigns.com** — main store (~$40k/mo, ~80% of revenue)
- **reseller.deadlinesigns.com** — gated reseller portal (~$10k/mo, ~20%)

The site runs WordPress 5.0.4 / WooCommerce 3.6.2 / PHP 7.4 — all seven years out of date. The site also serves as the company's **POS system** — staff use "Shop as Customer" to build quotes and place orders for walk-in/phone customers.

### Key Numbers

| Metric | Value |
|--------|-------|
| Live products | 89 (1,810 drafts) |
| Product variations | 525 |
| Total orders (all time) | 8,315 (5,776 main + 2,539 reseller) |
| Total users | 4,024 |
| Users who've ordered | 1,836 |
| Total quotes | 5,109 |
| Database size | 2.3 GB (bloated — real data ~400 MB) |
| Order date range | April 2019 – April 2026 |

---

## 1. Product System

### 1.1 Product Types (9 distinct types)

The site has 9 product types, each with unique pricing logic, admin UI, and customer-facing configurator.

#### Type 1: Printed Materials (Square-Foot Pricing)
**Products:** 22 (Vinyl Banner, Mesh Banner, Adhesive Decal, Corrugated Plastic, Foamcore, PVC, Aluminum Composite, Acrylic, Canvas, etc.)
**Category trigger:** Products in `flexible-materials` or `rigid-materials` with `_pricing_type = yes`
**Pricing:** Per square foot, role-based. Customer enters width (ft+in) and height (ft+in).
```
sqft = ceil(ft + in/12) width × ceil(ft + in/12) height
price = sqft × rate_per_sqft[role][side]
+ finishing_option_fee (if applicable)
× 1.2 (if die-cut selected)
+ upgrade_per_sqft × sqft (if reflective upgrade)
+ $35 (if artwork setup)
+ $10 (if artwork proof)
× quantity
```
**Per-product customizations:**
- Vinyl Banner (8120): finishing options dropdown (grommets, hemming +$20, pole pockets +$10-20)
- Corrugated Plastic (8131): flute direction selector (vertical/horizontal with images)
- Aluminum Composite (8146): aluminum color image picker (only for .040-color thickness)
- Ultraboard (8134): special variation handling on thickness change
- Adhesive Decal (8122): "Unlaminated"/"Laminated" labels instead of Single/Double
- Reflective Decal (56155): "Laminated" only (no double-sided)
- Low Tack (8123): material finish dropdown (Smooth/Textured)
- Cut contour option on 13 products: Simple Cut (free), Cloud Cut (free), Die Cut (+20%)

#### Type 2: Rigid Materials (Table-Lookup Pricing)
**Products:** 13 (hardcoded IDs 8327–8338, 56155)
**Pricing:** Lookup from `_price_table_materials` option by material × size × role. Single/double sided pricing.
**Inputs:** Material dropdown, size dropdown, side radio, extra checkbox, quantity

#### Type 3: Options Group (Hardware/Stand Products)
**Products:** 6 (hardcoded IDs 8316–8320, 56153 — retractable banners, A-frames, backdrop displays, tension banners, pole banners)
**Pricing:** Lookup from `_price_table_options` by stand type × option. Three pricing columns: Hardware Only, Print and Hardware, Print Only.
**Each product has its own template** (`product_8316.php`, etc.) with different size option structures.

#### Type 4: Size-Only Products
**Products:** 4 (hardcoded IDs 10346–10349)
**Pricing:** Simple size → price lookup from `_price_table_sizeonly`. Minimum quantities per size.
**Note:** Product 10346 is hardcoded as double-sided.

#### Type 5: Lawn Sign Kit
**Product:** 1 (hardcoded ID 8341)
**Pricing:** Quantity-tiered pricing with orientation (Landscape/Portrait) × size (24"×18" or 18"×12") × side × hardware. Price per unit decreases at higher quantities.
**Special:** Server-side quantity recalculation when cart quantity changes (`cal_unit_price` action).

#### Type 6: Real Estate Sign Kit
**Product:** 1 (hardcoded ID 10350)
**Pricing:** Multi-step wizard: Style (Banjo/H-Frame) → Frame Size → Material → Side. Complex nested pricing from `_price_table_realestate`.

#### Type 7: Poster Stand Kit
**Product:** 1 (hardcoded ID 10469)
**Pricing:** Multi-step: Stand Options → Color (Silver/Black) → Material → Side. From `_price_table_poster_stand_kit`.

#### Type 8: Large Commercial Real Estate Signs (LCRES)
**Product:** 1 (hardcoded ID 56157)
**Pricing:** Size × Side × Orientation from `_price_table_lcres`. Optional "refurbishing" indicator and location textarea.

#### Type 9: Custom Project
**Product:** 1 (hardcoded ID 56156)
**Pricing:** Manual price entry. Admin/internal use for custom work.

#### Type 10: Core Logo Package
**Product:** 1 (hardcoded ID 10351)
**Pricing:** Fixed base $400 + add-ons (extra designs +$50/+$125, revisions +$200, logo character +$200, trademark +$300, brand guide +$250, consultation +$99). Not role-based.

### 1.2 Shared Product Components

**Artwork Options (4 tabs on every custom product):**
1. Upload Artwork Now — file upload via gallery modal
2. Upload Artwork Later — no upload needed
3. Artwork Setup (+$35) — Deadline Signs creates artwork
4. I Need a Designer — sub-options: email me / schedule call (date/time picker + phone)

**Artwork Review Method:**
- Artwork Proof Not Required (free, default)
- Artwork Proof Required (+$10)

**Additional fields:** Notes textarea, deadline date picker, "+Add Sign" for multiple designs with per-design quantities.

**File Upload System:**
- Logged-in: WordPress media library upload
- Guest: session-token-based tracking (`_token_auto_delete`)
- Supports images, PDFs, EPS, AI files
- Gallery modal with select/delete controls per file

### 1.3 Role-Based Pricing

**Roles (in pricing order, cheapest first):**
1. `trade` — reseller tier (cheapest)
2. `wholesale` — standard B2B tier
3. `shop_manager` — staff tier
4. `administrator` — maps to `wholesale` on main store, `trade` on reseller portal
5. `project_client` — most expensive tier

**Price storage (sq/ft products):** Post meta per product/variation:
- `_value_single_{role}` / `_value_double_{role}`
- `_value_single_{role}_cutonly` / `_value_double_{role}_cutonly`
- `_value_single_{role}_upgrades` / `_value_double_{role}_upgrades`

**Price storage (other types):** wp_options tables:
- `_price_table_options`, `_price_table_materials`, `_price_table_sizeonly`
- `_price_table_lawn_sign_kit`, `_price_table_realestate`, `_price_table_poster_stand_kit`
- `_price_table_lcres`, `_price_table_quantity`, `_price_table_2`

### 1.4 Catalog Mode

Prices are **hidden from logged-out visitors**. Non-authenticated users see "Register to See Prices". The reseller portal goes further — the **entire site requires login** (WP Force Login plugin).

### 1.5 Product Categories

Essentially flat hierarchy (only 1 child category). 24 categories on main store. Products can belong to multiple categories simultaneously.

Key navigational categories: Build your Own Sign (21 products), Customize your Sign (33), Ready to Buy Sign (12), Premade Signs (10), Flexible Signs (14), Rigid Signs (27), Sign Hardware (24), Sign Kits (14).

### 1.6 Product Attributes

8 registered taxonomy attributes: kind, size, single-sided, double-sided, thickness (28+ terms), color-option, color (38 terms), packagequantity.

Many products also use custom (non-taxonomy) attributes for display purposes.

---

## 2. User System

### 2.1 Roles & Capabilities

| Role | Main Store Count | Reseller Count | Notes |
|------|-----------------|----------------|-------|
| wholesale | 3,244 | 23 | Default B2B customer |
| trade | — | 171 | Reseller tier (cheapest pricing) |
| customer | 12 | 1 | Rarely used |
| administrator | 6 | 4 | Staff (kris, chantel, carlie, megan) |
| shop_manager | 1 | — | Staff |
| project_client | — | — | Pricing tier exists but no users |

**Capability flags (secondary roles):**
- `paybycheck` — 355 users on main, 76 on reseller. Grants access to "Pay by Check" payment method.
- `tax_exempt` — 85 users on main, 163 on reseller. Exempts from NC 7% sales tax.

### 2.2 Registration Flows

**Main store:** Open registration via checkout or My Account. Ajax modal login/registration (`ajax-login-and-registration-modal-popup-pro`). Custom fields: company, phone, reseller flag, Mailchimp opt-in.

**Reseller portal:** Invite-only. Prospective resellers fill out a Quform registration requiring Federal Tax ID and E-595 Reseller Certificate. Admin manually approves and creates accounts. Users receive purple-branded welcome email.

### 2.3 Custom User Meta

| Meta Key | Purpose |
|----------|---------|
| `account_funds` | Store credit balance |
| `show_pay_by_check` | Pay-by-check permission |
| `_rush_shipping` | Rush shipping permission |
| `_invoice_separately` | Invoice separately flag |
| `_request_quote` | Quote request permission |
| `_change_delivery_shipping` | Delivery override |
| `_shipping_method` | Preferred shipping method |
| `_have_shipping_account` | Has own FedEx/UPS account |
| `reseller_register` | Reseller registration flag |

---

## 3. Cart & Checkout

### 3.1 Add-to-Cart Flow

Custom AJAX handler (`option_add_to_cart`) replaces standard WooCommerce add-to-cart. Sends:
- `product_id`, `variation_id`, `product_qty`
- `data_field[]` — array of {name, value, type} objects (dimensions, material, side, artwork choices)
- `price` — **CLIENT-CALCULATED unit price** (no server validation — security vulnerability)
- `ex_option[]` — extra option prices (finishing, grommets, etc.)
- `price_array`, `price_by_qty` — tiered pricing data (lawn sign kit only)

**Cart item meta stored:** `data_field`, `_final_price`, `_final_qty`, `_ex_option`, `minimum`, `artwork_setup`, `proof_sent`, `finishing_option`

**Price override mechanism:** WooCommerce's calculated prices are replaced at checkout:
- `woocommerce_before_calculate_totals`: sets price to `_final_qty × _final_price + extras`, forces WC qty to 1
- `woocommerce_after_calculate_totals`: restores per-unit price and actual qty for display

### 3.2 Cart Page

Custom template with 8 columns: Remove | Thumbnail | Product | Price | Quantity | Subtotal | Add Ons | Total.

Features: coupon modal popup, Account Funds apply modal, artwork edit modal, notes textarea (saved to cookie).

### 3.3 Five-Step Multistep Checkout

**Implementation:** Custom theme code (`form-checkout.php` + `script-checkout-new.js`), not a plugin. ~2,000 lines PHP + ~800 lines JS + ~1,454 lines CSS.

**Step 0 — Cart** (link back)

**Step 1 — Shipping:**
- "Pick up In-Store" (51-C Carpenter Ct NW, Concord, NC 28027)
- "Ship To" → address form → delivery method:
  - Ground Shipping — may trigger shipping quote modal (if no shipping class)
  - Expedited Shipping — triggers rush modal (zip code + due date)
  - Both offer "Request a Quote" or "Invoice Separately"
- Blind Shipping checkbox
- Admin-only: "Quote Shipping Price" modal to set custom shipping amount

**Step 2 — Billing + Payment:**
- Billing fields: name, company, address, city, state, zip, country
- Contact: email, phone
- Payment method selection shown alongside billing

**Step 3 — Review:**
- Full order review table (8 columns)
- Billing, contact, shipping summaries
- Price breakdown: subtotal, credits, coupons, shipping, processing fee (3.1%), tax, total
- Place Order button
- **Admin-only:** per-line-item price override

**Step 4 — Complete** (order received / thank you)

### 3.4 Shop-as-Customer (POS)

Plugin: `woocommerce-shop-as-customer` (cxThemes v2.16.1, locally patched).

Admin bar dropdown with customer search (Select2 AJAX). Three checkout modes:
1. **Checkout — Pay Now** — normal payment flow
2. **Checkout — Pay Later** — creates Pending Payment order, sends invoice email with payment link
3. **Checkout — FREE** — for zero-total orders, creates as Processing

"Send Email to Customer" checkbox controls whether transactional emails fire during SAC checkout.

### 3.5 Cart-to-Quote (C2Q)

Plugin: `woo-cart-to-quote` (v1.0.12). 5,109 quotes in system.

**Flow:**
1. Customer adds products to quote list (button text: "Add to Quote")
2. Customer submits quote
3. Admin reviews, can edit prices per line item (`_c2q_edited`, `_org_price` flags)
4. Admin sends quote → customer receives email with "Checkout Now" link
5. Customer clicks link → quote items load to cart with admin-set prices
6. Customer completes checkout normally

**Quote acceptance:** via `?accept_quote={id}` URL. Validates user ownership, loads items to cart with quote pricing, redirects to cart.

---

## 4. Payments

### 4.1 Active Payment Methods

| Method | Gateway ID | Notes |
|--------|-----------|-------|
| Stripe (custom) | `ds_stripe` | Payment Intents API, no SDK, 3D Secure support |
| PayPal Standard | `paypal` | email: web@2minds.biz |
| PayPal Express | `ppec_paypal` | Same account |
| Pay by Check | `cheque` | Role-gated (`paybycheck` capability) |
| Bill Me (COD) | `cod` | Invoice-based, not actual COD |
| Account Funds | `accountfunds` | Store credit, partial payment, min topup $10 |

### 4.2 Stripe Gateway Details

Custom plugin (`deadline-stripe-gateway` v1.0.5). No Stripe PHP SDK — uses `wp_remote_post()` with Basic auth.

**Flow:**
1. Stripe.js loaded in footer, Card Element mounted via polling
2. `stripe.createPaymentMethod()` on card complete → stores `pmId`
3. jQuery ajaxPrefilter injects `ds_stripe_payment_method` into WC checkout AJAX
4. Server creates + confirms PaymentIntent in single API call
5. 3D Secure handled via redirect or client-side `confirmCardPayment`
6. Stores `_ds_stripe_intent_id` as order meta
7. Supports admin refunds via `create_refund()`

Stripe API version: `2022-11-15`. Separate test/live keys configurable.

### 4.3 Convenience Fee

Plugin: `deadline-signs-convenience-fee` (v1.3.0). Single-file plugin.

- 3.1% fee on `cart_subtotal + shipping + taxes`
- Applied only on checkout page (not cart)
- Exempt: `accountfunds`, `bacs`, `cheque`, `cod`
- Recalculates dynamically when payment method changes

### 4.4 Account Funds

WooCommerce Account Funds plugin. Store credit system.
- Partial payment: enabled
- Minimum top-up: $10
- 5% discount for fund usage: configured but currently disabled
- 535 users have account fund balances

---

## 5. Order Management

### 5.1 Order Statuses

| Status | Label | Trigger |
|--------|-------|---------|
| `pending` | Pending Payment | Order created, awaiting payment |
| `processing` | Order Submitted | Payment received |
| `awaiting` | Prepress | Manual (admin sets) |
| `production` | In Production | Manual |
| `finishing` | Finishing | Manual |
| `installation` | Installation | Manual |
| `shipping` | Shipped / Ready for Pick-up | Manual |
| `on-hold` | On Hold | Manual |
| `completed` | Completed | Manual |
| `cancelled` | Cancelled | Manual or system |
| `refunded` | Refunded | Via Stripe refund |
| `failed` | Failed | Payment failure |

### 5.2 Production Dashboard

Plugin: `dashboard-deadlinesigns` (v1.0.2). Custom DB table `dashboard_datatable`.

**Tabs:** Processing, Prepress, In Production, Completed, Settings

**24-column table per order item:** Order#, Customer, Reference Title, Due Date, Product, Material, Qty, P&C (print&cut), CC (cut contour), DS (double side), Size (WxH), Options 1/2/3, Front artwork, Back artwork, Proof, Status dropdown, Notes, Deadline, action buttons.

**Features:**
- Material mapping (admin maps product IDs to material names)
- Cross-site aggregation (shows orders from all multisite blogs)
- Due date logic: item meta `_due_date` → `Deadline` meta → order date + 2 days
- Red highlight for overdue items
- DataTables server-side pagination with search
- Batch status changes

### 5.3 SMS Notifications

Provider: **Flowroute** (`https://api.flowroute.com/v2.1/messages`)
From: `19802221301`
Trigger: `woocommerce_order_status_changed` (priority 21)
Dedup: WordPress transients (60-second TTL)

| Status | Message |
|--------|---------|
| processing | "Thank you for your order #X..." |
| awaiting | "...is IN Pre-Production." |
| production | "...is IN PRODUCTION." |
| shipping | "...is READY FOR PICKUP or being PREPARED FOR SHIPMENT." |
| on-hold | "...has been put ON HOLD." |
| completed | "...has been COMPLETED." |
| cancelled | "...has been CANCELLED." |
| refunded | "...has been REFUNDED." |
| failed | "...payment has failed. Please visit deadlinesigns.com..." |

Phone validation: US format only (11 digits starting with 1).

### 5.4 Email Notifications

15 custom email templates including:
- Order Submitted (processing)
- Status change emails (prepress, production, shipping)
- Order completed
- Customer invoice (Pay Later)
- Admin note added to order (custom `WC_Note_Order_Email` class)
- Welcome / account creation
- Password reset
- Account funds increase
- Reseller-specific welcome (purple branding vs main store red)

### 5.5 PDF Invoices

Plugin: `woocommerce-pdf-invoices-packing-slips` (v2.3.5). 3,615 invoices generated. Custom templates in theme.

---

## 6. Shipping

### 6.1 Shipping Zones

| Zone | Region | Method |
|------|--------|--------|
| Far West USA | AK, CA, HI, OR, WA | Table Rate + Free Shipping (coupon required) |
| US Not Far West | 45 other states + DC | Table Rate + Free Shipping (coupon required) |
| Rest of World | — | Free Shipping (quoted) |

### 6.2 Table Rates (Main Store)

| Zone | Shipping Class | Condition | Cost |
|------|---------------|-----------|------|
| Far West | 12/18/24 Yard Letters | $1–100 | $35 |
| Far West | 12/18/24 Yard Letters | $101–200 | $45 |
| Far West | Handheld Signs | 1–20 items | $45 |
| Far West | Handheld Signs | 21–50 items | $70 |
| US Not Far West | 12/18/24 Yard Letters | $1–200 | $15 |
| US Not Far West | Handheld Signs | 1–20 items | $20 |
| US Not Far West | Handheld Signs | 21–50 items | $45 |
| US Not Far West | Retractable Banner Stand | per item | $20 |

### 6.3 Shipping Classes (12)

12 Yard Letters, 18 Yard Letters, 24 Yard Letters, Custom, Free Shipping, Handheld Signs, Included, Local Pickup Only, Practice, Retractable Banner Stand, Stackable Boxes, Yard Sign Kits, Yard Sign No Legs.

### 6.4 Custom Shipping Logic

- Custom shipping method class `Deadlinesigns_Shipping_Method` in child theme
- Ground shipping triggers quote modal if products have no shipping class
- Expedited shipping triggers rush modal (zip + due date)
- Admin can set custom shipping price via modal (stored in session `quote_shipping_price`)
- Customer FedEx/UPS account fields: `_fedex_account`, `_ups_account`
- Blind shipping option

---

## 7. Reseller Portal

### 7.1 Overview

Separate WooCommerce store on WordPress multisite blog 4 (`reseller.deadlinesigns.com`). Shares theme, users table, and several plugins with main store. Has its own products (synced from main), orders, and pricing.

### 7.2 Key Differences from Main Store

| Aspect | Main Store | Reseller Portal |
|--------|-----------|----------------|
| Access | Catalog mode (prices hidden when logged out) | Entire site requires login |
| Default pricing | Wholesale | Trade (cheapest) |
| Registration | Open (checkout or My Account) | Invite-only (tax ID + reseller cert required) |
| Tax | 7% NC tax | Most users tax-exempt (163/199) |
| Pay by Check | Some users | 76/199 users |
| Users | ~3,782 | 199 |
| Orders (all time) | 5,776 | 2,539 |
| Active customers (2025+) | — | 43 |
| Branding | Red accent | Purple accent |
| Product IDs | Standard | Some differ (e.g., Shipping product: 28889 vs 22047) |

### 7.3 Product Sync

Plugin `woo-multisite-product-category-sync` syncs product posts and categories from blog 1 → blog 4. Products exist in both blogs' tables but with **independent pricing** in each blog's postmeta.

### 7.4 Pricing Comparison (Example: Corrugated Plastic per sq/ft)

| Side | Trade | Wholesale | Project Client |
|------|-------|-----------|----------------|
| Single | $2.00 | $5.00 | $6.50 |
| Double | $4.00 | $7.00 | $8.50 |
| Cut only | $1.00 | $3.00 | — |

### 7.5 Reseller-Specific Pages

- Trade Account Registration (Quform with tax ID + E-595 cert)
- Reseller Artwork Specifications
- Reseller Pricing
- Downloadable Templates
- Material Videos
- Quote Terms / Drafted Quote / Quotes
- Reward Points
- Client Portal
- FAQ, Holiday Hours

---

## 8. Integrations

| System | Purpose | Volume |
|--------|---------|--------|
| **Xero** | Accounting | 5,537 orders synced |
| **Mailchimp** | Email marketing | Auto-subscribe on registration |
| **Zapier** | Automation | Order/status triggers |
| **Flowroute** | SMS notifications | All order status changes |
| **Stripe** | Payment processing | Primary gateway |
| **PayPal** | Payment processing | Secondary gateway |
| **Searchanise** | Product search | Active on storefront |
| **PDF Invoices** | Document generation | 3,615 invoices |
| **Google Analytics** | Tracking | GA Pro plugin |

---

## 9. Security Issues to Fix in Rebuild

1. **Client-side price trust** — Unit prices calculated in JavaScript and sent to server without validation. Any user can modify prices via browser dev tools. **Must implement server-side price verification.**

2. **Exposed API credentials** — Flowroute SMS credentials (access key, secret key) and Stripe keys are in source code. **Must use environment variables.**

3. **No CSRF on pricing table save** — The AJAX `wtp_save_table_price` handler has no nonce verification. **Must add CSRF protection.**

4. **Session-based guest tracking** — Guest uploads use PHP sessions with custom tokens. **Must use proper authentication or signed URLs.**

5. **Unrestricted file uploads** — `media_handle_upload` called for both logged-in and guest users without file type validation beyond WordPress defaults. **Must validate file types server-side.**

---

## 10. Data Migration Plan

### What moves to the new system:

| Data | Count | Method |
|------|-------|--------|
| Products (published) | 89 + 525 variations | SQL export + transform |
| Product categories | 24 | Manual or SQL |
| Product attributes | 8 taxonomy + many custom | Manual mapping |
| Product pricing tables | 9 wp_options entries | SQL export + transform to new schema |
| Per-product pricing meta | ~22 products × ~10 roles/sides | SQL export |
| Users | 4,024 | SQL export (passwords won't transfer — force reset) |
| User roles + capabilities | per-user | SQL export + role mapping |
| Orders (historical) | 8,315 | SQL export for reporting |
| Order items + meta | 25,399+ items | SQL export |
| Quotes | 5,109 | SQL export |
| Account fund balances | 535 users | SQL export |
| Shipping zones + rates | 2 zones, 12 classes, 10 rates | Manual config |
| Tax rates | 1 (NC 7%) | Manual config |
| Xero invoice IDs | 5,537 | Preserve in order migration |

### What doesn't move:
- WordPress post/page content (rebuild pages in Next.js)
- WPBakery page layouts (rebuild with Tailwind)
- Email log (943 MB — discard)
- User activity logs (122 MB — discard)
- Abandoned cart logs (116 MB — discard)
- Image optimizer data (51 MB — discard)
- SEO suggestions (53 MB — discard)
- Amazon/eBay/Etsy marketplace tables (not actively used)
- Dokan, Jetpack CRM, Gravity Forms tables (all empty or unused)

---

## 11. Recommended Architecture for Rebuild

### Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend | Next.js 14+ (App Router) | SSR for SEO, React for interactive configurators |
| Styling | Tailwind CSS | Fast iteration, no theme dependency |
| Database | PostgreSQL | Already on server, relational fits pricing/orders |
| ORM | Prisma | Type-safe queries, migrations |
| Auth | NextAuth.js | Role-based access, session management |
| Payments | Stripe Node SDK | First-class support, replace custom gateway |
| File Storage | S3-compatible (existing Wasabi) | Already using via ILAB Media Tools |
| SMS | Flowroute API (or Twilio) | Simple HTTP API |
| Email | Resend or SendGrid | Transactional emails |
| Search | Built-in PostgreSQL full-text or Meilisearch | Replace Searchanise |
| PDF | @react-pdf/renderer or Puppeteer | Replace WC PDF Invoices |

### Key Data Models

```
User (id, email, name, company, phone, role, tax_exempt, pay_by_check, account_balance)
Product (id, name, slug, type, categories[], status, max_width, max_height, pricing_type)
ProductVariation (id, product_id, attributes{}, sku)
PricingRule (id, product_id, variation_id?, role, side, cut_type, price_per_unit, upgrade_price)
PricingTable (id, product_type, role, data{}) — for table-lookup products
Order (id, user_id, status, billing{}, shipping{}, subtotal, tax, shipping_cost, convenience_fee, total, stripe_intent_id)
OrderItem (id, order_id, product_id, variation_id, quantity, unit_price, config{}, artwork{}, extras[])
Quote (id, user_id, status, items[], admin_notes, sent_at, accepted_at)
QuoteItem (id, quote_id, product_id, config{}, original_price, admin_price)
ShippingZone (id, name, regions[], method, rates{})
```

### Module Breakdown

1. **Auth & Users** — registration, login, role management, reseller application workflow
2. **Product Catalog** — browsing, search, category filtering, catalog mode (prices hidden when logged out)
3. **Product Configurator** — the interactive builder for each product type (width/height, material, side, artwork, etc.)
4. **Pricing Engine** — server-side price calculation with role-based rates. **No client-side price trust.**
5. **Cart** — custom cart with line item configuration display, extras, artwork
6. **Checkout** — multi-step (shipping → billing/payment → review → complete)
7. **Payments** — Stripe (with 3.1% fee), PayPal, Pay by Check, Bill Me, Account Funds
8. **Orders** — custom status workflow, production dashboard, status change triggers
9. **Notifications** — SMS (Flowroute), transactional email, order notes
10. **Quotes** — create, admin edit, send, accept, convert to order
11. **Shop-as-Customer** — admin impersonation for POS use
12. **Reseller Portal** — gated access, trade pricing, separate branding
13. **Admin Dashboard** — product management, pricing tables, order management, production tracking
14. **Integrations** — Xero accounting, Zapier webhooks, Mailchimp, PDF generation
15. **File Management** — artwork upload, gallery, per-user file library

---

## 12. What Would Impress Customers

The current site works but feels dated. Here's what a modern rebuild can do that WordPress can't:

1. **Live price calculator** — as customers adjust width, height, material, and options, the price updates instantly with smooth animations. No page reloads on thickness/variation change.

2. **Visual product preview** — show a scaled representation of the sign at the entered dimensions. "Your 3ft × 5ft vinyl banner will look like this."

3. **Artwork upload with live preview** — drag-and-drop artwork onto the sign preview. Show how it will look on the actual product.

4. **Order tracking dashboard** — real-time status with the progress bar, but modern. Push notifications when status changes. Photo of finished product before shipping.

5. **Instant quote builder** — instead of the clunky C2Q plugin, let customers build a quote visually, share it via link, and convert to order with one click.

6. **Reorder in one click** — past orders show "Reorder" button that pre-fills the configurator with the same specs and artwork.

7. **Mobile-first checkout** — the current 5-step checkout doesn't work well on phones. A single-page responsive checkout with collapsible sections.

8. **Saved projects** — customers can save partially-configured products and come back to them.

9. **Bulk order builder** — add multiple signs with different sizes/materials in one flow, see the total update live.

10. **Reseller portal that feels like a separate brand** — distinct URL, distinct color scheme, but seamless shared backend.
