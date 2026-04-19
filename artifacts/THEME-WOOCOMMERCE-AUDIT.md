# Deadline Signs - Theme & WooCommerce Comprehensive Audit

**Audit Date:** 2026-04-18
**Server:** OVH (ssh ovh)
**Site Root:** `/var/www/vhosts/deadlinesigns.com/httpdocs`
**Theme Path:** `wp-content/themes/print-left-navigation/`
**Parent Theme:** `printshop` by Netbase Team

---

## 1. THEME STRUCTURE

### 1.1 Theme Identity

- **Theme Name:** Print Left Navigation
- **Parent Theme:** printshop (by Netbase Team)
- **Template:** printshop
- **Version:** 1.0.0

### 1.2 Complete File Inventory

#### PHP Files (17 files, ~4,180 lines total)

| File | Lines | Purpose |
|------|-------|---------|
| `functions.php` | 1,166 | Main child theme functions, styles/scripts enqueue, metaboxes, product link fields, admin/login redirects, performance optimization |
| `account_functions.php` | 1,668 | Registration fields, custom order statuses, My Account customization, WooCommerce endpoints, order management, Mailchimp integration |
| `functions-checkout-multistep.php` | 235 | Custom shipping method logic, shipping rate overrides, order meta for shipping labels |
| `functions-shortcode.php` | 593 | 7 custom shortcodes for homepage/product display |
| `functions-my-account.php` | 245 | Account menu manager (admin panel), WMPCS multisite sync |
| `function-shipping-method.php` | 91 | Custom WC_Shipping_Method class "Deadlinesigns Shipping" |
| `ajax.php` | 182 | 8 AJAX handlers for checkout shipping/fund operations |
| `header-creativeleft.php` | ~200 | **PRIMARY HEADER** - top menu bar, logo, search, "Get a Quote", navigation, cart icon, user greeting |
| `header-creativeleft-second.php` | ~250 | **ALTERNATE HEADER** - left-sidebar style layout (commented out legacy code), menu with search, wallet display |
| `footer.php` | ~150 | 5-column footer with widget areas, copyright, Two Minds Group branding, Google Maps embed, mobile bottom navbar |
| `404.php` | ~60 | Custom 404 page with branded image, also serves static HTML files from `/html/` directory |
| `page-artwork-specifications.php` | ~60 | Standard page template with `content_only` query parameter support |
| `page-no-footer.php` | ~50 | Page template that hides the main footer section on desktop |
| `page-sidebar-left.php` | ~40 | Page template with shop sidebar on the left |
| `template_page_with_wc_notices.php` | ~70 | Page template with WC notices, breadcrumbs, and sign-in popup for non-logged-in users |
| `woocommerce.php` | ~80 | WooCommerce archive/shop/product template wrapper with breadcrumbs |
| `vc_templates/vc_images_carousel.php` | ~120 | WPBakery image carousel override |

#### CSS Files (8 files, ~6,106 lines total)

| File | Lines | Purpose |
|------|-------|---------|
| `style.css` | 1,760 | Main child theme styles - cart icon, header, products, responsive |
| `style-header.css` | 2,368 | Complete header redesign - top menu bar, megamenu, mobile navigation |
| `style-checkout.css` | 811 | Checkout form styling, billing fields, multistep layout |
| `style-checkouts.css` | 643 | Additional checkout styles (separate file for versioning) |
| `style-dashboard.css` | 311 | My Account dashboard styling |
| `style-fonts.css` | 114 | @font-face declarations for Montserrat, Muli, Sansation |
| `style-cart.css` | 85 | Cart page specific styles |
| `style-admin.css` | 14 | Admin panel styles |
| `css/megamenu-unused.css` | - | Unused MegaMenu styles (kept as reference) |
| `css/jquery.fancybox.min.css` | - | Fancybox lightbox styles |

#### JavaScript Files (6 files)

