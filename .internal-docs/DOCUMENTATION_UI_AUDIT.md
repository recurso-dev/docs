# Documentation UI/Content Audit — docs.recurso.dev

Date: 2026-08-12 (late night) · Phase 0 of `DOCUMENTATION_REDESIGN.md` ·
Method: six parallel evidence sweeps (IA/inventory · API reference vs
`recur-so/cmd/api/openapi.yaml` · developer journeys · accounting/tax truth vs
`ledger.go` · links/SEO/search/AI · visual/style/templates), every claim
carrying file:line evidence, verified against the product code — not the
docs' own claims.

**Verdict: the docs are structurally healthy but truth-broken.** Frontmatter,
components, and per-page craft are far above average (0 missing alt texts,
96% tagged fences, consistent Frames, real per-page templates already in
use). The two existential problems are (1) **fabrication**: the flagship
walkthrough, the ledger API pages, the webhook event catalog, and every
worked journal entry contain invented parameters, events, accounts, or
postings that the product does not have — in *financial* documentation; and
(2) **an IA organized by content-type instead of intent**, so the same task
is documented 3–4× across tabs with no canonical page. This is the same
diagnosis as the website audit: a precision-and-truth problem first, a
redesign problem second.

---

## 1. Current information architecture

6 tabs, 326 nav pages, 289 MDX files on disk (175 nav pages = API Reference).

- **Documentation** — 12 groups / 68 pages: Getting Started (7), Migrating (3),
  Tutorials (4), Core Billing (7), Monetization (10), Revenue & Finance (5),
  Growth & Recovery (7), Integrations (6), Platform (7), Customer Portal (4),
  Deploy & Operate (4), Help & Reference (4).
- **Setup Guides** — 7 groups / 28 pages. **Dashboard Guide** — 7 groups / 37
  pages (incl. 8 workflows). **SDKs** — 3 groups / 9 pages. **Compliance** — 3
  groups / 9 pages (India 7, EU 1, Global 1). **API Reference** — 8 groups /
  175 pages, nested 3 deep (docs.json:389-795).
- The split **Documentation vs Setup Guides vs Compliance vs Dashboard is by
  content-type, not user intent**: dunning is documented 4× (setup/dunning,
  advanced/dunning-campaigns, compliance/dunning, dashboard/dunning),
  cancel-flows 4×, India GST 4×, US tax 4×, webhooks 3×, wallets/organizations/
  revenue-recognition/accounting/payment-gateways/customer-portal 3× each —
  with no cross-linking contract saying which is canonical.
- `advanced/` (35 pages) is the real product documentation — the deepest
  content — but is named and positioned as an afterthought; the target
  CORE CONCEPTS / GUIDES split does not exist.

## 2. Current navigation

- Depth is fine (2 levels; API Reference 3). Documentation's 12 sidebar groups
  is a wall; "Platform" appears as a group name in 3 tabs, "Getting Started"
  in 3; icon "server" reused for two different groups (docs.json:124,144).
- Global anchors "Documentation" and "API Reference" (docs.json:803-810)
  duplicate the tabs.
- 0 nav entries point to missing files; no duplicate page entries. Theme
  "maple", OpenAPI playground configured (docs.json:930-941).
- **docs.json has NO `redirects`, `seo`, or `search` keys** — the site cannot
  be restructured safely (every rename 404s), and past restructures (the
  45-endpoint api-reference resync) left no redirect trail.

## 3. Developer journeys (charter's 12)

