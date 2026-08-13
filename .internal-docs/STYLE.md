# Docs style decisions

Short, binding decisions so future edits stay consistent. (Internal — this
directory is `.mintignore`d.)

## Heading case: per-surface convention (deliberate)

- **Reference surfaces** — core/, advanced/, compliance/, api-reference/,
  tutorials/, top-level pages: **Title Case** headings and page titles.
- **Task surfaces** — dashboard/, setup/ (Guides tab), sdk-guides/:
  **sentence case**, task-first ("Add a customer", "Manage invoices").

Rationale: the two tabs serve different modes (looking things up vs doing a
task); the case split tracks the surface, so it reads as intentional. Do
not mix cases within one directory.

## Other standing rules

- SDK tab labels: exactly `cURL`, `Node`, `Python`, `Go` (never
  "TypeScript" — the fence language stays `typescript`).
- Spelling: `canceled` everywhere EXCEPT literal wire values that really
  are `cancelled` (cancellation response `cancelled_at`, cancel-flow
  session status, GSTN `Cancelled`).
- Amounts in examples: minor units, realistic (₹ INR with GST, or USD);
  IDs are UUIDs, never Stripe-style prefixes.
- Every fact must be verifiable in recur-so source or the OpenAPI spec —
  never document a field, event, method, or env var you haven't seen
  defined. When the spec and handler disagree, the handler wins; fix the
  spec upstream.
- Every page ends with an onward link (Next steps cards or inline).
- Every code fence carries a language tag (`text` for ASCII tables).
- Every rename ships a redirect in docs.json in the same PR.
- Flow-style YAML descriptions containing commas must be quoted.
- Mintlify gotchas: `fonts` accepts heading/body only; `mint broken-links`
  parses every md/mdx including dot-dirs (this dir is excluded via
  `.mintignore`); read the Mintlify check-run OUTPUT on failures — the CI
  validate job passing does not mean the deploy passed.