| File | Purpose |
|------|---------|
| `js/script-checkout-new.js` | **CRITICAL** - 800+ lines, drives the entire 5-step checkout flow, step navigation, validation, shipping modals, Stripe card data capture |
| `js/customize.js` | Header/menu scroll behavior, mobile hamburger menu, product filters, product widget toggles |
| `js/script-product.js` | 3 lines - disables submit button on variant change for special products |
| `js/stripe.js` | **DISABLED** (March 2026) - Legacy Stripe Sources API JS, replaced by plugin's own Payment Intents JS |
| `js/jquery.sticky-kit.min.js` | Sticky sidebar positioning library |
| `js/jquery.fancybox.min.js` | Fancybox lightbox for login/register modals |

#### Font Files (14 files)

- **Montserrat** (SemiBold, Black) - used for H1, H2 headings
- **Muli** (Regular, Italic, ExtraBold, ExtraBoldItalic) - primary body font, H3-H5
- **BebasNeue** (multiple formats) - accent/display font
- **Oswald** Regular - secondary heading font
- **Raleway** Regular - commented out
- **Sansation** Bold - accent font

#### Image Assets (20+ files)

- Company logos (main site + reseller variants)
- Payment icons (Visa, Mastercard, Amex, Discover, PayPal, Stripe)
- UI icons (cart, upload, pencil, trophy, person, question mark)
- 404 error page image

---

## 2. WOOCOMMERCE TEMPLATE OVERRIDES

### 2.1 Override Count by Location

| Location | Count |
|----------|-------|
| Child theme (`print-left-navigation/woocommerce/`) | **52 files** |
| Parent theme (`printshop/woocommerce/`) | **62 files** |
| Plugin (`wc-special-product-pricing/woocommerce/`) | **8 files** |
| **Total unique WooCommerce overrides** | **~65 unique templates** (child overrides parent) |

### 2.2 Child Theme WooCommerce Overrides (Active - These Take Priority)

#### Cart Templates (7 files)

| File | Key Changes |
|------|-------------|
| `cart/cart.php` | **Heavily customized** - adds 5-step checkout progress bar at top, 8-column table (Remove, Thumbnail, Product, Price, Qty, Subtotal, Add Ons, Total), calculates `_ext_price` add-ons separately, customer notes textarea with cookie persistence, coupon/fund modals, artwork edit modal, Account Funds integration |
| `cart/cart-totals.php` | Custom order summary - shows Coupon (with modal Apply link), Account Credits row, deducts shipping from total display, Processing Fee label rename |
| `cart/cart-empty.php` | Override (inherited from parent) |
| `cart/cart-item-data.php` | Override (inherited from parent) |
| `cart/cart-shipping.php` | Override for custom shipping display |
| `cart/mini-cart.php` | Custom mini-cart layout with product thumbnails |
| `cart/proceed-to-checkout-button.php` | Custom checkout button |
| `cart/checkout-cart-left.php` | **Custom file** - left sidebar cart summary used in checkout |

#### Checkout Templates (9 files)

| File | Key Changes |
|------|-------------|
| `checkout/form-checkout.php` | **MASSIVE OVERRIDE** - 5-step tabbed checkout (Cart > Shipping > Billing > Review > Complete), 3-column layout (steps panel, sidebar order summary, hidden payment panel), coupon/fund/shipping modals, admin shipping price quotation modal |
| `checkout/form-shipping.php` | **Custom shipping UI** - radio buttons for "Pick up In-Store" vs "Ship To", shipping address fields, blind shipping checkbox, delivery method (Ground vs Expedited), disabled states based on cart contents |
| `checkout/form-billing.php` | Split into Billing Information + Order Contact Information sections, specific field ordering |
| `checkout/form-pay.php` | Payment form override |
| `checkout/payment.php` | Simplified payment method list, commented-out Account Funds inline integration |
| `checkout/review-order.php` | Sidebar order summary with product thumbnails, admin price override inputs, Account Credits, shipping quote link for admins, Processing Fee rename |
| `checkout/review-order-detail.php` | **Custom file** - full order review table (8 columns) with billing/shipping/payment info display panels for Step 3 |
| `checkout/terms.php` | Terms override |
| `checkout/thankyou.php` | Custom thank you with personalized message, Print PDF button (WPO WCPDF), Continue Shopping button |

