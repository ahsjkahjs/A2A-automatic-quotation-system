# A2A Automated Quoting System — Architecture Design

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Static Site](https://img.shields.io/badge/site-single%20HTML-success.svg)](#viewing-locally)
[![No Build](https://img.shields.io/badge/build-none-lightgrey.svg)](#viewing-locally)

A single-page, self-contained **conceptual architecture blueprint** for an
**Agent-to-Agent (A2A) automated quoting system** for B2B procurement and
supply-chain tendering. Buyer and supplier agents, each running inside their own
organization's security boundary, automatically complete the cross-enterprise
loop of **RFQ → quote → compare / negotiate → award → close & settlement** over
the open [Agent2Agent (A2A)](https://a2a-protocol.org/) protocol.

> **The LLM never sets the price directly.** The model handles understanding,
> orchestration and negotiation; deterministic rules and pricing models produce
> the price inside an authorized range; humans keep control of every key
> decision.

---

## Table of contents

- [Overview](#overview)
- [Why A2A (and how it relates to MCP)](#why-a2a-and-how-it-relates-to-mcp)
- [Key design principles](#key-design-principles)
- [Document contents](#document-contents)
- [Repository structure](#repository-structure)
- [Viewing locally](#viewing-locally)
- [Deploying to GitHub Pages](#deploying-to-github-pages)
- [Customization](#customization)
- [Technical notes](#technical-notes)
- [Rollout roadmap](#rollout-roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Disclaimer](#disclaimer)

---

## Overview

Traditional B2B request-for-quotation (RFQ) workflows are person-to-person or
portal-to-portal: emails, spreadsheets, supplier portals, phone calls and manual
approvals. They are slow, hard to compare, and do not scale to many suppliers and
many rounds of negotiation.

This blueprint upgrades that process to **agent-to-agent** collaboration:

- Each organization runs its **own agent** inside its own trust boundary.
- Externally, agents exchange **structured RFQs and quotes** over the open,
  vendor-neutral A2A protocol (JSON-RPC, SSE, webhooks, Agent Card discovery).
- Internally, agents call inventory, capacity, cost and pricing capabilities
  through tool interfaces (e.g. MCP) — sensitive data never leaves the domain.
- **Sealed bids**, a **human-in-the-loop (HITL) authority matrix**, idempotency,
  Saga compensation and an end-to-end audit trail make automation trustworthy.

The result is transaction infrastructure built on **protocol interoperability +
deterministic pricing + controllable autonomy + end-to-end traceability** — not a
chatbot that invents prices.

## Why A2A (and how it relates to MCP)

A2A and MCP are **complementary and orthogonal**, not competing:

| | **A2A** | **MCP** |
| --- | --- | --- |
| Connects | Agent ↔ agent (**across organizations**) | Agent ↔ tools / data (**inside one organization**) |
| Carries | Business tasks: RFQ, Quote, negotiation, award (Task / Message / Artifact) | Tool discovery and invocation (resources / tools / prompts) |
| Style | Stateful, long-lived, multi-turn, async streaming / push | Usually short request / response; exposes capabilities |
| Identity | Organization-level DID / mTLS / verifiable credentials | Internal service accounts, least-privilege scopes |
| Example | Buyer agent → supplier agent: *"quote these 500 parts, reply in 3 days"* | Supplier agent → internal ERP: *"query inventory and standard cost for SKU-123"* |

Cross-domain messages carry **only** the RFQ and the Quote (business semantics).
Cost build-up, floor prices, capacity and customer master data stay in the private
domain and are never serialized into A2A messages.

## Key design principles

1. **The LLM never prices.** Pricing is produced by deterministic rules /
   optimization models within a feasible region (cost floor, target margin,
   discount authority). The LLM only parses, clarifies and negotiates inside that
   region; out-of-range or below-cost requests are blocked and escalated.
2. **Interoperable by protocol.** Built on open A2A and JSON-Schema contracts so
   agents from different vendors and models can plug in without point-to-point
   integration. A new supplier joins simply by publishing an Agent Card.
3. **Sealed, fair bidding.** Before the deadline, quotes are encrypted / sealed
   (public-key or TEE) and invisible to buyers, suppliers and the platform alike;
   threshold decryption opens them at the deadline with a trusted timestamp.
4. **Controllable autonomy.** A HITL permission matrix routes decisions by
   amount, discount depth, anomaly, supplier novelty and contract-term deviation
   — auto, manager approval, multi-level / committee, or deny.
5. **Data stays in-domain.** Cost, floor price, capacity and customer lists never
   leave each organization; only authorized messages and necessary credentials
   cross domains (data minimization and data sovereignty).
6. **Eventually consistent and replayable.** Idempotency keys, async-first
   messaging, transactional Outbox/Inbox, Saga compensation, quote validity /
   price locking and four-party reconciliation.
7. **End-to-end auditable.** RFQs, quotes, counter-offers, pricing rationale,
   approvals, bid opening and awards are written to an append-only, hash-chained
   audit log.

## Document contents

The page (`index.html`) contains 14 sections:

1. [Overview & Design Goals](index.html#overview)
2. [Actors & Trust Boundaries](index.html#actors)
3. [Layered Architecture (L0–L6)](index.html#architecture)
4. [End-to-End Process (4 phases, 4 gates)](index.html#flow)
5. [A2A Protocol Interaction Sequence](index.html#sequence)
6. [Quote Task State Machine](index.html#state)
7. [Pricing Engine](index.html#engine)
8. [Trust, Security & Governance (+ HITL matrix)](index.html#trust)
9. [Reliability & Cross-Org Consistency](index.html#reliability)
10. [Federated Deployment](index.html#deployment)
11. [Reference Technology Stack](index.html#stack)
12. [Phased Rollout Roadmap](index.html#roadmap)
13. [Key Risks & Mitigations](index.html#risk)
14. [Protocol Split & Data Contracts (+ JSON samples)](index.html#contract)

All diagrams (actor overview, sequence diagram, state machine, deployment) are
**hand-authored inline SVG / CSS** — no images, no diagram services.

## Repository structure

```
a2a-auto-quoting-architecture/
├── index.html        # The entire single-page blueprint (HTML + CSS + JS + inline SVG)
├── README.md         # This file
├── LICENSE           # MIT License
└── .gitignore
```

There are intentionally **no build files, package managers or dependencies**.

## Viewing locally

No build step and no server are required.

- **Option A — open directly:** double-click `index.html` (or drag it into a
  browser).
- **Option B — serve locally** (optional, closest to production):

  ```bash
  # Python 3
  python -m http.server 8000
  # then open http://localhost:8000
  ```

  ```bash
  # or Node.js
  npx serve .
  ```

Works in any modern browser (Chrome, Edge, Firefox, Safari) and is responsive
down to mobile widths.

## Deploying to GitHub Pages

1. Create a repository on GitHub and push these files:

   ```bash
   git init
   git add .
   git commit -m "Add A2A automated quoting architecture blueprint"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<your-repo>.git
   git push -u origin main
   ```

2. On GitHub, open **Settings → Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Select the `main` branch and the `/ (root)` folder, then click **Save**.
5. Your site will be live at
   `https://<your-username>.github.io/<your-repo>/` within a minute.

Because the entry point is `index.html`, no build or framework preset is needed.

## Customization

- **Colors / theme:** all design tokens are CSS custom properties at the top of
  `index.html` under `:root` (e.g. `--buyer`, `--seller`, `--market`, `--gov`,
  `--ink`, `--paper`). Change them once to re-theme the whole document.
- **Fonts:** the page loads *Manrope*, *Newsreader* and *IBM Plex Mono* with full
  system-font fallbacks (`--sans`, `--serif`, `--mono`). Swap the `<link>` and
  the variables to use your own type system; the page remains readable if the web
  fonts cannot be fetched.
- **Content:** each section is a plain `<section id="...">`; edit text directly.
  Diagrams are inline SVG, so labels are searchable and translatable.
- **Company / scenario details:** thresholds in the HITL matrix, milestone
  timings, the tech stack and the JSON samples are illustrative placeholders —
  replace them with your organization's policies and data.

## Technical notes

- Pure **HTML5 + vanilla CSS + vanilla JavaScript**; no frameworks, no build, no
  external JavaScript libraries.
- The only external requests are web fonts (Google-Fonts-compatible CDN); the
  document degrades gracefully offline thanks to system-font fallbacks.
- Diagrams are inline SVG and CSS, so the page can be printed or saved to PDF.
- The small amount of JavaScript only highlights the current section in the top
  navigation (via `IntersectionObserver`) and is wrapped in a no-op guard.
- Responsive breakpoints at 960px and 600px; wide diagrams scroll horizontally
  inside their containers on small screens.

## Rollout roadmap

| Milestone | Window | Focus |
| --- | --- | --- |
| **M1** | 0–2 mo | Point-to-point connection: one buyer + one supplier, Agent Card discovery, RFQ / quote, basic rule pricing, manual approval |
| **M2** | 2–4 mo | Multi-supplier comparison: fan-out RFQ, TCO comparison, SSE + webhook, idempotency / Outbox, validity & price lock |
| **M3** | 4–6 mo | Autonomy & bidding: optimization-based pricing engine, multi-round negotiation, HITL matrix, MCP/ERP integration |
| **M4** | 6–9 mo | Trusted closed loop: sealed bids + TEE / threshold opening, anti-collusion, Saga compensation, contract / PO / reconciliation |
| **M5** | 9–12 mo | Scale & federation: multi-tenant / multi-region, reputation & credential network, reverse auction, industry templates |

Each milestone ships standalone value — you do not need the full trust /
federation stack before launching.

## Contributing

This is a conceptual reference; corrections, clarifications and additional
patterns are welcome.

1. Fork the repository.
2. Create a feature branch (`git checkout -b improve/<short-description>`).
3. Make your changes to `index.html` / `README.md`.
4. Verify the page opens cleanly and remains responsive.
5. Commit and open a pull request describing the change and rationale.

Please keep the document vendor-neutral and keep the core principle intact:
**deterministic pricing with controllable autonomy**, not free-form LLM pricing.

## License

Released under the [MIT License](LICENSE).

## Disclaimer

This document is a **conceptual, illustrative architecture design**, not an
official topology of the A2A protocol or of any named vendor. Protocol field
names should be checked against the current official
[A2A specification](https://a2a-protocol.org/); components describe
**capabilities, not physical deployment constraints**. Company names, identifiers,
amounts, thresholds and dates in the samples are fictitious examples. Any
production deployment must be reviewed against applicable procurement,
data-protection, trade-control and industry regulations.
