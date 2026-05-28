# List Bug Audit

Audit date: 2026-05-28  
Scope: static site files in repo root. Only this file was edited.

Verification passes run:
- JS syntax check: `app.js` parses successfully.
- JSON-LD parse check: `index.html`, `about/index.html`, and `faq/index.html` are valid JSON-LD.
- Local reference check: local `href`/`src` targets exist.
- Rechecked SEO metadata, filter logic, accessibility states, and deploy workflow.

## 1. High - Production web root is still a Git checkout

Evidence: `.github/workflows/deploy.yml:13-16` marks `/home/el/web/spawnvault.respawnsociety.web.id/public_html` as a Git safe directory, changes into it, runs `git fetch origin`, then `git reset --hard origin/main`.

Impact: if the web server does not block dot directories, `/.git/` can expose source history, remote config, refs, and deployment details. This setup also deploys every repo file into the public web root.

Recommendation: keep the Git checkout outside `public_html`, then sync only public artifacts (`index.html`, `about/`, `faq/`, `robots.txt`, `sitemap.xml`, `favicon.*`, `assets/`, `app.js`, `styles.css`). Also block dotfiles/directories at the web server level.

## 2. Medium - Non-site files are likely publicly deployed

Evidence: root contains `.github/`, `DESIGN.md`, `side_income_EN.xlsx`, and `listbug.md`. The workflow resets the whole repo into `public_html`.

Impact: source-only files, spreadsheets, workflow details, and this audit report can be downloadable from the live domain. `robots.txt` allows crawling everything.

Recommendation: deploy from a clean output folder or deny non-public files server-side (`.git`, `.github`, `*.md`, `*.xlsx`, etc.).

## 3. Medium - Home social/structured image uses ICO while declaring 1200x630

Evidence: `index.html:40-42` declares `og:image` as `assets/spawnvault.ico` with width `1200` and height `630`. `index.html:49`, `index.html:63`, `index.html:67`, and `index.html:122` also use the ICO. Local inspection shows `assets/spawnvault.ico` contains icon sizes up to `256x256`; `assets/spawnvault.png` exists and is `1254x1254`.

Impact: social previews and structured image consumers can reject or crop the image incorrectly. `summary_large_image` previews generally expect a normal raster image URL, not an ICO, and the declared aspect ratio is false.

Recommendation: use a real PNG/JPG social image, ideally `1200x630`, and update OG, Twitter, and JSON-LD image references consistently.

## 4. Medium - Structured data advertises a search endpoint that does not exist

Evidence: `index.html:96-101` declares a `SearchAction` with `https://spawnvault.respawnsociety.web.id/?q={search_term_string}`. The live script only reads `lang` from the query string at `app.js:298`; no code handles `q`.

Impact: Google can see an internal site search capability that the page does not implement. Users or crawlers landing on `?q=...` get the same unfiltered page.

Recommendation: remove `SearchAction` or implement real query handling/search UI for `q`.

## 5. Medium - JSON-LD ItemList is inconsistent with visible cards

Evidence: `index.html:129` says `numberOfItems` is `22`, but the JSON-LD list has only 11 `ListItem` entries. It also lists Outlier, DataAnnotation, Scale AI, Prolific, and Freecash at `index.html:134-139`, while those are not visible card names on the homepage.

Impact: structured data can become misleading and inconsistent with page content, which may hurt SEO trust or eligibility for rich results.

Recommendation: generate ItemList from the same card data used by the page, or remove apps that are not actually rendered.

## 6. Medium - Public copy advertises platforms that are not listed as cards

Evidence: `about/index.html:133`, `faq/index.html:156`, `faq/index.html:164`, `faq/index.html:172`, and translations in `app.js:200`, `app.js:211`, `app.js:215`, `app.js:219`, `app.js:264`, `app.js:275`, `app.js:279`, and `app.js:283` mention Outlier, DataAnnotation, Scale AI, Prolific, and Freecash. Those platforms are not present as visible homepage cards.