#### My Account Templates (13 files)

| File | Key Changes |
|------|-------------|
| `myaccount/dashboard.php` | **Rich dashboard** - welcome message, Account Credits balance, 4 tile cards (Orders, Quotes, Reward Points, Image Library), profile card with avatar, recent orders table, recent quotes via shortcode |
| `myaccount/navigation.php` | Custom navigation with user first name in data attribute |
| `myaccount/form-login.php` | Branded login/register with tabbed interface, social login integration (NextendSocialLogin), "Keep me signed in" checkbox |
| `myaccount/form-edit-account.php` | Account edit form override |
| `myaccount/form-add-payment-method.php` | Payment method form override |
| `myaccount/form-lost-password.php` | Lost password form override |
| `myaccount/form-reset-password.php` | Reset password form override |
| `myaccount/lost-password-confirmation.php` | Confirmation page override |
| `myaccount/my-address.php` | Address management override |
| `myaccount/my-orders.php` | Orders list with custom columns (Reference Title, Receipt, Notes, Reorder) |
| `myaccount/orders.php` | Orders table override |
| `myaccount/library-files.php` | **Custom endpoint** - Image Library file management |
| `myaccount/enter-email.php` | **Custom endpoint** - forces email entry for social-login users |

#### Single Product Templates (9 files)

| File | Key Changes |
|------|-------------|
| `content-single-product.php` | Minimal changes - allows product summary for non-logged-in users (was previously blocked) |
| `single-product/add-to-cart/variable.php` | Adds out-of-stock labels to variation options, `data-product_link` attribute for variant redirects, uses `custom_wc_dropdown_variation_attribute_options()` |
| `single-product/add-to-cart/variation-add-to-cart-button.php` | Simplified - removes duplicate `woocommerce_before/after_add_to_cart_button` hooks |
| `single-product/add-to-cart/grouped.php` | Grouped product override |
| `single-product/price.php` | Price display override |
| `single-product/product-image.php` | Product image override |
| `single-product/related.php` | Related products override |
| `single-product/short-description.php` | Short description override |
| `single-product/tabs/tabs.php` | Uses `<h3>` instead of standard tab links |
| `single-product/tabs/additional-information.php` | Additional info tab override |
| `single-product/tabs/description.php` | Description tab override |

#### Loop/Archive Templates (4 files)

| File | Key Changes |
|------|-------------|
| `content-product.php` | Custom product card layout with thumbnail wrapper, split info sections for grid vs list view |
| `content-product_cat.php` | Category card override |
| `content-newproduct.php` | New product display template |
| `loop/add-to-cart.php` | Add to cart button override |
| `loop/pagination.php` | Pagination override |
| `loop/rating.php` | Rating display override |
| `loop/title.php` | Product title override |

#### Email Templates (15 files)

| File | Key Changes |
|------|-------------|
| `emails/customer-completed-order.php` | Custom completion message: "Your order is now completed. If you wish to reorder..." |
| `emails/customer-processing-order.php` | Processing order notification |
| `emails/customer-invoice.php` | Invoice email override |
| `emails/customer-reset-password.php` | Password reset email |
| `emails/customer-new-status-order.php` | **Custom email** - sent when order moves to Prepress, In Production, or Shipped |
| `emails/customer-readyforpickup-order.php` | **Custom email** - ready for pickup notification |
| `emails/customer-account-funds-increase.php` | Account funds increase notification |
| `emails/admin-note-order.php` | **Custom email** - notifies customer when admin adds order note |
| `emails/email-order-details.php` | Order details in emails override |
| `emails/email-order-items.php` | Order items in emails override |
| `emails/email-addresses.php` | Email address display override |
| `emails/email-styles.php` | Email CSS styles override |
| `emails/email-header-login-modal.php` | **Custom** - login modal email header |
| `emails/email-header-login-reseller-modal.php` | **Custom** - reseller login modal header |
| `emails/email-header-register-modal.php` | **Custom** - register modal email header |
| `emails/email-footer-login-modal.php` | **Custom** - login modal email footer |
| `emails/email-footer-login-reseller-modal.php` | **Custom** - reseller login modal footer |
| `emails/email-footer-register-modal.php` | **Custom** - register modal email footer |