| Journey | State |
|---|---|
| Understand Recurso | GOOD — index → concepts → core/overview (nit: concepts.mdx:279-280 lists `subscription.created` twice) |
| Run locally | GOOD start, frayed: `make demo` verified real (ports/key/Mailhog match Makefile); but `make migrate` **does not exist** (migrations auto-run at boot, cmd/api/main.go:137-145), `API_KEY_SECRET` env var is fabricated, "Create an API Key" is a chicken-and-egg (dashboard login requires a key) |
| First customer | BROKEN example — end-to-end.mdx:88, us-quickstart.mdx:71, concepts.mdx:151 use `"address": {...}`; API accepts only `billing_address`/flat fields (handler/customer.go:79) — **silently ignored**, so us-quickstart's state/zip "driving the tax lookup" never land |
| Product/plan | GOOD — matches CreatePlanRequest exactly |
| Create subscription | SPLIT-BRAIN — quickstart is real; end-to-end.mdx:127, core/subscriptions.mdx:23,29, api-reference/subscriptions/create.mdx:16,45, troubleshooting.mdx:183 teach a **fabricated `payment_gateway` param + `checkout_url` response** (neither exists in Go code or the docs' own openapi.yaml) |
| Usage-based billing | STRONGEST — quickstart → advanced/usage-billing → tutorials/usage-based-saas; shapes match spec |
| Generate invoice | DEAD END — nothing after the subscription step says how to see the first invoice; the real fast paths (`/v1/subscriptions/{id}/advance`, hosted checkout) live only in core/payments.mdx, unreferenced from getting-started |
| Collect payment | TWO STORIES — core/payments.mdx is accurate (hosted `/checkout/{invoice_id}`, gateway by currency, matches checkout.go); end-to-end.mdx tells the fake `checkout_url` story |
| Understand the ledger | Great destination (advanced/ledger.mdx), no road — quickstart Next Steps omits it |
| Webhooks | WRONG HEADER everywhere — code sends `X-Recurso-Signature` (webhook_worker.go:127); advanced/webhooks.mdx:62, troubleshooting.mdx:131, going-to-production, both migration guides say `recurso-signature`. quickstart.mdx:290-293 verifies over `JSON.stringify(req.body)` — the exact anti-pattern troubleshooting.mdx:120-140 warns against |
| Deploy | MISSING MIDDLE — going-to-production covers keys/nginx/monitoring, but `docker-compose.prod.yml`, `k8s/`, `render.yaml`, `railway.json` exist in recur-so and are never referenced |
| Troubleshoot | GOOD — symptom→cause→fix with real error strings (one drift: the payment_gateway accordion) |

**The quickstart ends at webhooks, never at the ledger** — the differentiator
("books that provably balance") is never observed. end-to-end.mdx also has an
internal amount inconsistency: plan `499900` (Step 1) vs invoice
`subtotal: 4999 / total: 5899` (Steps 4-5).

## 4. Content inventory

289 MDX on disk; 326 nav entries; all resolve. 17 pages missing
`description` (16 api-reference stubs: billing/set-charges, simulate-charges,
`gateways/{connect,disconnect,list}`, `metrics/{create,delete,get,list,update}`,
`subscriptions/{bill-usage,usage-amount}`, `usage-alerts/{create,delete,list}` +
snippets/snippet-intro.mdx which has no frontmatter). Title casing splits by
tab: core/setup/compliance/advanced 100% Title Case; dashboard mixed 21
sentence / 15 Title; api-reference 171 Title + 5 stragglers. Images: 71 files
(52 dashboard, 15 setup); hero-light/dark.png and screenshot-placeholder.svg
referenced by nothing.

## 5. API reference inventory

- Hybrid build: `api.openapi` → vendored `api-reference/openapi.yaml`
  (**byte-identical to recur-so/cmd/api/openapi.yaml** — the v0.12.0 resync is
  verified for the spec) + 170 hand-written MDX pages with `openapi:`
  frontmatter pointers (playground + prose). Zero bogus pointers.
- **136 of 306 operations (44%) have no page.** Missing families: auth (21),
  settings (13), import (12), analytics (10), portal/subscriptions/wallets (6
  each), webhooks/entities/users (5 each). Painful gaps:
  `GET/PUT /v1/customers/{id}`, `GET /v1/subscriptions/{id}` (+addons,
  preview-change, commitment), the **entire wallets CRUD** (only close.mdx
  exists), `POST /v1/usage/events/batch`, `POST /v1/invoices/{id}/send`,
  `GET/PUT /v1/plans/{id}`, credit-note approve/reject/void/pdf.
- Sampled pages are genuinely good (minor-unit amounts, tenant-scoping/404
  semantics, ledger side-effects documented) — but only 2/176 pages mention
  `Idempotency-Key`, ~20 mention any error, none states auth explicitly,
  pagination is rarely stated per-page.
- authentication.mdx is accurate (Bearer, `rsk_test_`/`rsk_live_`,
  key-mode vs gateway_mode, 401 key_mode_mismatch; matches spec
  securitySchemes).
- The spec itself has only **9 `example:` values across 306 operations** —
  playground-generated samples are schema junk.

## 6. Missing documentation

The 136 undocumented operations (§5); a Deploy page (compose-prod/k8s/render/
railway); a "first invoice" bridge after the subscription step; a ledger
observation step in the quickstart; `/compliance/data-residency` (linked
twice, never written); an audit-log API page (linked from okta-sso); explicit
Prerequisites sections (exactly 1 page repo-wide has one —
advanced/payment-gateways.mdx:46); llms.txt (none in repo, no gen script —
relies on Mintlify plan auto-serving, unverified).

## 7. Duplicate documentation

Quickstarts ×4 (quickstart, us-quickstart, end-to-end, dashboard/workflows/
first-subscription); sdk.mdx (1,494 words) is a full parallel of the SDKs
tab; the 3-4× per-task tab duplication catalog in §1; literal byte-dupe
`api-reference/ledger/export 2.mdx` (iCloud artifact — unrouted but Mintlify
publishes orphans; delete it); 6 duplicate page titles ("Quotes", "Customers",
"Invoices", "Subscriptions" collide core|advanced vs dashboard; "Cancel
Subscription" collides across two API groups).

## 8. Inconsistent terminology

- **53 "cancelled"** vs the API enum `canceled` — including a heading
  (core/subscriptions.mdx:195) and event-table rows where enum and gloss
  disagree in the same row (concepts.mdx:280, advanced/webhooks.mdx:124,
  api-reference/webhooks/events.mdx).
- "pricing plan" ×3 vs standard "plan". Otherwise clean (no customer/client
  drift — "client" hits are SDK variables).
- Two heading-case regimes by tab (Title Case vs sentence case); SDK tab
  labels split "TypeScript" vs "Node" for the same SDK
  (core/subscriptions.mdx:19 vs tutorials/usage-based-saas.mdx:46).

## 9. Inconsistent examples

- **ID style**: Stripe-style `cust_abc123`/`sub_def456`/`wal_8kQ2` in 43+
  places vs the product's raw UUIDs (47 UUID mentions coexist) — same defect
  class the website audit caught.
- Stale example dates: `new Date('2024-02-15')` etc. — 9 hits in
  core+compliance in a 2026 product.
- `api.recurso.io` host in api-reference/ledger/entries.mdx:23.
- end-to-end.mdx amount inconsistency (§3).
- Multi-language coverage thin: 402 bash vs 45 python / 37 go fences; core/*
  CodeGroups are cURL+TS only; portal/* (36 fences) and us-quickstart (10)
  have zero CodeGroups.

## 10. Broken links

3 broken internal links: advanced/automation.mdx:34 + advanced/
integrations.mdx:41 → `/compliance/data-residency` (never written);
advanced/okta-sso.mdx:52 → `/api-reference/account/list-audit-logs` (no such
page). 1 stale anchor: advanced/integrations.mdx:12 →
`#gocardless--adyen-experimental` (heading renamed, slug is
`#gocardless--adyen`). 0 broken image refs. Rot risks: evidence.mdx pins
`v0.9.0` as "current" (product is v0.12.0) and hardlinks ~10
`blob/main/internal/service/*.go` paths; faq.mdx:164 → ROADMAP.md and
going-to-production.mdx:239 → docs/incident-runbook.md need existence checks.

## 11. Weak pages

1. **end-to-end.mdx** — flagship walkthrough built on the fabricated
   subscription/checkout flow + inconsistent amounts.
2. **core/subscriptions.mdx** — `payment_gateway`/`checkout_url` in every
   example.
3. **advanced/webhooks.mdx** — canonical verification snippet uses the wrong
   header; documents `recurso.webhooks.constructEvent`, an SDK helper **no
   SDK ships** (sdk-guides/workflows/webhooks.mdx:322 says so — internal
   contradiction).
4. **us-quickstart.mdx** — `address` silently dropped defeats the page's
   premise; also uses `payment_gateway`.
5. **quickstart.mdx** (Config/API-key sections) — fabricated env var,
   nonexistent `make migrate`, login chicken-and-egg, stops short of the
   ledger.

## 12. Strong pages

1. **troubleshooting.mdx** — symptom→cause→fix, real error payloads, honest
   about optional TigerBeetle.
2. **advanced/ledger.mdx** — worked postings per event, account codes,
   reconciliation semantics (its *content* needs the §truth fixes, but its
   shape is the model for concept pages).
3. **core/payments.mdx** — matches the real checkout implementation, honest
   constraints, accurate sequence diagram.
Also: authentication.mdx, api-reference/errors.mdx (canonical error envelope
documented once, well).

## 13. Search / discoverability problems

No `search` key; **0/328 pages have `keywords`** — zero synonym support.
Demonstrated gap: "metered billing" matches NO page title; "metering" only
dashboard/metering-usage.mdx; the canonical advanced/usage-billing.mdx
("Usage-Based Billing") is unreachable by that query. Three near-duplicate
titles compete for the same query (tutorials/usage-based-saas,
sdk-guides/workflows/usage-based-billing, dashboard/workflows/
usage-based-product). 6 duplicate titles (§7) make results ambiguous.

## 14. AI-agent discoverability problems

**137/328 pages open with a literal "## Overview"** — an agent chunking by
heading gets 137 indistinguishable sections. Prerequisites are explicit on
exactly 1 page. No llms.txt in-repo. 40 untagged fences (advanced/ledger.mdx
×6, revenue-recognition ×6, payment-gateways ×4, smart-retry ×4 — mostly
ASCII ledger tables that lose the copy button). 26 of 152 non-API pages end
with no onward link. Positives: contextual AI menu configured
(copy/claude/chatgpt/cursor), self-served /openapi.json documented, Postman
collection.

## 15. Visual inconsistencies

Theme is stock-Mintlify green: primary #10B981/#34D399/#059669 (Tailwind
emerald defaults, docs.json:5-9), **no `fonts` key** — no brand serif, no
JetBrains Mono for numerals; logo/favicon ARE on-brand (ink #0B1F1A). Mixed
png/jpg screenshot formats. Card-wall hub pages (dashboard/index.mdx 36
Cards, setup/index.mdx 27, index.mdx 14). Components otherwise rich and
consistent: Card 385, Step 355, CodeGroup 136, Frame 75 across 328 files;
snippets/ dir effectively empty (starter file only).

## 16. Accessibility problems

Near-clean: 0/75 images missing alt; no layout tables; callouts moderate
(~1.5/page, max 8). Remaining: 40 untagged fences (screen-reader + copy
affordance), heading-case drift, and the 137 "Overview" headings (also an
a11y/scan problem). Mintlify handles keyboard/contrast at the framework
level; verify post-theme-change.

## 17. SEO problems

No `seo`/metatags/og-image/canonical config (pure defaults); 16 missing
descriptions; 6 duplicate titles; no redirects key (renames 404). Titles
otherwise 327/328 present.

## 18. TRUTH DEFECTS (fix before anything else — this is financial documentation)

Verified against recur-so code, in reader-damage order:

1. **Fabricated ledger API surface** — `advanced/ledger.mdx:140-260` +
   `api-reference/ledger/entries.mdx:12-66`: `start_date`/`end_date` filters
   (actual: `account_id` required, `code`, paging — handler/ledger.go:32-52),
   `lacc_`/`ltxn_` ID prefixes, an `entries[]` sub-array with per-entry
   `currency`, `reference_type`, `total_count`, `"object":"list"` — the real
   response is flat UUID rows `{debit_account_id, credit_account_id, amount,
   code, reference_id}`. Same fabrication class as docs PR #37. Integrations
   fail on first call.
2. **Fabricated webhook events + broken verification** — the code ships
   exactly **10 event types** (domain/webhook.go:67-94); docs invent
   `ledger.entry.created`, `accounting.*` ×4, `credit_note.*` ×4,
   `subscription.activated/updated/paused/resumed/past_due`,
   `payment.refunded/disputed`, `invoice.finalized/past_due`,
   `customer.deleted` (advanced/webhooks.mdx:120-152, api-reference/webhooks/
   events.mdx:40-95 — which also omits the real `subscription.renewed`).
   Header is `X-Recurso-Signature` + `X-Recurso-Event-ID`, hex HMAC-SHA256
   (webhook_worker.go:125-128,193) — docs say `recurso-signature` and ship a
   `constructEvent` SDK helper that doesn't exist.
3. **Journal entries the transfer-based ledger cannot produce** — every row
   is ONE debit + ONE credit, yet docs show: payment as "DR 1000 Cash / CR
   2100 Deferred" (collapses Code-1 + Code-3, erasing AR —
   revenue-recognition.mdx:77-78,105-106; advanced-billing.mdx:236-237); tax
   reclass as "DR 1100 AR / CR 2200" (Code-6 debits Deferred/Revenue, never
   AR — ledger.go:377-383; double-counts AR — ledger.mdx:118-119); a
   "Discount Applied DR 5100/CR 1100" posting that **doesn't exist**
   (discounts only reduce Code-1 gross — ledger.mdx:127-128,
   revenue-recognition.mdx:250-251); an invented cancellation-refund posting
   (actual = Code-4 5000→1000 + Code-5 2100→5000 —
   revenue-recognition.mdx:220-221); a classic 3-leg compound FX entry the
   model can't represent (multi-currency.mdx:276-281).
4. **Recognition credited to 4000 in every example** — Code-2 credits **4100
   Recognized Revenue** (ledger.go:1345, domain/ledger.go:128); ledger.mdx:
   100-101, finance-overview.mdx:23, revrec mermaid:19. The docs' trial
   balance never ties to the product's. ledger.mdx:46-54 chart omits 1200 TDS
   Receivable, 2300 Customer Credit, 4100; 5100's real name is "Credits &
   Adjustments" (not "Discounts"); account 4500 "FX Gain" doesn't exist.
5. **Fabricated capability** — multi-currency.mdx:186-200 claims per-currency
   ledger balances/dual-currency entries; the ledger stores no
   per-transaction currency (domain/ledger.go:116-120) — contradicting the
   page's own honest note at :178. accounting.mdx:16 claims QuickBooks "full
   two-way sync"; the sync is push-only (its own table at :50 agrees).
6. **Fabricated getting-started plumbing** — `payment_gateway`+`checkout_url`
   (§3), `address` (§3), `make migrate`, `API_KEY_SECRET`,
   `LEDGER_BACKEND`/`TIGERBEETLE_ADDRESSES`/`TIGERBEETLE_CLUSTER_ID`
   (actual: `TIGERBEETLE_ADDRESS` — cmd/api/main.go:217).
7. **Idempotency header split** — middleware reads `Idempotency-Key` on all
   mutating methods (middleware/idempotency.go:74); testing.mdx:165,172 and
   sdk.mdx:320 say `X-Idempotency-Key` (silent no-op) — **and the product
   spec's own info description carries the same bug**
   (cmd/api/openapi.yaml:14): file upstream. introduction.mdx:170 also
   under-scopes it to "POST requests".
8. **introduction.mdx:76-104 fabricates the response envelope** — bare object
   with `"object": "customer"` and `{"object":"list","has_more",
   "total_count"}`; the product returns `{data: ...}` and the docs' own
   endpoint pages show it correctly. Pagination table says default 10/max
   100; reality is default 50/cap up to 1000 (recur-so conventions).
9. **Stale**: revrec report params/fields (real: `month`/`year`,
   `recognized_amount`/`deferred_balance` — handler/revrec.go:45-65;
   report.mdx:30 names `deferred_amount`); reconciliation table lists 8
   discrepancy types, code has ~20; evidence.mdx pins v0.9.0.
   Honest & verified-true: Peppol/US-tax/GST filing boundaries, 8
   aggregations, 7 charge models, retry policy, GL-export columns.

## 19. Highest-leverage improvements & migration order

**Wave 1 — Truth pass (before any restructure; small PRs, verify each against
code):** §18.1-18.9 + the 3 broken links/1 anchor + delete `export 2.mdx` +
"cancelled"→"canceled" sweep. Also file the upstream openapi.yaml
idempotency-header bug and add real `example:` values to the spec while
there.

**Wave 2 — Safety rails:** add `redirects` support to docs.json workflow
(policy: every rename ships a redirect); add `keywords` to ~15 top pages
(usage-billing: "metered billing, metering"); de-dupe the 6 title collisions;
descriptions for the 16 API stubs.

**Wave 3 — IA (charter Phase 1):** collapse to intent-based structure —
GET STARTED (one quickstart ending at the ledger; us-quickstart becomes a
tax guide) / CORE CONCEPTS (promote advanced/) / GUIDES (fold setup/* task
recipes; task-titled) / API REFERENCE / DEVELOPMENT / DEPLOYMENT (new deploy
page) / REFERENCE — with redirects for every moved URL. Kill sdk.mdx in
favor of the SDKs tab.

**Wave 4 — Journeys + quickstart (charter Phases 2-3):** rewrite end-to-end
around the real hosted-checkout flow; quickstart final step = "watch the
books balance"; bridge subscription→invoice; Prerequisites sections on all
guides.

**Wave 5 — API reference completion:** the 136 missing operations (start:
wallets CRUD, get-by-id family, usage batch, credit-note lifecycle);
idempotency + auth + pagination stated per page; UUID examples everywhere.

**Wave 6 — Accounting docs (charter Phase 10):** rewrite ledger/revrec
examples in the true transfer-based form with the full 10-account chart —
reuse the corrected postings from the website's AccountingEntry work.

**Wave 7 — Language + templates:** Python/Go tabs in core CodeGroups; one
casing regime; "## Overview" → topic-bearing headings; next-step links on the
26 dead-end pages; tag the 40 fences.

**Wave 8 — Visual system (LAST, charter Phase 17):** brand palette + fonts
block (mono numerals minimum); re-shoot all 67 dashboard/setup screenshots
against v0.12.0 (the shotrig from the website mission does this); thin the
card-wall hubs; delete dead hero images.

**Wave 9 — QA:** Mintlify build + link check in CI (note: `mint` CLI requires
Node LTS — this machine's Node 26 refuses; pin via nvm/volta or CI), llms.txt
verification, keyboard/mobile pass, design-director scoring.