Impact: users are told they can find platforms that are not actually available in the directory, which can reduce trust and make the referral hub feel incomplete.

Recommendation: either add those platforms as cards or remove them from FAQ/About/translation copy.

## 7. Medium - Payout filters still return wrong cards because `data-payout` is inconsistent

Evidence:
- Bling says `BTC / PayPal` at `index.html:575` and `index.html:743`, but both cards use `data-payout="crypto-pay"` at `index.html:571` and `index.html:741`, so PayPal filter misses it.
- Gimi says `Bank Transfer / USDT` at `index.html:617`, but uses `data-payout="paypal"` at `index.html:613`, so PayPal wrongly includes it and Crypto misses it.
- ShopBack says `Cashback (IDR)` at `index.html:667`, but the passive card uses `data-payout="paypal"` at `index.html:665`.
- Xworld says `Dana / GoPay / OVO / USDT` at `index.html:760`, but uses only `data-payout="idr"` at `index.html:759`, so Crypto misses it.
- Filter logic checks exact tokens from `data-payout` at `app.js:388-391`.

Impact: users filtering by PayPal, IDR, or Crypto see inaccurate results.

Recommendation: normalize `data-payout` tokens to include every real payout method, for example `data-payout="crypto-pay paypal"` or `data-payout="idr crypto-pay"`.

## 8. Medium - Reset filter icon is removed by the i18n system

Evidence: the reset button contains icon markup at `index.html:340-341`, but it also has `data-i18n="filter-all"`. `applyLang` replaces translated nodes via `el.innerHTML = val` at `app.js:306-308`.

Impact: on page load or language toggle, the reset icon span is deleted. CSS that targets `.filter-reset-icon` at `styles.css:496`, `styles.css:501`, and `styles.css:505` no longer applies.

Recommendation: translate only the text span, or make the translation value include the icon markup intentionally.

## 9. Medium - Clipboard copy can fail silently

Evidence: `app.js:344-345` calls `navigator.clipboard.writeText(code).then(...)` without feature detection or `.catch(...)`.

Impact: copy buttons fail with no user feedback when clipboard permission is denied, the page is not in a secure context, or Clipboard API is unavailable.

Recommendation: add a fallback and failure state, or disable copy buttons when Clipboard API is unavailable.

## 10. Medium - `localStorage` access is unguarded

Evidence: `app.js:145`, `app.js:157`, `app.js:298`, and `app.js:305` call `localStorage` directly. `about/index.html:66` and `faq/index.html:80` also call `localStorage.getItem` inline.

Impact: browsers can throw `SecurityError` when storage is blocked or unavailable. If that happens in `app.js`, later features such as language, menu, copy, and filters can stop running.

Recommendation: wrap storage reads/writes in safe helper functions with `try/catch`, and continue with defaults when storage is unavailable.

## 11. Medium - Third-party script is loaded dynamically without integrity pinning

Evidence: `app.js:29-30` creates a script tag and loads `https://unpkg.com/lenis@1.1.20/dist/lenis.min.js`. No `integrity` or `crossOrigin` is set.

Impact: if the CDN response is compromised or altered, arbitrary third-party JavaScript runs on the site.

Recommendation: self-host the script or set an SRI hash and `crossOrigin="anonymous"`; also consider a CSP.

## 12. Low - Canonical and hreflang conflict for client-only language variants

Evidence: `index.html:8-11`, `about/index.html:8-11`, and `faq/index.html:8-11` set canonical URLs without `?lang=...` while declaring `?lang=en` and `?lang=id` as hreflang alternates. `sitemap.xml:10-12`, `sitemap.xml:22-31`, `sitemap.xml:38-40`, and `sitemap.xml:47-49` also list query-string language variants.

Impact: crawlers can receive mixed signals: alternate language URLs are declared, but the canonical says the non-query URL is preferred. Bots that do not execute JS also see the same initial document before language replacement.