#### Order Templates (3 files)

| File | Key Changes |
|------|-------------|
| `order/order-details.php` | Dual layout - 8-column table for thank-you page (with Add Ons column), 3-column for account page |
| `order/order-details-item.php` | Order item display override |
| `order/order-details-customer.php` | Customer details override |

#### PDF Templates (3 template sets)

- `pdf/Simple-Custom/` - Custom invoice, packing slip, styles, functions
- `pdf/Simple-Custom-bk/` - Backup of custom PDF templates
- `pdf/Simple-New/` - Newer PDF template set

#### Other Templates

| File | Purpose |
|------|---------|
| `product-searchform.php` | Custom product search form |
| `product-sticky.php` | Sticky product bar |
| `wishlist-view.php` | Wishlist view override |
| `wpnetbase-product-reviews.php` | Product reviews override |
| `wpnetbase-single-product-reviews.php` | Single product reviews override |
| `single-product-reviews.php` | Reviews override |
| `class-wc-note-order-email.php` | **Custom WC Email class** - "Order Note" email type |

### 2.3 Plugin WooCommerce Overrides (`wc-special-product-pricing`)

8 template overrides specific to the "Special Product" pricing system:

| File | Purpose |
|------|---------|
| `content-single-product.php` | Special product single page layout |
| `single-product/add-to-cart/grouped.php` | Grouped product add-to-cart for special pricing |
| `single-product/add-to-cart/variable.php` | Variable product with special pricing |
| `single-product/product-image.php` | Product image for special products |
| `single-product/short-description.php` | Short description for special products |
| `single-product/tabs/additional-information.php` | Additional info for special products |
| `single-product/tabs/description.php` | Description for special products |
| `single-product/tabs/tabs.php` | Tabs for special products |

---

## 3. CHECKOUT FLOW - 5-Step Multistep System

### 3.1 Architecture

The checkout is a **fully custom multistep implementation** built with:
- **PHP:** `checkout/form-checkout.php` (layout), `checkout/form-shipping.php`, `checkout/form-billing.php`, `checkout/review-order-detail.php`, `checkout/review-order.php`, `checkout/payment.php`
- **JavaScript:** `js/script-checkout-new.js` (~800 lines) drives all step navigation and validation
- **CSS:** `style-checkout.css` + `style-checkouts.css` (~1,454 lines combined)
- **PHP Functions:** `functions-checkout-multistep.php` (shipping logic), `ajax.php` (AJAX handlers)

### 3.2 Step Details

#### Step 0: Cart (cart.php)
- Shows on `/cart/` page
- 8-column product table (Remove, Thumbnail, Product, Price, Qty, Subtotal, Add Ons, Total)
- Customer notes textarea (persisted via cookie to checkout)
- Coupon apply modal, Account Funds apply modal
- Artwork edit modal for uploaded files
- Progress bar shows all 5 steps

#### Step 1: Shipping (form-shipping.php)
- **Two radio options:**
  1. **Pick up In-Store** - 51-C Carpenter Ct NW, Concord, NC 28027
  2. **Ship To** - Expands full shipping address form
- Ship To reveals: all shipping fields + phone, Blind Shipping checkbox, delivery method selection
- **Delivery Method sub-options:**
  - Ground Shipping (with tooltip explanation)
  - Expedited Shipping
- Expedited triggers a modal for zip code + due date entry with two sub-choices:
  - "Request Quote" (sends email to info@deadlinesigns.com)
  - "Invoice Separately" (proceeds, shipping billed later)
