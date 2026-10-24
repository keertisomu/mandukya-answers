# Turiya — Māṇḍūkya guidance app

A calm mobile app where you type an everyday problem and get an answer that looks
at it through the **four states of consciousness** of the Māṇḍūkya Upaniṣad —
grounded in **Gauḍapāda's Māṇḍūkya Kārikā** (the commentary) — with one verse
attached per state. Every answer auto-saves to a private, device-only **Journal**.

> **Status:** planning / pre-implementation. This repository currently holds the
> v1 requirements, architecture, and plan. No application code yet.

## What v1 is

Two screens — **Ask** and **Journal** — and nothing else. The core is a single
request that returns a four-state reading (jāgrat → svapna → suṣupti → turīya)
plus one small practice to try today. Crisis safety is built in from day one:
crisis language gets helplines, not a reflection.

**v1.1** follows soon after launch: follow-up check-ins on past questions and a
"pattern mirror" of recurring stories in the Journal, both entirely on the
device.

## Guiding choices

- **Private by construction** — your problems and answers stay on your device; the
  server never stores them.
- **No model training** — the LLM provider's API tier does not train on inputs.
- **Faithful to the commentary** — answers are grounded in retrieved passages of
  Gauḍapāda's Kārikā (RAG), not the bare Upaniṣad.
- **React Native (iOS + Android)** client; a thin, extensible **TypeScript API**
  over **Postgres + pgvector**.

## Docs

| Doc | What's in it |
|---|---|
| [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) | Functional + non-functional requirements, scope, acceptance criteria |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System design, client/backend, data model, the answer pipeline |
| [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md) | Milestones M0–M7 (v1) and M8 (v1.1), critical path, risks |
| [docs/DECISIONS.md](docs/DECISIONS.md) | The key decisions (DEC-1…DEC-10) and what would change them |
| [docs/MARKET_VALIDATION.md](docs/MARKET_VALIDATION.md) | Demand, competitors, audience, v2 feature ranking, risks |
| [design/](design/) | The original high-fidelity design handoff (reference only) |

## Open questions (see DECISIONS.md → Open questions)

- Confirm the LLM model (default: Claude Sonnet 5.5).
- Translation source for the Kārikā corpus (+ licensing).
- Expo vs bare React Native (default: Expo).
- Analytics in v1 (default: none).
