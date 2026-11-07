# Implementation Plan: Turiya v1 — Learn, Ask & Journal

**Branch**: `claude/dreamy-gauss-yzxial` | **Date**: 2026-11-07 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-turiya-ask-journal/spec.md`

## Summary

A React Native (Expo) app with three tabs:
- **Learn**: about 30 self-paced lessons on Kārikā chapter 1.
- **Ask**: a four-state reflection on an everyday problem.
- **Journal**: lesson responses and reflections.

Learn and Journal run **entirely on the phone and offline**. Lessons are reviewed Markdown files
bundled at build time, and all personal data lives in MMKV.

Only Ask uses the server, a thin TypeScript service:
1. crisis checks (word lists, then a model);
2. hybrid search over the Kārikā;
3. structured generation with Claude;
4. an output safety check.

The server stores nothing personal, and search embeddings run on the server itself, so Anthropic is
the only outside party that ever sees a question.

The order of work, the content track and the release checklist are in
[docs/IMPLEMENTATION_PLAN.md](../../docs/IMPLEMENTATION_PLAN.md). This plan covers the technical
design.

## Technical Context

**Language/Version**: TypeScript 5.x everywhere; Node.js 24 LTS on the server

**Primary Dependencies**:
- **App**: Expo SDK 58 (React Native 0.88, New Architecture), Expo Router, `react-native-mmkv`,
  `expo-notifications`, `@expo-google-fonts/*`, Zod.
- **Server**: Hono, AI SDK 6 (`ai`, `@ai-sdk/anthropic`), `@huggingface/transformers` with
  `bge-small-en-v1.5`, `pg` + `pgvector`, Zod.

**Storage**:
- **Phone**: MMKV, JSON, device only.
- **Server**: Postgres 16+ with pgvector, holding the shared corpus and anonymous counts only.

**Testing**: Vitest (shared, server), Jest + React Native Testing Library (app), Maestro (end to
end), and an eval runner for the golden and crisis sets.

**Target Platform**: iOS and Android at the versions Expo SDK 58 supports, built with Xcode 26 /
the iOS 26 SDK or later. The server runs as a Linux container in a Sydney region.

**Project Type**: mobile app + API, in a pnpm monorepo.

**Performance Goals**:
- The app opens to a lesson with no network call.
- A lesson renders in under 1 second.
- Ask returns a full reflection in under 30 seconds (SC-006); target p95 under 15 seconds.

**Constraints**:
- Learn and Journal work fully offline.
- No personal data on the server, and no question text in logs.
- No secrets in the app.
- Consent before any question is sent.
- Australian storefronts only.

**Scale/Scope**:
- About 30 lessons (41 verses), 3 tabs, about 8 screens.
- A corpus of a few hundred passages.
- Early users in the hundreds to low thousands. The server is stateless and scales horizontally.

All earlier unknowns are resolved in [research.md](./research.md). None remain.

## Constitution Check

*Gate: checked before research and again after design. Both pass.*

| Principle | How v1 meets it | Pass |
|---|---|---|
| I. Reflection, not prescription | The prompt and schema force reflective output. The output safety check blocks verdicts and advice. Lesson prompts are reviewed. Golden eval checks "no verdict" on every change | ✅ |
| II. Do no harm, no negativity | Output safety check; warm copy rules for lessons and the UI; built-in fallback is steadying | ✅ |
| III. Textual integrity | Verses fixed in reviewed files, never generated; canonical text and commentary in separate fields; every verse names its source; Sanskrit proofread; lessons reviewed by someone in the tradition (`reviewedBy` required for release) | ✅ |
| IV. Safety is a hard gate | Crisis check in code before generation (word lists + model) and on lesson responses (on the phone); output check after generation; built-in fallback on any failure | ✅ |
| V. Calm, faithful experience | Design tokens from the handoff; 44px targets; reduced motion; v1.1 and v2 features not built | ✅ |
| VI. Privacy by default | Journal, progress and responses on the phone only; no accounts; AI key on the server only; embeddings on our server, so only Anthropic sees questions; consent first | ✅ |
| VII. Evidence-based quality gates | Golden eval set (30–40 situations) and crisis test set run in CI on every prompt, model or content change; a regression blocks merge | ✅ |
| Tech: React Native client | Expo / React Native | ✅ |
| Tech: Postgres backend, not vector-only | pgvector **plus** full-text search, merged (research R3) | ✅ |
| Tech: cost-effective model tier | Cheapest model that passes the golden eval (research R13) | ✅ |
| Tech: canonical and commentary layers | Separate fields in lesson files and the database; `source` on every verse | ✅ |

No violations, so Complexity Tracking is empty.

## Project Structure

### Documentation (this feature)

```text
specs/001-turiya-ask-journal/
├── spec.md              # What v1 does
├── plan.md              # This file
├── research.md          # Technology decisions (Phase 0)
├── data-model.md        # Entities, rules, state changes (Phase 1)
├── quickstart.md        # How to run and validate v1 (Phase 1)
├── contracts/
│   ├── answer-api.md    # POST /v1/answer, GET /v1/health
│   └── lesson-file.md   # Lesson file format for authors and reviewers
├── checklists/
│   └── requirements.md  # Spec quality checklist
└── tasks.md             # Created by /speckit-tasks (not yet)
```

### Source Code (repository root)

```text
apps/
├── mobile/                      # Expo app
│   ├── app/                     # Expo Router screens
│   │   ├── (tabs)/learn.tsx, ask.tsx, journal.tsx
│   │   ├── lesson/[id].tsx
│   │   ├── support.tsx
│   │   └── consent.tsx
│   ├── src/
│   │   ├── features/learn/      # Path progress, unlock rule, lesson view
│   │   ├── features/ask/        # Ask flow, fallback reflection, API client
│   │   ├── features/journal/    # List, entry views, relative times
│   │   ├── features/safety/     # Support screen, on-phone crisis check, help link
│   │   ├── features/reminders/  # 7-day scheduling (research R6)
│   │   ├── storage/             # MMKV wrapper, versioned keys
│   │   └── theme/               # Design tokens, fonts
│   └── e2e/                     # Maestro flows (quickstart scenarios)
└── api/                         # Hono server
    ├── src/
    │   ├── routes/              # /v1/answer, /v1/health
    │   ├── pipeline/            # crisis, retrieve, generate, safety-check
    │   ├── embeddings/          # bge-small-en-v1.5 loader
    │   └── db/                  # queries
    └── tests/                   # unit + contract tests
packages/
└── shared/                      # Zod schemas (answer, lesson), four states,
                                 # crisis word lists, AU support services
content/
├── README.md                    # Pronunciation scheme, writing guide
└── path/karika-ch1/             # ~30 lesson files (contracts/lesson-file.md)
db/
├── migrations/
└── seed/                        # Loads content/ + licensed corpus, computes embeddings
evals/
├── golden/                      # 30–40 situations
├── crisis/                      # Crisis and non-crisis wording
└── run.ts                       # Eval runner used in CI
```

**Structure Decision**: a pnpm monorepo with two apps (`mobile`, `api`), one shared package, and
top-level `content/`, `db/` and `evals/`. Content sits at the top level because non-developers edit
it. Evals sit at the top level because they test the whole pipeline.

## Complexity Tracking

None. The Constitution Check has no violations.