- Certain product IDs disable pickup or delivery options (shipping_and_packaging products)

#### Step 2: Billing (form-billing.php)
- Split into two sections:
  1. **Billing Information** - first/last name, company, address, city, state, zip, country
  2. **Order Contact Information** - email, phone
- Standard WooCommerce field rendering with custom ordering

#### Step 3: Review (review-order-detail.php)
- Full 8-column order table (same as cart)
- Billing Information summary
- Order Contact Information summary
- Shipping Information summary (pickup vs ship-to with all details)
- Delivery Method summary
- Payment Method summary
- Cart totals (subtotal, credits, coupons, shipping, processing fee, tax, total)
- Terms and conditions
- Place Order button

#### Step 4: Complete (payment.php + submit)
- Payment method selection (Stripe, Check, Account Funds)
- After payment panel is shown, "Submit Order" button triggers form submit

### 3.3 Validation (script-checkout-new.js)

- **Step 1 validation:** Requires shipping option selection. If "Ship To", requires all shipping fields filled. If delivery method selected, checks for ground/rush selection
- **Step 2 validation:** Validates all required billing fields (first name, last name, address, city, state, zip, country, email, phone)
- **Step 3 validation:** No additional validation (review only)
- **Step 4 validation:** Requires payment method selection. For Stripe, validates card element is complete
- **Between steps:** Next button is disabled until current step validates. Back button always enabled
- Navigation via tab clicks also validates (can only go to previously completed steps)

### 3.4 Stripe Integration

- **Current:** Uses Stripe plugin's native Payment Intents API JS (standard WooCommerce Stripe plugin)
- **Legacy (DISABLED):** Had a custom `stripe.js` override using the Sources API - this was disabled on 2026-03-20 because it caused "invalid token" errors during Shop As Customer operations
- The checkout JS captures Stripe card type/number/exp/cvc data via `woocommerce_checkout_params` for display in the review step
- Payment element renders in the hidden "Complete" panel, becomes visible at Step 4

### 3.5 Shipping Logic (functions-checkout-multistep.php)

Key shipping behaviors:
- **Custom shipping method:** `Deadlinesigns_Shipping_Method` (WC_Shipping_Method subclass) with base cost $0
- **Rate filtering:** Intercepts `woocommerce_package_rates` to:
  - Set cost to $0 if cart contains "Shipping and Packaging" product (ID 28889 main / 22047 reseller)
  - Apply quoted shipping price from session for rush/custom items
  - Zero out shipping if customer has own shipping account
  - Disable local pickup when Shipping & Packaging product is in cart
- **Display logic:** Shows "Please Select", "TBD", "Wait quote", "N/A", "Included", "Free", or actual price based on complex conditions
- **Order meta:** Saves `_shipping_method_deadlinesigns_label`, `_shipping_method_deadlinesigns_value`, `_shipping_method_delivery`, `_shipping_method_delivery_due_date`

### 3.6 AJAX Endpoints (ajax.php)

| Action | Purpose |
|--------|---------|
| `apply_used_fund` | Apply Account Funds credit amount to cart |
| `change_shipping_method` | Toggle shipping method flag on user meta |
| `have_shipping_account` | Flag user as having own shipping account |
| `change_delivery_shipping` | Toggle ground/rush shipping |
| `request_quote_shipping` | Request shipping quote (sends email to info@deadlinesigns.com) |
| `invoice_separately_shipping` | Mark shipping to be invoiced separately |
| `update_quote_shipping_cart` | Admin: set quoted shipping price in session |
| `deadlinesigns_remove_coupon` | Remove coupon from cart |
| `deadlinesigns_remove_fund` | Remove Account Funds from cart |

---

## 4. PAGE TEMPLATES

### 4.1 Custom Page Templates