Recommendation: create real static language pages (`/en/`, `/id/`) with matching canonical/hreflang metadata, or avoid indexing query-string language variants.

## 13. Low - Default metadata mixes Indonesian copy with English language/locale

Evidence: `index.html:2`, `about/index.html:1`, and `faq/index.html:1` set `lang="en"`, while titles/descriptions contain Indonesian-focused copy at `index.html:6`, `index.html:16`, `about/index.html:16`, and `faq/index.html:6`. OG locale is also `en_US` at `index.html:44`, `about/index.html:30`, and `faq/index.html:30`.

Impact: screen readers, search engines, and social parsers can classify the page language incorrectly.

Recommendation: align initial `html lang`, Open Graph locale, title, description, and body copy for the default page.

## 14. Low - 2025 copy is stale in 2026

Evidence: `index.html:36`, `index.html:308`, `index.html:942`, `about/index.html:167`, `faq/index.html:219`, `app.js:194`, `app.js:258`, and `sitemap.xml:9`/other entries use 2026 dates while visible/footer copy still says `2025`.

Impact: snippets and on-page trust signals can look outdated.

Recommendation: update year-sensitive copy to 2026 or remove fixed years where possible. `foundingDate: 2025` can remain if that is the actual founding date.

## 15. Low - Filter and expandable controls do not fully expose selected/open state

Evidence: filter buttons at `index.html:352-374` only toggle CSS class `active`; Bling toggles at `index.html:578` and `index.html:746` only change text. There is no `aria-pressed` for filters and no `aria-expanded` for the Bling buttons. The burger has `aria-expanded` at `index.html:270`, but no `aria-controls`. The More menu Escape handler closes the panel at `app.js:19-23` but does not reset the button's `aria-expanded` to `false`.

Impact: assistive technology does not reliably know which filter is active or whether expandable panels are open.

Recommendation: update `aria-pressed` on filter buttons, `aria-expanded` on Bling toggles, add `aria-controls` for controlled menus/panels, and sync `aria-expanded` when closing with Escape.

## 16. Low - Filter count language does not refresh after changing language

Evidence: `updateFilterUI` formats the count based on `currentLang` at `app.js:351-368`, but `applyLang` at `app.js:303-323` does not call `updateFilterUI`.

Impact: if a user changes language after filtering, the count can remain in the previous language until another filter action runs.

Recommendation: call `updateFilterUI(currentFilter)` after changing language, or store the active filter and refresh the count inside `applyLang`.

## 17. Low - Filters leave empty sections visible

Evidence: `setFilter` only toggles `.hidden` on individual `.card` elements at `app.js:378-393`; it does not hide parent sections that have no visible cards.

Impact: after a payout/effort filter, users can scroll through section headings with no visible cards.

Recommendation: after filtering cards, hide empty sections or show an explicit empty-state message.

## 18. Low - Homepage has no main landmark or skip link

Evidence: `index.html:240` starts `<body>`, `index.html:243` defines `<nav>`, sections start at `index.html:381`, and footer starts at `index.html:927`, but homepage has no `<main>`, `role="main"`, or `.skip-link`. By contrast, `about/index.html:71`/`122` and `faq/index.html:85`/`136` include skip link and main landmark.

Impact: keyboard and screen-reader users have a harder time jumping directly to the homepage's main content.

Recommendation: add `<a href="#main" class="skip-link">...` and wrap hero, filter bar, and content sections in `<main id="main">`.

## 19. Low - Secondary CTA class is used but never styled

Evidence: `about/index.html:141` and `faq/index.html:194` use `class="btn btn-secondary"`. `styles.css:731-743` defines `.btn`, `.btn-primary`, `.btn-accent`, and `.btn-outline`, but no `.btn-secondary`.

Impact: secondary CTA buttons render with only base button styling and can look unfinished or inconsistent.

Recommendation: define `.btn-secondary` or replace those links with an existing button class.

