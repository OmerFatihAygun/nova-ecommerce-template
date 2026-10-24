# Changelog

## 1.1.0 — 2026-10-22

**Marketplace**
* Products belong to sellers. The product page shows "Sold by" and links to the seller's public
  store page (`/store/<slug>`)
* *Sell on Nova* landing page with a seller application form (`/sell`). Set
  `NUXT_PUBLIC_ADMIN_URL` to show its seller sign-in links; without it they are hidden
* Admin: **Sellers** (store applications, approve / reject) and **Payouts** (commission per
  department, 14-day hold after delivery, Monday payouts, settlement calendar, statements)
* Admin: **seller portal** — sign in as `seller@example.com` and the admin shows only that
  store's dashboard, products, orders, payments, payouts and returns

**Storefront**
* Guest checkout: cart and checkout without an account, a receipt for the order, and
  **Track an order** (`/track-order`) by order number and e-mail. Guest orders join the account
  when the shopper registers with the same e-mail
* Coupons page (`/coupons`) and an **Ask Nova** help chat that answers from the help centre
* Home page reorganised: hero with side banners, assurance row, shortcut boxes, Mega Deals,
  "In Focus", Best Sellers, department cards and featured brands. Deal of the day, budget
  tiles, collections, editorial blocks, testimonials and the gallery still ship as optional
  sections
* Demo card payments follow the documented test cards: any number is approved, `…0002` is
  declined (the order page then offers "Complete Payment" for the same order) and `…3220` goes
  through 3-D Secure. Previously every card order ended on "waiting for payment confirmation",
  and paying again after a decline created a second order
* The hosted demo can show preview photos (`NUXT_DEMO_PHOTOS=true`, see `utils/demo-art.ts`);
  the download keeps the SVG artwork in every slot. Home hero and side banner artwork no
  longer carries text, because the page writes the headline over it

**Performance and accessibility**
* Faster first screen: responsive WebP images with per-screen sizes, the first home slide preloaded,
  variable fonts, sections below the first screen hydrate only when scrolled near
  (`.defer-render` + `hydrate-on-visible`), the demo database builds its tables on first use,
  locale text is no longer bundled twice, and links prefetch on hover/focus instead of on sight
* Product cards load only their first photo until hovered (the hover gallery used to download every photo)
* Accessibility: the brand orange is a shade darker (`#CC4A00`) so white text on it reaches the 4.5:1
  contrast of WCAG AA, carousel dots have 24 px touch targets, accessible names start with the text
  shown on the control (WCAG 2.5.3)
* The demo build uses WebP photos (`tools/convert_demo_photos.mjs`)

**Languages**
* The storefront is fully translated into **Turkish**: every label, message, help article, legal
  page, category name and campaign text (about 2,000 messages). English stays the default
* German, Spanish, French, Italian and Japanese were removed — they only covered a few labels.
  The admin is English. Adding a language is one JSON file (see "Languages" in the documentation)
* Dates and numbers follow the language; the language picker reloads the page so everything,
  including the server-rendered HTML, switches at once

**Account screens**
* Sign in, create account and forgot password use a focused layout (`layouts/auth.vue`): brand
  strip, logo, language and theme pickers, one centred column and the legal links — no shop
  header or footer. "Sign in | Create account" tabs, "stay signed in", "Continue with" Google,
  Apple, Facebook and passkey (`SocialSignIn.vue`; the demo signs in the demo shopper), and
  "continue as guest" under the form
* Password field with a show/hide button and, on "Create account", a live strength meter
* The "sign in to continue" dialog offers clear option rows; the guest button has a new look
* **Fix:** an account created on the registration page was lost on reload or in a new tab, so
  the visitor appeared signed out and could not sign in with it again ("incorrect e-mail or
  password"). Demo accounts are now remembered in the browser (`nova:demo-accounts`, the
  password only as a SHA-256 hash), the server no longer drops a session it cannot see, and the
  browser restores it right after hydration