| Template | File | Usage |
|----------|------|-------|
| Page No Footer | `page-no-footer.php` | Hides footer on desktop for full-screen content |
| Page Sidebar Left | `page-sidebar-left.php` | Shows shop sidebar on left side |
| Page with Notices | `template_page_with_wc_notices.php` | Shows WC notices + breadcrumbs, includes sign-in popup for guests |
| Page Artwork Specifications | `page-artwork-specifications.php` | Standard page with `content_only` parameter support (strips header/footer for iframe embedding) |

### 4.2 Custom Shortcodes

| Shortcode | Function | Purpose |
|-----------|----------|---------|
| `[products_top_section ids="1,2,3"]` | `wn_shortcode_products_top_section` | Displays product grid with thumbnail, title, truncated description (80 chars) |
| `[products_top_section_new ids="1,2,3"]` | `wn_shortcode_products_top_section_new` | Same but thumbnail before title, supports inner content |
| `[category_left_side slug="..." position="right" ...]` | `wn_shortcode_category_left_side` | Category showcase with product slider - category info on one side, products on other |
| `[popular_left_side slug="..." ...]` | `wn_shortcode_popular_left_side` | Popular products slider sorted by total_sales, with "View" buttons |
| `[list_all_products include_ids="..."]` | `list_all_products` | Full product catalog organized by category with hover preview popup |
| `[wpbsearch]` | `wpbsearchform` | Simple search form shortcode |
| `[show_rush_date]` | `deadlinesigns_shortcode_rush_date` | Displays requested rush delivery date from session |
| `[show_cart_info]` | `deadlinesigns_shortcode_cart_info` | Renders full cart table (8 columns) as a shortcode - used in review step |
| `[WN_MYCURRENTPOINT]` | `wn_mwb_wpr_mytotalpoint_shortcode` | Shows user's reward points converted to currency value |

---

## 5. NAVIGATION AND MENUS

### 5.1 Menu Locations

| Location | Registered | Usage |
|----------|------------|-------|
| `primary` | Yes | Main navigation bar below header |
| `footer` | Yes | Footer menu links |
| `About Us Menu` | Custom (by name) | Dropdown under "About US" in header (main site only) |
| `printshop-sidebar-menu` | Yes | Left sidebar navigation (legacy, commented out) |

### 5.2 Header Structure (header-creativeleft.php - PRIMARY)

```
Top Menu Bar:
  [Logo] [Search] [Payment Logos] [Get a Quote] [About Us (main site)] [Hello, {Name} dropdown] [Cart Icon]

Main Navigation Bar:
  [Mobile Hamburger] [Primary Menu Items]
```

- **Logo:** Static image from Wasabi S3 CDN (different for main vs reseller)
- **Search:** Product search form in top bar
- **User menu:** If logged in, shows "Hello, {FirstName}" with My Account navigation dropdown + cart icon with count badge. If not, shows "Sign in" link
- **Cart icon:** Custom shopping cart image with item count badge
- **Mobile:** Hamburger icon toggles menu with slide animation

### 5.3 MegaMenu

- The site uses `wp_nav_menu()` with `container_id => 'mega-menu-wrap-primary'`
- Menu dropdowns are handled by custom JS in `customize.js` (click-toggle `.sub-menu` with slide animation)
- There is a `css/megamenu-unused.css` file suggesting a MegaMenu plugin was previously used but replaced with custom CSS
- Mobile menu uses a slide toggle with `primary-nav-bar-mobile` trigger

### 5.4 Account Navigation

- Custom admin panel at WooCommerce > Account Menu allows drag-and-drop reordering and hiding of menu items
- Account menu items are dynamically managed via `wn-custom-menu-account` option
- Custom endpoints: `file-library` (Image Library), `enter-email` (force email entry)

---

## 6. PAGE BUILDERS

### 6.1 WPBakery (Visual Composer)

- **Active:** Yes - the child theme has a `vc_templates/vc_images_carousel.php` override
- **Usage:** Homepage product carousels, category pages, likely all major content pages
- The parent theme `printshop` is deeply integrated with WPBakery (registered shortcodes, VC elements)
- Custom product display shortcodes (`products_top_section`, `category_left_side`, `popular_left_side`) are designed to work within VC rows/columns

