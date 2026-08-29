# David Chen (@cooldavidie)

**I build AI products end to end, solo** — from large-scale data pipelines and physics-level simulators to production SaaS with real users, billing, and multilingual RAG.

- **[DataAssistant AI](https://www.dataassistant-ai.com)** — AI sales rep for industrial distributors. Live multi-tenant SaaS: Claude + RAG grounded in the customer's own catalog, human-approved replies in 5 languages, Stripe billing. *(live product, private code)*
- **[CA-SIM](https://cooldavidie.github.io/ca-sim-demo/)** — event-level digital twin of AI data-center power & cooling, running entirely in the browser, plus an open event-dictionary spec ([EDL](https://github.com/cooldavidie/edl-dict)). *(public demo + open spec)*
- **TrueProductData** — a 720k-product industrial data platform with a provenance-first schema, served to AI agents over MCP. *(private infra)*

davidchen.cch@gmail.com · [dataassistant-ai.com](https://www.dataassistant-ai.com)

---

## DataAssistant AI — AI sales rep for industrial distributors

**Live:** [app.dataassistant-ai.com](https://app.dataassistant-ai.com) · [marketing site](https://www.dataassistant-ai.com) (EN/ES/FR/TH/zh-TW)

Plugs into a sales team's Gmail or Outlook inbox, scores leads, runs outbound campaigns, and drafts replies grounded in the company's own product / pricing / lead-time data — with a human approval queue in front of every send.

**AI engineering** (hand-built pipeline, no LangChain):

- **Hybrid multilingual retrieval** — pgvector semantic search + lexical/SKU matching (pg_bigm), fused with Reciprocal Rank Fusion; Vertex AI multilingual embeddings (chosen because word-boundary-based search fails for Thai/Chinese); locale-aware number normalization so "3,7 kW" survives tokenization.
- **A numeric grounding guard** — every figure in a generated reply must trace back to a retrieved source row, the customer's own email, or a legitimate derived total (unit price × asked quantity). Anything else is flagged as invented and held for human review instead of sent.
- **LLM ops** — Claude model tiering by task complexity, per-model rate limiters, Redis response cache, daily cost ceiling with tracked spend, versioned prompt registry with variants.
- **Regression-driven RAG** — checked-in eval harness (including adversarial/security cases) plus chaos tests: API down, malformed model output, hallucinated figures.

**Production engineering:**

- Multi-tenant auth (JWT, TOTP 2FA, Google + Microsoft OAuth, team permissions, audit logs), Stripe subscriptions across 5 tiers with quota enforcement at both API and UI layers.
- GCP end to end: 4 Cloud Run services, Cloud Build CI/CD, Secret Manager, Cloud SQL + pgvector, Celery workers with ~12 scheduled jobs, Sentry on both tiers.
- ~110k lines of Python + TypeScript · 300+ API endpoints · 71 DB tables · 30 migrations · 460 commits over 12 months.

*Code is private (commercial product) — but the full architecture is documented in **[dataassistant-ai-architecture](https://github.com/cooldavidie/dataassistant-ai-architecture)**, and I'm happy to do a live deep-dive of any subsystem.*

---

## CA-SIM — Compute Availability Simulator for AI data centers

**Try it:** [live demo](https://cooldavidie.github.io/ca-sim-demo/) · **Open spec:** [EDL event dictionary](https://github.com/cooldavidie/edl-dict) (CC-BY 4.0)

For any electrical or cooling event — a grid voltage dip, a source-transfer gap, a feeder fault, a 2 Hz GPU training power cycle — CA-SIM answers: **how much compute is lost, for how long, and what margin remains.** Event-level hybrid engine (dt = 0.5 ms) over SST / BESS / 800 VDC / UPS / CDU / GPU-rack architectures, running entirely in the browser.

- **Zero-dependency numerics** — ODE integration, Durand–Kerner polynomial root-finding, marching-squares contours, a µs-level protection-window EMT module, and OpenUSD export are all hand-written; React is the only runtime dependency.
- **EDL (Event Description Language)** — an open, versioned dictionary of data-center power/load events with machine-readable waveform profiles and **mandatory provenance** (grid-code clause, published incident, or public dataset). Certificates cite the exact dictionary version.
- **Coverage certificates** — parameter sweeps run in a Web Worker into pass / derate / outage heatmaps with zero-margin contours, exported as fingerprinted (sha256) JSON or a print-ready report, plus a diff mode that flags pass→fail flips against a prior certificate.
- **Calibrated against public data** — MIT Supercloud V100 traces and published GenAI H100 power profiles; every model parameter carries a declared calibration grade. Includes a DC-link control-loop resonance screening module validated against published cases (and the derivation caught a sign error in the source paper).
- **Physics under test** — 137 tests including a golden-value regression suite treated as a contract: changing a default parameter *is* changing a golden value.

---

## TrueProductData — industrial product-data platform for AI agents

**720k+ products · 2.3M typed spec facts · 13+ manufacturer sources · served over MCP** *(private infra)*

The pipeline that makes industrial parts data trustworthy enough for AI agents to quote from. Multi-source ingestion (manufacturer APIs, catalogs, official PDFs, part-number grammar decoding) → layered normalization → a provenance-first "golden" PostgreSQL database → an MCP server any agent can query.

- **Layered, idempotent normalization** — the raw layer stores manufacturer wording verbatim and is never overwritten; every derived layer is re-computable. A dictionary fix means recompute, never re-scrape.
- **A canonical spec vocabulary as versioned contracts** — `Ie`, "operational current / at AC-3 / rated value", and "Rated Operational Current (Ie)" all resolve to one typed key with unit, qualifiers, source, confidence, and dictionary version attached to every fact.
- **LLM-assisted, not LLM-in-the-loop** — Claude drafts mapping dictionaries and extracts specs from PDFs; the production normalization pass is deterministic, reproducible, and auditable.
- **Entity resolution** — exact MPN matching backed by DB constraints, GTIN cross-referencing, fuzzy matching, and cross-source validation with weighted confidence and a human review queue.
- **Contract-driven MCP server** — category-agnostic tools (`search_products` with parametric spec constraints, `find_equivalents` for cross-brand substitutes, `get_datasheet`, …) driven purely by the data contracts: a new product category becomes agent-queryable with zero new code.

---

## Stack

**AI systems:** Claude API (structured outputs, tool use, model tiering, cost governance) · RAG (pgvector, hybrid retrieval, RRF, eval harnesses) · MCP servers · prompt registries
**Backend:** Python · FastAPI · SQLAlchemy 2.0 async · PostgreSQL (pgvector, pg_bigm) · Celery + Redis · Alembic
**Frontend:** React 18 · TypeScript · Vite · Tailwind · Redux Toolkit / react-query · i18next (5 locales)
**Infra:** GCP (Cloud Run, Cloud SQL, Cloud Build, Secret Manager, Vertex AI) · Docker · Stripe · OAuth (Google & Microsoft) · Sentry · GitHub Actions
**Simulation:** hand-rolled numerical methods · DSL design · OpenUSD · Web Workers

---

*Building AI products and want to compare notes — or hiring for a team that ships? Email me: davidchen.cch@gmail.com*
