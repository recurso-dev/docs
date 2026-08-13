# Documentation Redesign Charter (founder directive, 2026-08-12)

Transform docs.recurso.dev (Mintlify) from "a collection of technical pages"
into **"the definitive operating manual for Recurso."** A developer goes from
"never used Recurso" to "working billing integration" as fast as possible.

Quality bar: Stripe / Vercel / Linear / GitHub / Cloudflare docs — **bar, not
visual identity; the docs must feel like Recurso.** NOT greenfield: significant
docs exist; do NOT rewrite blindly; do NOT start with colors/CSS.

## Most important principle

Documentation is a PRODUCT; every page is an interface answering: what is
this / why does it matter / when to use / how to use / what does it return /
what can go wrong / what next. Write around **developer tasks**, not feature
existence. Docs must be: precise, technically authoritative, scannable,
executable, consistent, searchable, AI-agent friendly, API-first,
example-driven, honest, maintainable.

## Hard rules

- Docs describe the REAL product — never invent APIs, params, behavior,
  examples, capabilities. Verify against the recur-so codebase.
- Never pseudo-code where executable code is possible; examples copy-pasteable
  with realistic values (never `"name": "string"`).
- Small verifiable batches; never claim completion without verification.
- Use Mintlify capabilities (nav/groups/tabs/OpenAPI/components/search/SEO/
  keywords) rather than fighting the framework; consult current Mintlify docs
  before custom behavior. Navigation = the IA; keep it deliberate and shallow.

## Phase 0 — AUDIT FIRST (no content/design changes)

Inspect: docs.json, all MDX, API reference + OpenAPI config, examples, images,
components, frontmatter, redirects, links, search, SEO, versioning, auth docs,
webhooks, SDKs, conceptual/troubleshooting/self-hosting docs. Read DESIGN.md,
ART_DIRECTION.md, ARCHITECTURE.md, DOCUMENTATION_RULES.md (product repo) +
actual codebase. Output `docs/DOCUMENTATION_UI_AUDIT.md` with 19 sections:
1 IA today · 2 navigation · 3 developer journeys · 4 content inventory ·
5 API-reference inventory · 6 missing docs · 7 duplicates · 8 inconsistent
terminology · 9 inconsistent examples · 10 broken links · 11 weak pages ·
12 strong pages · 13 search problems · 14 AI-discoverability problems ·
15 visual inconsistencies · 16 a11y problems · 17 SEO problems · 18 highest-
leverage improvements · 19 recommended migration order.

## Phases 1–25 (conceptual targets)

1. **IA**: organize by user intent, not internal ownership. Conceptual shape:
   GET STARTED (intro/what-is/architecture/quickstart/install/first flow) ·
   CORE CONCEPTS (customers/products/plans/subscriptions/usage/meters/pricing/
   invoices/payments/tax/ledger/revrec/reconciliation) · GUIDES (task-titled) ·
   API REFERENCE · DEVELOPMENT (SDKs/webhooks/idempotency/errors/pagination/
   rate limits/testing/local dev) · DEPLOYMENT (self-host/cloud/env/db/prod/
   security/backups) · REFERENCE (errors/events/statuses/data model/config).
   Use actual capabilities; no empty sections.
2. **Journeys** (≥12): understand / run locally / first customer / product-plan /
   subscription / usage-based billing / invoice / payment / ledger / webhooks /
   deploy / troubleshoot. Clear beginning-middle-end; no random jumps.
3. **Quickstart**: copy-paste install→configure→start→create resource→billing
   object→invoice→observe ledger. Real commands, real responses, verified.
4. **Conceptual model**: teach Customer→Product→Plan→Subscription→Usage→
   Invoice→Payment→Ledger explicitly + debit/credit/accounts/journal entries/
   posting/reconciliation BEFORE API details.
5. **API reference**: use Mintlify OpenAPI integration where appropriate; keep
   synced with actual API; every endpoint: method/path/purpose/auth/params/
   body/response/errors/examples/idempotency/pagination.
6. **Examples**: cURL + JS/TS + Go + Python (actual SDK surface only) forming
   an executable narrative (customer→…→ledger).
7. **Errors first-class**: status/code/message/cause/resolution per important
   operation — never bare "400 Bad Request".