### 6.2 SiteOrigin

- No SiteOrigin files found in the child theme
- The parent theme may have SiteOrigin support but it is not actively used in overrides

### 6.3 The Grid

- The Grid plugin is used on the homepage (scripts dequeued on non-front pages for performance)

---

## 7. CSS CUSTOMIZATIONS

### 7.1 Font System

```
H1: Montserrat Black (900), uppercase
H2: Montserrat SemiBold (600)
H3: Muli ExtraBold (800) - also used for product titles
H4: Muli Regular (normal)
H5: Muli Regular Italic
Body: Muli Regular
Strong in descriptions: Muli ExtraBold Italic
```

### 7.2 Key CSS Features

- **style.css (1,760 lines):** Cart icon positioning, placeholder colors (white), product grid, form styling, extensive responsive breakpoints
- **style-header.css (2,368 lines):** Complete header redesign - sticky header, top menu bar layout, search positioning, user dropdown, cart badge, mobile hamburger menu, sub-menu animations
- **style-checkout.css (811 lines):** Checkout form field styling, billing field sizing, phone field layout with SMS opt-in, price override inputs
- **style-checkouts.css (643 lines):** Multistep checkout tab bar, step panels, shipping options layout, delivery method modals, sidebar order summary, responsive checkout
- **style-cart.css (85 lines):** Cart-specific table styling
- **style-dashboard.css (311 lines):** My Account dashboard tiles, profile card, order/quote lists, responsive dashboard

### 7.3 Performance Optimization

- Non-logged-in users: all stylesheets dequeued from `<head>` and re-enqueued in `wp_footer` (CSS deferred loading)
- Various plugin CSS/JS dequeued on pages where not needed (xoo-wsc, dashicons, FAQ font-awesome, The Grid on non-homepage)
- PayPal CSS only loaded on cart/checkout pages

---

## 8. EMAIL TEMPLATES

### 8.1 Custom Email Templates

| Template | Purpose |
|----------|---------|
| `customer-completed-order.php` | Custom "Your order is now completed" message with reorder instruction |
| `customer-processing-order.php` | Order submitted notification |
| `customer-new-status-order.php` | **Custom** - Generic status change notification (Prepress, In Production, Shipped) |
| `customer-readyforpickup-order.php` | **Custom** - Ready for pickup notification |
| `admin-note-order.php` | **Custom** - "A note has been added to ORDER #X" with login button |
| `customer-account-funds-increase.php` | Account funds increase notification |
| `customer-invoice.php` | Invoice email |
| `customer-reset-password.php` | Password reset email |
| `email-order-details.php` | Order details table styling |
| `email-order-items.php` | Order items rendering |
| `email-addresses.php` | Billing/shipping address display |
| `email-styles.php` | Email CSS styles |

### 8.2 Custom Email Class

`class-wc-note-order-email.php` - A custom WooCommerce email class (`wc_note_order`) triggered when admin adds an order note with attachments. Supports file attachments stored in `second_featured_img` post meta.

### 8.3 Modal Email Templates (6 files)

Used for login/register modal popups (likely AJAX-loaded):
- `email-header-login-modal.php` / `email-footer-login-modal.php`
- `email-header-login-reseller-modal.php` / `email-footer-login-reseller-modal.php`
- `email-header-register-modal.php` / `email-footer-register-modal.php`

---

## 9. CUSTOM ORDER STATUSES

The site has 5 custom order statuses beyond standard WooCommerce:

| Status | Slug | Email Trigger |
|--------|------|---------------|
| Prepress | `wc-awaiting` | Yes - customer-new-status-order.php |
| In Production | `wc-production` | Yes - customer-new-status-order.php |
| Finishing | `wc-finishing` | No |
| Installation | `wc-installation` | No |
| Shipped or Ready for Pick-up | `wc-shipping` | Yes - customer-new-status-order.php |

Additionally, "Processing" is renamed to "Order Submitted".

---