* **Fix:** "Stay signed in" was missing; unticked, the session now ends with the browser session
* **Fix (static demo):** after any full page load the shopper looked signed out, even with the
  demo account — a generated page carries the build-time state, so the browser never read its
  session cookie. The browser now always checks the cookie once and applies the session after
  hydration. Checkout pre-selects the saved address and the header reloads favourites as soon
  as the session is known (they used to run once, too early)

**Code standards**
* Every admin text now goes through `$t()` / `tt()` — about 1,250 messages in
  `admin/i18n/locales/en.json` (screens, tables, filters, toasts, errors, dialogs). Translating the
  admin is one JSON file. Counts use plural forms ("1 order", "6 coupons")
* The last hard-coded storefront texts moved to the locale files: checkout validation messages,
  payment method names, shipping labels on the product page, cookie purposes and durations,
  opening hours, the default wishlist name. They were English in Turkish mode
* All Pinia stores (9 storefront, 1 admin) are setup stores (Composition API); names, getters and
  actions are unchanged
* `NUXT_PUBLIC_API_BASE_URL` and a `useApi()` fetch client in both apps, so a connected backend
  needs no hard-coded address; footer social links moved to `data/site-contact.ts`
* Fixes: password-strength colour did not change in Turkish; dialogs now focus their close
  button in every language (`data-dialog-close`); expiry and CVC labels are linked to their
  inputs; removed an unused component (`GoogleAddressPicker.vue`)

**Home layouts**
* Three alternative home pages next to the default marketplace home: **Fashion**
  (`/home/fashion`), **Deals** (`/home/deals`) and **Single product** (`/home/single-product`).
  They reuse the existing blocks and the real catalog, translate into Turkish and follow dark
  mode. The footer has a "Home layouts" row to switch between them in the demo; the
  documentation explains how to make one the default
* The home page's deal rail moved into `useHomeRails()` so the layouts share it
* Fixes found while testing them: the rating summary on the product page is no longer an
  `<aside>` inside `<main>` (accessibility checker), and the "hrs / min / sec" labels of the
  Deal of the day countdown are no longer dimmed

**Campaign design**
* Coupons are ticket cards with a coloured stub, perforated edge and expiry badge (also fixes the
  squeezed layout on phones)
* Mega Deals is a dark card with segmented countdown tiles, red percentage badges and a
  "claimed" bar per product; the three home cards beside it (reasons, Mega Deals, In Focus) now
  share the same height
* The deals page opens like a large store's deal page: "Limited time" badge with a live
  countdown, price shortcuts (Under $50 … $250 and above) and a "Deal of the day" card
* The hero headline gets a "Shop now" button and stays readable in dark mode; the thin promo
  strip is hidden on phones, where it was too small to read
* Phones: the hero's side banners (women's / men's fashion) show as two cards under the hero,
  and the service promises are one slim rotating line instead of a block of check marks

**Help and footer pages**
* The help chat no longer floats over every page. It opens from "Ask Nova" in the header, the
  mobile menu, the help centre and the footer ("Chat with us"); on phones it is a bottom sheet
* Help, shipping, returns, contact, track order, about, careers, warranty, sell, corporate quote
  and the legal pages get a section tab bar (a side list on large screens), so related pages are
  one tap apart
* Product names, descriptions and option values (colour, size…) are translated in Turkish
* The back-to-top button sits above the mobile bottom bar and is hidden on cart, checkout and
  product pages
* Hosted demo only: bright campaign banners and full product galleries (close-up and studio
  shots) built from the preview photos (`tools/make_demo_banners.py`, `tools/make_demo_gallery.py`)

**Admin**
* Images are served as they are (`image.provider: 'none'`). In a static export the default
  provider pointed product, brand, banner and review images at `/_ipx/` files that did not exist

**Fixes before release**
* "Shop by department" cards share one frame: a 2 × 2 grid of equal squares (three products with
  different photos, named and priced, plus a "See all" tile); one compact row on phones, which
  cuts the home page by about 4,700px on a phone
* Order numbers are short (`4F223991`) everywhere, and **Track an order** accepts the short number
  (the full id and a leading `#` still work). It used to ask for the 36-character id
* Cart and checkout: the sale-price saving is shown as "You save $30.00 on sale prices" under the
  total instead of a "Discount" row that was not part of the sum, so the rows add up
