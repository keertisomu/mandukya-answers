# Turiya — learn the Māṇḍūkya Upaniṣad

A calm mobile app for **learning the Māṇḍūkya Upaniṣad through Gauḍapāda's
Māṇḍūkya Kārikā**, one short lesson a day, and for living what it teaches. When
something is weighing on you, **Ask** looks at it through the Upaniṣad's **four
states of consciousness**, with one verse attached per state. Lessons, your
responses and your reflections are kept in a private, device-only **Journal**.

> **Status:** planning / pre-implementation. This repository holds the v1 spec,
> requirements, architecture and plan. No application code yet.

## What v1 is

Three tabs — **Learn**, **Ask** and **Journal**:
- **Learn** is the centre. It's a Path of about 30 daily lessons through chapter 1
  of the Kārikā: the twelve mantras with Gauḍapāda's twenty-nine kārikās.
  - Each lesson has the verse, a plain-language explanation, and a "live it
    today" prompt.
  - One new lesson a day, no streaks, fully offline.
- **Ask** answers an everyday problem with a four-state reading (jāgrat → svapna →
  suṣupti → turīya) plus one small practice. It links to the lessons for its
  verses.
- **Journal** keeps lesson responses and Ask reflections on your device.

Crisis safety is built in from day one: crisis language gets helplines, not a
reflection. v1 launches in **Australia only**; India is next.

**v1.1** follows soon after launch: follow-up check-ins on past questions and a
"pattern mirror" of recurring stories in the Journal, both entirely on the
device.

## Guiding choices

- **Private by construction** — lessons, responses and the Journal never leave
  your device. Only an Ask question is sent, after you agree, and the server
  never stores it.
- **Faithful to the text** — lessons are fixed, proofread and reviewed by someone
  grounded in the tradition, never generated. Ask answers are grounded in
  retrieved passages of Gauḍapāda's Kārikā (RAG).
- **No model training** — the LLM provider's API tier does not train on inputs.
- **React Native (iOS + Android)** client; a thin, extensible **TypeScript API**
  over **Postgres + pgvector**, needed only for Ask.

## Docs

| Doc | What's in it |
|---|---|
| [specs/001-turiya-ask-journal/spec.md](specs/001-turiya-ask-journal/spec.md) | The product spec (source of truth): user stories, requirements, success criteria |
| [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) | Detailed requirements, content and design dependencies, acceptance criteria |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System design, lesson content pipeline, on-device data, the answer pipeline |
| [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md) | Content and design tracks, milestones M0–M8 (v1) and M9 (v1.1), risks |
| [docs/DECISIONS.md](docs/DECISIONS.md) | The key decisions (DEC-1…DEC-11) and what would change them |
| [docs/MARKET_VALIDATION.md](docs/MARKET_VALIDATION.md) | Demand, competitors, audience, feature ranking, risks |
| [design/](design/) | The original design handoff for Ask and Journal (reference only) |

## Open questions (see DECISIONS.md → Open questions)

- Translation source for Kārikā chapter 1 (+ licensing) — on the critical path.
- Who writes the lesson explanations, and who reviews them.
- Confirm the LLM model (default: Claude Sonnet 5.5).
- Expo vs bare React Native (default: Expo).
- Analytics in v1 (default: none).