8. **Webhooks journey**: event model/types/signing/verification/retries/
   idempotency/ordering/duplicates/failure handling; production-safe patterns.
9. **Idempotency**: why/which ops/how to supply key/retry/timeout/repeat
   semantics with concrete examples.
10. **Accounting docs** (differentiator — don't hide): double-entry, journal
    entries, posting, accounts, revenue, tax liability, AR, payments,
    reconciliation. Real example (₹118,000 invoice → DR AR 118,000 / CR
    Revenue 100,000 / CR Tax Payable 18,000) then WHY. NOTE: verify against
    the product's ACTUAL transfer-based posting semantics (Code-1 gross to
    Deferred for subscriptions + Code-6 tax reclass — see recur-so
    docs/ACCOUNTING_PRINCIPLES.md and ledger.go) — never document the
    simplified 3-leg form if the product posts differently.
11. **Usage-based billing**: meters/events/aggregation/tiers/pricing/
    commitments/wallets/entitlements; show event→aggregation→price→invoice
    line→accounting entry.
12. **Tax**: only actual capabilities (India GST/e-invoicing/UPI · EU VAT/
    EN 16931/Peppol · US sales tax/nexus); distinguish Recurso vs external
    providers; never imply unimplemented compliance.
13. **Self-hosting**: prerequisites→install→config→db→env→first start→health→
    production→backups→upgrades→troubleshooting.
14. **Terminology**: canonical glossary; one preferred name per concept,
    everywhere (nav/titles/API/examples/code).
15. **Style**: concise American English; direct; active; short paragraphs;
    aggressive headings; no marketing fluff; don't assume accounting
    knowledge. "Create a subscription" not "Creating Your Very First…".
16. **Templates**: CONCEPT (what/why/how/example/related/next) · GUIDE (goal/
    prereqs/steps/verification/common errors/next) · API (purpose/auth/
    request/response/params/errors/examples/related) · REFERENCE (definition/
    fields/behavior/constraints/examples/related).
17. **Visual system LAST**: shared Recurso brand with recurso.dev +
    app.recurso.dev but feels like documentation. Restrained; scannable; no
    gradients/hero bloat/marketing animation.
18. Mintlify-native implementation. 19. **Search**: titles/headings/keywords/
    synonyms ("metered billing"="usage billing"); no stuffing. 20. **AI/agent
    friendliness (CRITICAL)**: explicit terminology, unambiguous headings,
    canonical examples, explicit prerequisites/constraints/errors, related
    links, stable terms, copyable code; humans first, structurally precise.
21. **A11y** WCAG 2.2 AA + reduced motion. 22. **Link integrity**: no 404s/
    stale paths/bad anchors/orphans; redirects on renames. 23. **SEO** without
    sacrificing usability. 24. **Testing**: build, link checks, validation,
    verify examples against a LOCAL Recurso instance where possible, desktop+
    mobile+keyboard. 25. **Visual QA** across every content type.

## Execution order

1 Audit → 2 IA → 3 journeys → 4 terminology → 5 templates → 6 quickstart →
7 core concepts → 8 API reference → 9 guides → 10 accounting → 11 usage
billing → 12 webhooks/errors/idempotency → 13 self-hosting → 14 visual →
15 search → 16 AI discoverability → 17 a11y → 18 SEO → 19 links → 20 full QA.

## Definition of done

A new developer can discover Recurso, understand the architecture, run it,
create a customer, create billing, process usage, generate an invoice,
understand the resulting ledger, integrate webhooks, handle errors, and
deploy — without asking the team basic questions.

## Final design-director test

Find any answer in under 30s · copy the example · understand WHY it works · understand
failure behavior · AI agent retrieves unambiguously · one coherent product ·
trustworthy enough for financial infrastructure. Any NO → fix.

## Progress report format (every cycle)

COMPLETED (exact files) · CONTENT · INFORMATION ARCHITECTURE · API · DESIGN
SYSTEM · TESTS (exact commands+results) · LINKS (broken found/fixed) · AI
READINESS · REMAINING (exact) · NEXT (highest-leverage).

## Final principle

Not "beautiful" — FAST. PRECISE. EXECUTABLE. TRUSTWORTHY. DEVELOPER-FIRST.
The reader should feel: "these people have thought through every detail of
their billing infrastructure."