* The order page labels the tax line "Tax (included)"
* One return window everywhere: 14 days (home page, header, banners and sample contracts said 30
  while the policy pages said 14)
* One support phone number and e-mail for the footer and the Contact page (`data/site-contact.ts`)
* Admin: coupon and discount percentages are limited to 1–100 %; emptying an optional field
  (minimum cart, usage limit, start/end date) now removes it instead of keeping the old value
* The compare bar no longer covers the "Add to Cart" / "Checkout" bar on phones
* Dependencies: Vue 3.5.44 and sharp 0.35.5 (security fixes, pinned with `overrides`), newer Vite
  and svgo. `npm audit --omit=dev` is clean for the admin; for the storefront it lists only
  node-forge, reached through `@nuxt/image → ipx → listhen` (the development server's HTTPS
  helper; node-forge has no patched release yet). The rest of `npm audit` is development tooling
  (Nuxt devtools, Vitest, browser data)
* Tests: 100 storefront tests (new: order numbers, guest order lookup)
* "Shop by department": every card shows four products (a department with fewer is topped up from
  its neighbours); "See all" sits next to the department name
* Seller portal: a seller sees only their own lines and amounts of an order that several sellers
  share, never the customer's card, and cannot change, refund, cancel or fulfil such an order
  ("Shared order" badge, read-only); the same rules are in the data functions, not just the screens
* Admin: the select menu works with the keyboard (arrow keys, Home/End, type-ahead, Enter,
  Escape) and no longer closes when its own list scrolls; the phone menu drawer is closed to
  Tab while hidden, locks page scroll, closes with Escape and reports `aria-expanded`
* Demo data fits the USD / US template: 8 % included sales tax, customers in the five delivery
  countries, US-style barcodes and support phone; warranty reads "2 years" on the product page
  and in the cart
* Header: the service row no longer runs into the promises on 1,000–1,400 px screens

**Quality pass (browser crawl, axe-core, html-validate, Lighthouse)**
* Fixed: a hydration warning on every account page and the wishlist (they are now rendered in
  the browser only — they are private and `noindex`)
* Fixed: on a static host, card hover images and the "no image" fallback returned 404 because the
  image optimiser only knows the sizes it saw while pre-rendering; SVG images now bypass it
* The admin shows order numbers in capitals (`4F223991`), the same as the shop, so a customer's
  number can be found by eye
* Valid markup: no `<h1>` inside a `<span>`, no heading or `<div>` inside a `<button>`, valid
  `autocomplete`, unique ids on the search page, `aria-label` only on elements that allow it
* Accessibility: named progress bars, switches, tree toggles and form fields; focusable scroll
  regions; bigger tap targets for the hero dots; descriptive "Read the cookie policy" links;
  contrast raised for small text (brand text, grey captions, status colours) — the brand orange
  itself is unchanged and is 3.06:1 with white text (see the troubleshooting table)
* Fixed: phone fields could not be submitted when the number started with `+` (the browser's
  digits-only pattern rejected it); they now accept a leading `+` and open the phone keypad
* Test that compiles every translation message (a stray `@`, `|` or brace breaks the production
  build) and checks that English and Turkish have the same keys
* The mobile menu drawer and the select menus of the admin work with the keyboard
* Meta descriptions on Search and Compare; the old `/corporate/become-a-seller` address
  redirects to `/sell`; the admin has a favicon link and a `<main>` landmark on the sign-in page
* Hero image is fetched with high priority; `nuxt` and `@nuxt/image` are pinned to the tested
  versions (see "Dependency versions" in the documentation); the unused `@nuxt/fonts` dependency
  was removed

**Admin: roles and seller applications**
* Staff roles are enforced in the demo: each person's role (Owner, Admin, Manager, Support,
  Analyst) decides which areas appear in the menu and open, which are read-only ("View only"
  notice) and which writes are refused; suspended people cannot sign in, an invited person is
  activated by the first sign-in. Demo accounts: `owner@`, `admin@`, `priya.nair@` (Manager),
  `marcus.webb@` (Support), `sofia.reyes@` (Analyst), `seller@` — all with `demo1234`