## 10. MULTISITE ARCHITECTURE

The site operates as a WordPress Multisite:
- **Main site** (`is_main_site()`): deadlinesigns.com - public-facing store
- **Reseller site**: separate branding, different logo, different product IDs for shipping
- Key differences by site:
  - Different logos (Wasabi S3 CDN)
  - Different "Shipping and Packaging" product IDs (28889 main / 22047 reseller)
  - Different header layouts (reseller has "Resources" dropdown instead of "About Us")
  - User membership checks - users must be members of current blog to access
  - WMPCS plugin handles product sync between sites

---

## 11. KEY INTEGRATIONS

| Integration | Implementation |
|-------------|----------------|
| **Stripe** | WooCommerce Stripe plugin (Payment Intents API) |
| **Account Funds** | `woocommerce-account-funds` plugin - credits system, partial payment |
| **Quote System** | `c2q` (Cart to Quote) plugin - quote endpoints, email templates |
| **Reward Points** | `mwb_wpr` (MakeWebBetter) points system |
| **PDF Invoices** | WPO WCPDF plugin with 3 custom template sets |
| **Mailchimp** | Auto-subscribe on registration (main site only) |
| **Shop As Customer** | `WC_Shop_As_Customer` - admin can shop as any user |
| **Wasabi S3** | Cloud storage for media/images |
| **Tawk.to** | Live chat widget with mobile integration |
| **NextendSocialLogin** | Social login on registration/login pages |
| **Product Filters** | `prdctfltr` plugin for shop filtering |
| **The Grid** | Homepage product grid |
| **Fancybox** | Login/register modal popups |
| **jQuery Sticky Kit** | Sticky sidebar elements |
| **WPBakery** | Page builder for content pages |

---

## 12. ADMIN CUSTOMIZATIONS

### 12.1 Product Admin

- **Product Link metabox:** Links single-sided/double-sided product variants
- **Under Title editor:** WYSIWYG editor for expanded product title (`_bhww_expanded_title_wysiwyg`)
- **WMPCS Change ID Sync Box:** Manual product ID sync between multisite blogs

### 12.2 Order Admin

- **HREF Field metabox:** Custom URL field on orders
- **Order Note for Customer metabox:** Rich text note with file upload (triggers custom email notification)

### 12.3 Account Menu Admin

- Drag-and-drop account menu reordering at WooCommerce > Account Menu
- Show/hide toggle for each menu item

---

## 13. CRITICAL FINDINGS FOR REBUILD

### 13.1 Highest Complexity Areas

1. **Checkout Flow** - The 5-step multistep checkout is the most complex custom feature (~2,000 lines of PHP + 800 lines JS + 1,454 lines CSS). Every aspect is custom.
2. **Shipping Logic** - Multiple shipping scenarios (ground, rush, quoted, own account, included, local pickup) with complex conditional display
3. **Add-On Pricing** - `_ext_price`, `_ex_option`, `_final_price`, custom price overrides create a complex pricing pipeline
4. **Admin Price Override** - Administrators can override subtotals per line item during Shop As Customer sessions

### 13.2 WooCommerce Template Versions

Most templates reference WooCommerce 2.x-3.x versions (3.0.0, 3.2.0, 3.5.0, 3.6.0). These are significantly outdated and would need updating for modern WooCommerce (8.x+).

### 13.3 Technical Debt

- Backup files scattered throughout (`-bk-3-6-2021`, `-bk-4-6-2023`, `_backup`)
- Disabled Stripe override code still present
- Duplicate parent/child theme overrides (child overrides parent which overrides WooCommerce)
- Some CSS syntax errors (stray `*/` in style.css)
- jQuery loaded twice (hardcoded in header + wp_head)
- Mixed use of `get_template_directory_uri()` and `get_stylesheet_directory_uri()` in scripts

### 13.4 Multisite Dependencies

Many functions use `is_main_site()` checks with hardcoded product IDs, making the codebase tightly coupled to the specific WordPress multisite installation.
