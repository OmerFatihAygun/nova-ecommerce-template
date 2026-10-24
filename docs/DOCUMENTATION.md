# Nova — Documentation

Nova is an e-commerce template made of two independent Nuxt 3 applications:

| App | Folder | What it is |
| --- | --- | --- |
| **Storefront** | `storefront/` | The customer-facing shop: catalog, search, product pages, cart, checkout, account area, help and legal pages. |
| **Admin** | `admin/` | The back-office dashboard: orders, products, customers, campaigns, analytics, team permissions, activity log and settings. |

Both run on built-in **demo data** — no database, API key or account is needed to start them.
Connecting your own backend is covered in [Connecting a real backend](#7-connecting-a-real-backend).

### What this is — and what it is not

Nova is a **front-end template** (JavaScript / TypeScript), not a platform theme.

* It is **not PHP** and not a WordPress, Shopify, WooCommerce, PrestaShop or Magento theme.
  There is nothing to upload to a PHP host and no database to import.
* It runs on **Node.js**: the storefront is a Nuxt server (or a static export for a demo), the
  admin is a static single-page app or a Node server.
* Stack: Nuxt 3, Vue 3, TypeScript, Pinia, Tailwind CSS 4 (no Bootstrap, no jQuery),
  `@nuxtjs/i18n`, Vitest. Icons are inline SVG (no icon font); fonts are self-hosted
  (Inter and Plus Jakarta Sans via `@fontsource`), so nothing is loaded from Google at run time.
* It ships **demo data**, not a shop back end. To sell real products you connect your own API
  (see [Connecting a real backend](#7-connecting-a-real-backend)) or keep the demo for a prototype.

---

## 1. Requirements

* **Node.js 20.19+ or 22.12+** (22 recommended — the Dockerfiles use Node 22)
* **npm 10+** (ships with Node). Use **npm**: the lock files and the `overrides` in each
  `package.json` are npm's. (pnpm and Yarn read different override settings.)

### Dependency versions

`nuxt` and `@nuxt/image` are pinned to exact versions on purpose — the ones this template was
built and tested with (Nuxt 3.21.8; `@nuxt/image` 2.0.0 in the storefront). Newer releases are
not drop-in: `@nuxt/image` 2.2 needs the Nuxt 4 tool-chain, and Nuxt 3.21.11 changes the
generated auto-import types so `npm run typecheck` reports errors. Upgrade them together, in a
branch, and run `npm run typecheck`, `npm test` and `npm run build`. `npm audit` lists
advisories for development tooling and for `node-forge` (reached only through the
development server's HTTPS helper); `sharp` and Vue are already patched through `overrides`.

### Browsers

The storefront and the admin are built for the **current and previous major versions** of
Chrome, Firefox, Safari and Edge on desktop, and the current Safari / Chrome on iOS and
Android. Internet Explorer is not supported. Layouts are designed from **320 px** wide
upwards. Languages are left-to-right only (the storefront ships English and Turkish) — there is
no RTL stylesheet.

## 2. Quick start

Each app is installed and started on its own:

```bash
cd storefront
npm install
npm run dev          # http://localhost:3000
```

```bash
cd admin
npm install
npm run dev -- --port 3001   # http://localhost:3001
```

Run them in two terminals. (Both default to port 3000; the admin's "Back to site" link
expects the storefront on 3000 — change it with `NUXT_PUBLIC_STOREFRONT_URL`.)

### Demo sign-in

| App | E-mail | Password |
| --- | --- | --- |
| Storefront | `olivia.carter@example.com` | `demo1234` |
| Admin | `admin@example.com` (Admin) or `owner@example.com` (Owner) | `demo1234` |
| Seller portal (admin app) | `seller@example.com` (Northvale Outdoor) | `demo1234` |

The login pages also have one-click "fill demo account" buttons. No account is needed to buy:
the storefront checkout also works as a guest (see "Guest checkout" below).

Accounts created on the storefront's *Create account* page are remembered in that browser
(`localStorage` key `nova:demo-accounts`, password stored only as a SHA-256 hash), so they
survive a reload and can sign in again. "Continue with Google / Apple / Facebook / passkey"
signs in the demo shopper. Both are demo conveniences — connect your identity provider in
`stores/auth.ts` (`signIn`, `signUp`, `signInWithProvider`, `restore`). The account pages use
their own layout, `layouts/auth.vue`.

### Scripts

| Command | Storefront | Admin |
| --- | --- | --- |
| `npm run dev` | dev server | dev server |
| `npm run build` | production build (Node server) | production build (Node server) |
| `npm run generate` | static export (demo hosting) | static export |
| `npm run preview` | preview a build | preview a build |
| `npm run typecheck` | `nuxt typecheck` | `nuxt typecheck` |
| `npm test` | unit tests (Vitest) | — |

The storefront build runs a TypeScript check (`typeCheck: true`) and takes a few minutes.

## 3. Project structure

Both apps follow the standard Nuxt layout:

```
pages/          Routes (one file = one URL)
components/     UI, grouped as atoms / molecules / organisms (storefront)
composables/    Data layer (useStore*/useAdmin*) and shared logic
stores/         Pinia stores (cart, auth, favourites, ...)
layouts/        Page shells
middleware/     Route guards (auth)
plugins/        Cookie consent, analytics, auth bootstrap
mock/           In-memory demo database + seed data
i18n/locales/   Translation files (storefront: en, tr · admin: en)
assets/css/     main.css — design tokens and global styles
public/         Static files (favicon, demo images)
server/         Storefront only: sitemap.xml, robots.txt, address & Turnstile endpoints
tests/          Storefront only: Vitest unit tests
data/           Static lists that are not demo records (cookie categories, regions, home media)
utils/          Small pure helpers (validation, formatting, translation helpers) — auto-imported
types/          Shared TypeScript types
```

Components, composables, `utils/` exports and Pinia stores are **auto-imported** by Nuxt, so
most files have no `import` lines for them. Pinia stores are written as setup stores
(`defineStore('cart', () => { … })`: `ref` for state, `computed` for getters, plain functions
for actions), one file per concern. Component names do not include their folder:
`components/organisms/TheHeader.vue` is used as `<TheHeader />`.

### How data flows

Every screen reads and writes data through the same four layers. Only the bottom one knows
the data is fake:

```
pages/*.vue, components/*.vue        what the shopper sees
        │
use*  composables (usePlp, usePdp,   page logic: filters, sorting, mapping API types to
useCatalog, useOrders, useSearch…)   the UI's Product type, SSR caching
        │
stores/*.ts (Pinia)                  state shared between screens: cart, auth, checkout,
                                     favourites, addresses, returns, support
        │
useStore* / useAdmin* composables    THE DATA LAYER — one function per backend call,
                                     returning the shape a real API would return
        │
mock/db.ts  ←  mock/seed/*.ts        in-memory demo database (replace with your API)
```

| Name pattern | Layer | Example | Replace when connecting a backend? |
| --- | --- | --- | --- |
| `useStore*.ts` (storefront), `useAdmin*.ts` (admin) | Data layer | `useStoreCatalog().listProducts()` | **Yes** — swap the function bodies |
| `use*.ts` (everything else) | Page logic / UI helpers | `usePlp`, `useCatalog`, `useCurrency` | No |
| `stores/*.ts` | Shared state (Pinia) | `useCartStore()` | No (sign-in: see below) |
| `mock/` | Demo database | `mock/seed/catalog.ts` | Delete once every data-layer function is connected |

A few names are worth knowing:

* `useCatalog()` turns the data layer's `ApiProduct…` types into the `Product` type the cards
  and product page use, and offers ready-made lists (by category, brand, seller, deals).
* `useCategories()` is the category tree from the data layer (mega menu, category pages);
  `data/category-tree.ts` is the same tree available synchronously for breadcrumbs.
* `stores/favorites.ts` is the shopper-facing "Favourites"; its data layer is
  `useStoreWishlist.ts` — the same concept under the word the backend uses.
* Text in scripts goes through `tt('key')`, text in templates through `$t('key')`; demo
  content (category names, banners, product names) through `contentText()` — see
  [Languages](#languages).

### Where do I change…?

| What | Storefront file | Admin file |
| --- | --- | --- |
| Colours, fonts, radii, dark mode | `assets/css/main.css` | `assets/css/main.css` |
| Logo / shop name | `components/organisms/TheHeader.vue`, `TheFooter.vue` | `layouts/default.vue` |
| Header links, department bar | `components/organisms/TheHeader.vue` | — |
| Mega menu | `composables/useNavigation.ts` (built from the category tree) | — |
| Support phone, e-mail and opening hours | `data/site-contact.ts` | — |
| Footer social links (Instagram, X, YouTube) | `data/site-contact.ts` (`SOCIAL_LINKS`; empty hides the icon) | — |
| Footer columns | `components/organisms/TheFooter.vue` | — |
| Footer page tabs (help, corporate, legal) | `components/molecules/InfoSectionNav.vue` | — |
| Home page sections and their order | `pages/index.vue` (see [Home page sections](#home-page-sections)) | — |
| Banners, coupons, campaigns (demo) | `mock/db.ts` → `buildBanners()`, `buildCoupons()`, `buildCampaigns()` | `mock/db.ts` |
| Products, categories, brands, sellers (demo) | `mock/seed/catalog.ts`, `mock/seed/vendors.ts` | `mock/seed/catalog.ts`, `mock/seed/vendors.ts` |
| Demo customers and sign-in accounts | `mock/seed/people.ts`, `stores/auth.ts` | `mock/seed/people.ts`, `stores/auth.ts` |
| Free-shipping threshold, shipping fee | `stores/cart.ts` (`FREE_SHIPPING_THRESHOLD`, `SHIPPING_FEE`) | — |
| Currency | `composables/useCurrency.ts` | `composables/useCurrency.ts` |
| Help centre articles | `composables/useHelpCenter.ts` (texts in `i18n/locales/*.json`) | — |
| Legal pages | `pages/legal/*.vue` (texts in `i18n/locales/*.json`) | — |
| Checkout steps and payment | `stores/checkout.ts`, `components/organisms/Checkout*.vue` | — |
| Demo payment rules (test cards) | `composables/useStoreOrders.ts` | — |
| Admin sidebar items and who sees them | — | `layouts/default.vue` (`NAV_ITEMS`) |
| Admin roles and permissions matrix | — | `composables/useAdminTeam.ts` |
| Translations | `i18n/locales/en.json`, `tr.json` | `i18n/locales/en.json` |
| Optional settings (API keys, site URL…) | `.env.example` | `.env.example` |

## 4. Configuration

Copy `.env.example` to `.env` and fill in only what you use. Nothing is required.

### Storefront

| Variable | Purpose |
| --- | --- |
| `NUXT_PUBLIC_SITE_URL` | Public address of the site — used for absolute URLs in `sitemap.xml`, `robots.txt` and social-share tags. **Set this in production.** |
| `NUXT_PUBLIC_IMAGE_BASE_URL` | Optional CDN prefix for images. Empty = served from `public/`. |
| `NUXT_PUBLIC_ADMIN_URL` | Address of the admin app (the seller portal). The *Sell on Nova* page (`/sell`) links its sign-in page; empty = those links are hidden. |
| `NUXT_PUBLIC_API_BASE_URL` | Base address of your backend API, read by `useApi()` (`composables/useApi.ts`). Unused by the demo data. |
| `NUXT_ALLOW_EMBEDDING` | Set to `true` **only for a demo** that must run inside an iframe (a marketplace live preview). By default the security headers forbid framing (`X-Frame-Options: DENY`). Read at build time. |
| `NUXT_GOOGLE_MAPS_API_KEY` / `NUXT_PUBLIC_GOOGLE_MAPS_API_KEY` | Address autocomplete and map. Without a key the address form still works; only the map and suggestions are hidden. |
| `NUXT_PUBLIC_TURNSTILE_SITE_KEY` / `NUXT_TURNSTILE_SECRET_KEY` | Cloudflare Turnstile bot protection on public forms. With both empty the widget is not rendered. |
| `NUXT_PUBLIC_GTM_ID`, `NUXT_PUBLIC_GA_ID`, `NUXT_PUBLIC_META_PIXEL_ID` | Analytics. Loaded only after the visitor accepts the matching cookie category. |

### Admin

| Variable | Purpose |
| --- | --- |
| `NUXT_PUBLIC_STOREFRONT_URL` | Target of the sidebar's "Back to site" link. |
| `NUXT_PUBLIC_API_BASE_URL` | Base address of your backend API, read by `useApi()` (`composables/useApi.ts`). Unused by the demo data. |

## 5. Customising

### Brand colour and design tokens

All design tokens are CSS variables at the top of `assets/css/main.css` in each app. To
rebrand, change the `--color-brand-50 … --color-brand-900` scale (the default is orange,
`#ff5e00` at 600). Radii, shadows, fonts (`--font-sans`, `--font-display`) and spacing
tokens live in the same block. Update `theme-color` in the storefront's `nuxt.config.ts`
and replace `public/favicon.svg`, `apple-touch-icon.png` and `og-default.png`.

### Dark mode

Both apps support light, dark and "follow system". The visitor's choice is stored in the
`nova_theme` cookie (see `composables/useTheme.ts`) and applied to `<html data-theme>` by a tiny inline script
**before first paint**, so there is no flash.

The dark palette is one block of variables at the end of each `main.css`
(`[data-theme="dark"] { … }`). Surfaces that are dark by design (admin sidebar, tooltips,
the primary dark button) intentionally keep their look. To restyle dark mode, edit that
block only.

The toggle is in the storefront footer and mobile menu, and in the admin sidebar.

### Languages

The storefront ships in **English** (default) and **Turkish**; the admin ships in English.
Both apps keep **every** interface text in JSON files under `i18n/locales/` — the storefront
about 2,100 messages per language, the admin about 1,250 — so no screen has hard-coded text. The language picker keeps the URL
(`strategy: 'no_prefix'`), saves the choice in the `nova_lang` cookie and reloads the page, so
the server renders the next page in the chosen language too.

How the storefront's text is wired:

* Templates use `$t('key')`. Scripts, composables and Pinia stores use `tt('key')`
  (`utils/tt.ts`), which works outside components and falls back to English. The admin works
  the same way (its own `utils/tt.ts` and `i18n/locales/en.json`).
* Counts use two forms, `"{p0} order | {p0} orders"`, with the count as the last argument:
  `$t('key', { p0: n }, n)` / `tt('key', { p0: n }, n)`.
* Labels in fixed lists (table columns, status names, filter tabs) are getters —
  `get label() { return tt('…') }` — so they are read in the current language.
* Dates and numbers follow the language through `appLocaleTag()` (`utils/locale-tag.ts`).
* Category names and banner texts come from the demo data in English; `contentText()`
  (`utils/content-text.ts`) shows the translation from the `content` section of the locale file.
  A real backend would return them in the shopper's language.
* Catalog content — product names, descriptions, reviews, seller descriptions — is not
  translated: it comes from your own database in your language.

**Add a language**

1. Copy `i18n/locales/en.json` to `i18n/locales/xx.json` and translate the values (keep the keys
   and the `{placeholders}`).
2. Add `{ code: 'xx', name: 'Name', file: 'xx.json' }` to `i18n.locales` in `nuxt.config.ts`.
3. Add the date/number tag to `utils/locale-tag.ts` and `'xx'` to `LocaleCode` in `i18n/loader.ts`.

To ship a single language, keep one entry in `i18n.locales`; the picker hides itself when there
is nothing to choose.

**Translate the admin** — the same two steps in the admin app: copy `i18n/locales/en.json` to
`xx.json`, translate it, and add the locale to `i18n.locales` in the admin's `nuxt.config.ts`.

**Load translations from an API instead** — `i18n/loader.ts` documents the one-line swap.

### Currency

Prices are formatted by `composables/useCurrency.ts`; the demo currency is USD and is set
in the mock database (`CURRENCY` in `mock/db.ts`).

### Demo images

Product, brand, category, banner and home-page images in `public/demo/` are original SVG
artwork drawn for this template — a soft gradient with a line icon of the subject. No
photographs are included, so there is nothing to license separately. Replace the files (keep the
names) or change the paths in `mock/seed/catalog.ts`.

The author's hosted live demo is built with `NUXT_DEMO_PHOTOS=true` and shows preview photos in
some of these slots. Those photos are not part of the download; without the variable every slot
uses the SVG art above (see `utils/demo-art.ts`).

`tools/gen_placeholders.py` regenerates all of them (`python3 tools/gen_placeholders.py public`,
and `--home public` for the home-page slots). Add `--labels` to print the slot name and
recommended size on every image — handy as a shot list when you prepare your own photos.

### Home layouts

The template ships **four home pages**. They are assembled from the same blocks, so the data,
translations and dark mode work in all of them.

| Layout | Route | File | Fits |
|---|---|---|---|
| Marketplace (default) | `/` | `pages/index.vue` | General store: hero, mega deals, best sellers, department cards |
| Fashion lookbook | `/home/fashion` | `pages/home/fashion.vue` | Clothing, shoes, accessories: editorial hero, "new in", two image + product stories, trending tabs |
| Deals | `/home/deals` | `pages/home/deals.vue` | Outlet / discount store: deal of the day, price tiles, biggest discounts, coupons, under-$100 rail |
| Single product | `/home/single-product` | `pages/home/single-product.vue` | One product launch: gallery, options, specs, reviews, FAQ, closing call to action |

**Make one of them your home page.** Copy the `<script setup>` and `<template>` of the layout
file into `pages/index.vue`, then delete the `pages/home/` folder (`LayoutHero.vue` stays in
`components/organisms/` — the fashion layout uses it). To keep the default home and use another
layout only as a landing page, leave the file where it is and add its path to `STATIC_PATHS`
in `server/routes/sitemap.xml.ts` (the layouts are left out of the sitemap on purpose: they are
alternatives to `/`).

**Configure them.**

* *Fashion* — the four departments are the `DEPARTMENTS` list at the top of the file (category
  slugs). Names and photos come from the category tree, so they follow what you set in the admin.
* *Deals* — nothing to set. It lists products that have an "old price" higher than the current
  price (`useHomeRails().loadDeals`); the home page's deal rail uses the same function.
* *Single product* — set `FEATURED_SLUG` at the top of the file to the slug of any product. Price,
  options, stock, specifications and reviews are read through `usePdp()`, the same composable as
  the product page. An unknown slug returns a 404 with a message that names the constant.

**Remove the demo switcher.** The footer's "Home layouts" row (`HOME_LAYOUTS` in
`components/organisms/TheFooter.vue`, plus the `<nav>` that renders it) exists so visitors of the
demo can compare the layouts. Delete both in your own store.

### Home page sections

`pages/index.vue` assembles the home page from these blocks, top to bottom:

* **Hero** (`HomeHeroCarousel.vue`) — a thin promo strip, the sliding banner and two side
  banners. The headline is written over the image by the page, so banner artwork should carry
  no text of its own. Banners come from the `home-hero`, `home-side` and `home-promo` placements
  in the admin's Banners screen (in the demo: `buildBanners()` in `mock/db.ts`).
* **Assurance row** (`HomeAssurance.vue`) and **shortcut boxes** (`CategoryCircleStrip.vue`).
* **Three columns** (`HomeFeatureColumns.vue`) — "More reasons to shop" (`home-reasons`),
  "Mega Deals" with a countdown to the end of the `home-mid` banner, and "In Focus"
  (`home-focus`).
* **Best Sellers** (`HomeTopRanked.vue`) — a product rail with a small rank on the photo.
* **Department cards** (`HomeCategoryGrid.vue`), **featured brands**
  (`HomeFeaturedBrands.vue`) and product rails (recently viewed, new arrivals,
  recommendations).

**Coupons** are a page (`/coupons`, `HomeCouponStrip.vue`), linked from the department bar next
to Today's Deals. Codes that have not started, have expired or are used up are left out.
Visitors copy a code and paste it in the cart.

#### Optional sections

These ship ready to use but are not on the default home page. Add one to `pages/index.vue`
where you want it:

* **Deal of the day** (`HomeDealOfDay.vue`) shows the first discounted product large, with
  four more beside it. Pass the real end date of your campaign as the `endsAt` prop (an ISO
  date-time); without it the demo counts down to the next local midnight. A date in the past
  hides the countdown.
* **Deals by budget** (`HomeBudgetTiles.vue`) links to `/category/deals?min=…&max=…`. Change the
  price limits with the `limits` prop (default `[100, 250, 500, 1000]`) and the currency with
  `currency`.
* **Featured collections** (`HomeCollections.vue`), **promo cards** (`HomePromoTrio.vue`),
  **editorial split** (`HomeEditorialSplit.vue`), **trending tabs** (`HomeTrendingTabs.vue`),
  **buying guides** (`HomeBuyingGuides.vue`), **customer reviews** (`HomeTestimonials.vue`),
  **photo gallery** (`HomeGallery.vue`) and a **seller invitation** (`HomeSellerInvite.vue`).

### Home page images

The optional editorial blocks (collections, promo cards, editorial splits, the customer
gallery) read their images from one place: `data/home-media.ts`. The files live in
`public/demo/home/`; the table below lists the recommended size for each slot (run the
generator with `--labels` to print the same information on the images).

| Slot | Recommended size | Used by |
|---|---|---|
| `collection-work` | 1000 × 1000 | Featured collections — large tile |
| `collection-sound`, `collection-summer`, `collection-skin`, `collection-home` | 800 × 800 | Featured collections — small tiles |
| `promo-sale`, `promo-new`, `promo-free` | 1200 × 600 | Promo cards |
| `editorial-home`, `editorial-outdoor` | 1200 × 1000 | Editorial split blocks |
| `gallery-1` … `gallery-8` | 600 × 600 | "Styled by you" gallery |

To use your own photos, save them under the same names in `public/demo/home/` and change
`HOME_IMAGE_EXT` in `data/home-media.ts` from `'svg'` to `'jpg'` (or `'webp'`). Headlines,
links and product picks for every block are edited in the same file. Keep the light/dark text
legible: promo and collection cards draw a dark gradient over the photo, so avoid very dark
photos with text in the bottom third.

## 6. Features

### Storefront

* Home with a hero and side banners, shortcut boxes, Mega Deals, a Best Sellers rail,
  department cards, featured brands and product rails. Recently viewed sits near the top once
  the shopper has browsed. More home sections ship as optional components (see above), and
  **three alternative home layouts** (fashion, deals, single product) are included — see
  "Home layouts" above.
* Category and brand listing pages with filters, sorting and infinite scroll
* Product page: gallery, variants, specs, reviews, installments, coupons, sticky buy bar,
  "Sold by" link to the seller's store
* Search with suggestions, compare, wishlist, recently viewed, recommendations
* Cart drawer + cart page, coupon codes, multi-step checkout (address, shipping, payment, review)
* **Guest checkout** — cart and checkout work without an account; the order is found again
  on **Track an order** (`/track-order`) with its number and e-mail, and joins the account
  when the shopper registers with the same e-mail
* **Marketplace** — *Sell on Nova* landing page with a seller application form (`/sell`) and
  a public page per seller (`/store/<slug>`)
* **Coupons** page (`/coupons`) and an **Ask Nova** help chat that answers from the help-centre
  articles (`components/organisms/SupportChat.vue`)
* Account area: orders and tracking, returns, addresses, payment methods, coupons,
  notifications, support tickets, security, profile
* Login, register and **forgot password** (demo: simulates sending the reset e-mail)
* Help centre, solution centre, legal pages (generic sample text with a "Sample text" notice —
  replace it with documents reviewed by a lawyer; the notice lives in
  `components/molecules/LegalPage.vue`), corporate pages (careers, warranty, quote, become a seller)
* Cookie consent with categories; analytics loads only after consent
* Dark mode, English and Turkish, SEO tags, `sitemap.xml`, `robots.txt`, security headers

### Admin

* Dashboard and analytics
* Orders, payments, returns, shipping
* Products (list + detail editor), categories, brands
* Customers, reviews, support tickets
* Campaigns, coupons, discounts, banners
* Agreements and legal documents
* **Permissions** — team table (invite, change role, suspend/reactivate, remove) and a
  role-by-area access matrix
* **Activity Log** — filterable audit trail (by admin, area, action, search)
* **Sellers** — store applications with approve / reject
* **Payouts** — commission per department, a 14-day hold after delivery, a payout every
  Monday, a six-week settlement calendar and per-store statements
* **Seller portal** — the same app signed in with the seller role shows only that store:
  its dashboard, products, orders, payments, payouts and returns
* Settings, dark mode (English interface; every text in `i18n/locales/en.json`, ready to translate)

### Marketplace and sellers

Nova is a marketplace: every product belongs to a seller (a *vendor*).

* Vendors and the brands they sell are in `mock/seed/vendors.ts` (both apps). The product page
  links each product to its seller's store page.
* Commission rates per root department and the settlement clock (14-day hold, Monday payouts)
  are in `admin/mock/seed/commission.ts`; the Payouts screen computes everything from the
  demo orders (`admin/composables/useAdminPayouts.ts`).
* The seller account is bound to its store through `vendorId`. `admin/composables/useVendorScope.ts`
  narrows products, orders, payments and returns to that store, and the sidebar and route guard
  hide the operator screens.
* Set `NUXT_PUBLIC_ADMIN_URL` on the storefront so *Sell on Nova* can link the seller sign-in.


## 7. Connecting a real backend

The demo is designed to be unplugged. Everything the UI reads or writes goes through the
`useStore*` composables (storefront) and `useAdmin*` composables (admin). They currently
read an in-memory database (`mock/db.ts`) but return the shape a real API would, so:

1. Open the composable for the feature you are connecting, e.g. `composables/useStoreCatalog.ts`.
2. Replace the function bodies with calls to your API. `useApi()` (`composables/useApi.ts`, in
   both apps) is a `$fetch` client already pointed at `NUXT_PUBLIC_API_BASE_URL`, so no address is
   hard-coded: `const product = await useApi()<ApiProduct>(`/products/${id}`)`.
3. Keep the function signatures and return types — pages, components and Pinia stores don't change.
4. Once every composable is connected, delete `mock/`.

Where to start:

| Concern | File |
| --- | --- |
| Sign-in (storefront) | `stores/auth.ts` |
| Sign-in (admin) | `stores/auth.ts`, `middleware/auth.global.ts` |
| Catalog, orders, reviews, returns, tickets | `composables/useStore*.ts` |
| Admin team & roles | `admin/composables/useAdminTeam.ts` |
| Admin activity log | `admin/composables/useAdminActivity.ts` |
| Sellers, applications, payouts | `admin/composables/useAdminSellers.ts`, `useAdminPayouts.ts`, `useVendorScope.ts` |
| Seller applications (storefront) | `composables/useSellerApplications.ts` |
| Guest checkout and order lookup | `composables/useGuestSession.ts`, `composables/useStoreOrders.ts` |
| Payment | `components/organisms/CheckoutPayment.vue` (the demo does not charge cards) |

### Payments and e-mail

The demo approves test cards and sends no e-mail (see "Test cards" below). To go live, replace
two things — the screens stay as they are.

**Payments (example: Stripe).**

1. On the server, create a payment for the order: add `server/api/payments/intent.post.ts`, read
   the order from your database, call `stripe.paymentIntents.create({ amount, currency,
   metadata: { orderId } })` with your secret key (keep it in a private `runtimeConfig` entry,
   never in `public`) and return `client_secret`.
2. In the browser, swap the card inputs in `components/organisms/CheckoutPayment.vue` for Stripe's
   card element, so card numbers never reach your own server. The checkout store
   (`stores/checkout.ts`) already waits for a payment result through `initiatePayment()` in
   `composables/useStoreOrders.ts`: make that function fetch the client secret and confirm the
   payment, and return the same result shape (`isCompleted`, `requiresRedirect`, …).
3. Mark the order paid from a **webhook** (`server/api/payments/webhook.post.ts`), not from the
   browser's answer. Keep the `payment_completed` analytics event where it is.

**E-mail (example: any SMTP service or a transactional e-mail API).**

1. Add `server/utils/mail.ts` with one `sendMail({ to, subject, html })` function for your provider.
2. Call it where the order is created (your endpoint behind `checkout()` in `useStoreOrders.ts`)
   for the order confirmation, and from your ticket and password-reset endpoints. The texts are in
   the translation files, so e-mails can reuse them.
3. The confirmation page says "We sent the receipt to …" only because the demo pretends; keep it
   once your server really sends the message.

### Things the demo deliberately does not do

* **Staff roles are enforced in the demo — in the browser.** Every person on the Permissions
  screen has one role (Owner, Admin, Manager, Support agent, Analyst), and the matrix there
  decides what that role can do: an area is *full* (view and change), *view* (read only, with a
  "View only" notice) or *none* (hidden from the menu and the route is blocked). The same check
  runs first in every function that changes data (`composables/useAdminAccess.ts`,
  `assertCanEdit`). Change a person's role on the Permissions screen and their access changes.
  Sign in as `owner@`, `admin@`, `priya.nair@` (Manager), `marcus.webb@` (Support),
  `sofia.reyes@` (Analyst) — password `demo1234` — to compare. Suspended people cannot sign in; an
  invited person is activated by their first sign-in. Because it all runs in the browser, enforce
  the same matrix on your server and keep these checks for the interface. The seller role is
  separate: it is limited to one store by `useVendorScope`, never sees the customer's card, and
  cannot change an order that also holds other sellers' items ("Shared order").
* **Seller applications cross between the two sites through a link.** The shop and the admin are
  separate sites that share no storage. After applying on `/sell`, the applicant gets an "Open it
  in the admin queue" button (when `NUXT_PUBLIC_ADMIN_URL` is set); the admin's Sellers page reads
  the application from the address, validates it and adds it to the queue. Approving it opens a
  store: the applicant can sign in with their e-mail and `demo1234` and sees an empty seller
  portal. Everything resets on a full reload. With a real back end, the form posts to your API
  and both screens read the same list.
* **Payouts are calculations only.** No money moves; the statements are derived from the demo
  orders.
* **Guest orders** are remembered in a cookie (`nova_guest_receipts`) and in `localStorage`, so
  the receipt and Track an order survive a refresh. Attaching them to an account by e-mail is
  done without verification in the demo — verify the e-mail address in your API first.
* **Team members and the activity log live in memory** and reset on a full page reload.
  Activity entries added from the Permissions page appear in the log while you navigate
  inside the app.
* **No payments are processed** and no e-mails are sent (forgot-password only simulates it).
  Alternative sign-in buttons on the login page (e-mail link, passkey, SMS code) show a
  "not configured" notice — wire them to your own provider.
* **Test cards** (any expiry date in the future, any CVC): any number is approved; a number
  ending in `0002` is declined and the order page offers "Complete Payment" for the same
  order; a number ending in `3220` goes through the 3-D Secure step and comes back paid.
  The rules live in `initiatePayment` in `composables/useStoreOrders.ts`.
* The demo session is a plain cookie. Replace it with real, server-verified sessions
  before going live.

## 8. Deployment

### Storefront in production — needs a Node host

The storefront has server routes (`sitemap.xml`, `robots.txt`, address autocomplete, Turnstile
verification), so deploy it as a **Node server**:

```bash
npm run build
node .output/server/index.mjs        # PORT and HOST env vars are respected
```

Or with Docker (the included `Dockerfile`):

```bash
docker build -t nova-storefront .
docker run -p 3000:3000 --env-file .env nova-storefront
```

Any Node-capable host works (a VPS, Render, Railway, Fly.io, Google Cloud Run, ...).
Set `NUXT_PUBLIC_SITE_URL` to the public address. Security headers are set in
`nuxt.config.ts`; the Content-Security-Policy is **report-only** — see the comment there
before switching it to enforcing.

### Hosting a live demo (both apps, static, free)

For a public demo, both apps export to plain files — no server needed:

```bash
npm run generate        # in storefront/ and in admin/  →  .output/public
```

Upload each `.output/public` folder to a static host (Netlify, Cloudflare Pages,
GitHub Pages, S3 + CDN) — one site per app, e.g. `demo.example.com` and
`admin.demo.example.com`. Notes:

* The storefront export pre-renders every page, product and category (about 1,400 files,
  mostly optimised images) as flat `page.html` files (`nitro.prerender.autoSubfolderIndex`
  is `false`). Cloudflare Pages redirects `page/index.html` to `page/` and that loops
  with the trailing-slash redirect, so the home page flashes and then fails. **Do not**
  add `/* /200.html 200` either: Pages applies it even when `index.html` exists and the
  browser lands on `/200`. **Netlify:** you may add `/* /200.html 200` (Netlify serves
  real files first). Other hosts: rewrite unknown paths to `/index.html` or `/200.html`.
* A static storefront has **no server routes**: `sitemap.xml`, `robots.txt`, address
  autocomplete and Turnstile verification are not available. That is fine for a demo, not
  for a live shop.
* Set `NUXT_PUBLIC_SITE_URL` and `NUXT_PUBLIC_ADMIN_URL` (storefront) and `NUXT_PUBLIC_STOREFRONT_URL` (admin) **before**
  running `generate`, so absolute links point at your demo addresses.
* Everything the demo does lives in the visitor's browser, so a public demo is safe to share:
  nothing is written to a server, and every visitor starts from the same seed data.
* **Showing the demo in an iframe** (marketplace live previews do this): run
  `NUXT_ALLOW_EMBEDDING=true npm run generate` for the storefront so the generated headers do not
  forbid framing, and do not add an `X-Frame-Options` header at the host. Inside a third-party
  iframe some browsers (Safari especially) refuse to keep cookies. The theme choice and the
  cookie-consent answer are therefore also saved in `localStorage` and read back from there, and the
  demo sign-in is held in memory, so the demo keeps working inside the frame (only a full reload
  of the admin returns to its sign-in page). Leave this variable unset on a real shop.

### Admin — static hosting is enough

The admin renders in the browser on demo data, so it can be exported as plain files:

```bash
npm run generate        # output: .output/public
```

Upload `.output/public` to any static host (Netlify, Cloudflare Pages, GitHub Pages, S3 + CDN).
Because it is a single-page app, configure the host to serve `index.html` for unknown
paths (a "SPA fallback" / rewrite `/*` → `/index.html`). It can also run as a Node server
exactly like the storefront, with the same Dockerfile.

The admin sends `noindex, nofollow`; keep it on its own (sub)domain, ideally behind your
own access control.

## 9. Testing

```bash
cd storefront
npm test
```

The suite covers the cart, checkout, coupon and favourites stores, the demo card payment
rules (approve, decline, 3-D Secure), guest-order ownership, stored orders, help-chat matching
and the product-attribute helpers. `vue-tsc` type-checking runs as part of `npm run build`; run
`npx nuxi typecheck` for a standalone check in either app.

## 10. Troubleshooting

| Symptom | Fix |
| --- | --- |
| `npm install` fails with "text file busy" in Docker | The Dockerfiles install with `--maxsockets 1` and retry once for exactly this reason. |
| Address map or suggestions missing | Set a Google Maps API key (see §4). The form works without it. |
| Admin "Back to site" opens the wrong address | Set `NUXT_PUBLIC_STOREFRONT_URL`. |
| Team / activity changes disappear | Expected — demo data is in memory and resets on reload. |
| City is a text box, not a dropdown | The city list was removed; only states/regions are predefined (`data/geo-regions.ts`). City is free text. |
| Port 3000 already in use | `npm run dev -- --port 3001` |
| `npm run typecheck` fails with "implicitly has an 'any' type" after updating packages | You moved Nuxt past the pinned version (see "Dependency versions"). Restore the pinned versions from `package.json` and run `npm ci`. |
| `npm run generate` ends early with no `_nuxt` folder | `@nuxt/image` was updated to 2.2 or newer. Restore `@nuxt/image` 2.0.0 and run `npm ci`. |
| `npm install` prints a list of audit warnings | See "Dependency versions". Do not run `npm audit fix --force` — it upgrades Nuxt past the tested version. |
| Brand colour looks too light for small text | The brand orange `#ff5e00` is 3.06:1 on white (fine for buttons and large text). Small text uses `brand-700` (`#b04100`, 5.4:1). To change the brand, edit the `--color-brand-*` scale in `assets/css/main.css` and keep `brand-700` at 4.5:1 or more on white. |

## 11. Support & updates

Support is provided through the item page of the marketplace you bought from (Envato Market
or TemplateMonster) within the support period of your licence. See `CHANGELOG.md` for what changed in each release.