* Seller applications: after applying on `/sell`, an "Open it in the admin queue" button carries
  the application to the admin's Sellers page; approving it lets the applicant sign in as the
  owner of a new (empty) store
* Documentation: a "Payments and e-mail" guide (Stripe PaymentIntent + webhook, a `sendMail()`
  helper) for replacing the two simulations

**Code clarity**
* Documentation: new "How data flows" (pages → page logic → Pinia stores → data layer → mock
  database) and "Where do I change…?" table in section 3, also in the HTML help file
* `useLiveCatalog` / `useLiveCategories` are now `useCatalog` / `useCategories`;
  `utils/productAttributes.ts` → `product-attributes.ts`, admin `utils/sortRows.ts` → `sort-rows.ts`
* Source comments rewritten for buyers: no notes left over from development, no references to a
  specific backend; each data-layer file says what to replace when you connect your API
* Admin error banners say "Data could not be loaded" (was "Live data…"); the Settings preview
  note no longer promises an API
* Fixed: the Mega Deals countdown no longer causes a hydration warning on the home page

**Both apps**
* Source comments no longer name other online stores
* Unit tests for the demo payment rules, guest-order ownership and stored orders

## 1.0.1 — 2026-10-15

* HTML help file (`documentation.html`)
* Flat HTML export so Cloudflare Pages does not redirect-loop category URLs
* Dropped `/* /200.html 200` on Cloudflare (that sent every page to `/200`)
* `THIRD_PARTY_LICENSES.md` and `OFL.txt` included in the package
* Node `engines` field (`>=20.19.0`); browser support listed in the documentation
* Preview set includes a mobile home and a mobile product page

## 1.0.0 — 2026-10-06

First public release of Nova: a storefront and an admin dashboard, both on demo data.

**Storefront**
* Catalog, search, compare, wishlist, cart, multi-step checkout, account area, help and
  legal pages, cookie consent
* Forgot-password page (demo — simulates sending the reset e-mail)
* Home page with varied sections instead of repeated product rails: mega deals, featured
  collections mosaic, numbered Top 10 rail, promo cards, tabbed trending products, a
  "Deal of the day" with countdown, "Deals by budget" tiles, a "Coupons & offers" strip with
  copy-to-clipboard codes, two editorial blocks, category cards, customer reviews and a photo
  gallery
* Demo artwork: every image is original SVG art (gradient plus line icon, no text on the
  images); `tools/gen_placeholders.py --labels` prints slot names and sizes when you need a shot list
* Sign-in page with neutral alternative-method buttons (e-mail link, passkey, SMS code) —
  no third-party brand logos
* Legal pages: generic sample text with a visible "Sample text" notice
* `NUXT_ALLOW_EMBEDDING=true` lets a demo build run inside an iframe (live previews); off by default
* "All deals" now covers every discounted product (up to 60) instead of the first 20
* A category link with filters (for example the "Deals by budget" tiles) keeps its filters when
  it is reloaded or shared on a statically hosted demo
* Light / dark / system theme with no flash on load
* Six languages: English, German, Spanish, French, Italian, Japanese
* Offline state/region lists for the five delivery countries (US, CA, GB, DE, AU) —
  no third-party geo dataset

**Both apps**
* The theme choice is also kept in `localStorage`, so it survives a reload where cookies are refused
  (a demo shown in a third-party iframe, notably in Safari)
* `npm ci` now works with npm 10 (the version bundled with Node 22) as well as npm 11; the admin's
  lock file was out of sync for npm 10. Both Dockerfiles were built and run successfully.

**Admin**
* Dashboard, analytics, orders, payments, returns, shipping, products, categories, brands,
  customers, reviews, support, campaigns, coupons, discounts, banners, agreements, settings
* Sign-in also works on static hosts that serve pages with a trailing slash (`/login/`)
* Permissions page: team table (invite, change role, suspend/reactivate, remove) and a
  role-by-area access matrix
* Activity Log page: filterable audit trail
* Light / dark / system theme, six languages

**Known limits** — see "Things the demo deliberately does not do" in `DOCUMENTATION.md`.
