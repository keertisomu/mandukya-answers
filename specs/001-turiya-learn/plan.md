# Implementation Plan: Turiya v1 — Learn the Māṇḍūkya

**Branch**: `claude/dreamy-gauss-yzxial` | **Date**: 2026-11-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-turiya-learn/spec.md`

## Summary

A React Native (Expo) app with two tabs:
- **Learn**: about 32 self-paced, phrase-by-phrase lessons through Kārikā chapter 1, plus the
  person's saved lessons.
- **Journal**: the person's replies to lesson prompts.

**There is no server, no AI and no network use.**
- Lessons are reviewed Markdown files, validated and bundled at build time.
- Progress, saved lessons, the Journal and settings live in MMKV on the phone.
- Crisis checks on replies run on the phone.
- The reminder is a local notification.

The order of work, the content track and the release checklist are in
[docs/IMPLEMENTATION_PLAN.md](../../docs/IMPLEMENTATION_PLAN.md). The Ask server design waits in
[docs/ASK_V2_DESIGN.md](../../docs/ASK_V2_DESIGN.md).

## Technical Context

**Language/Version**: TypeScript 5.x

**Primary Dependencies**: Expo SDK 58 (React Native 0.88, New Architecture), Expo Router,
`react-native-mmkv`, `expo-notifications`, `@expo-google-fonts/*`, Zod. Not included:
`expo-updates`, analytics or crash-reporting SDKs (research R8).

**Storage**: MMKV on the phone (JSON). No server storage.

**Testing**: Vitest (shared package and content build), Jest + React Native Testing Library
(screens), Maestro (end to end), and a crisis test set in CI.

**Target Platform**: iOS and Android at the versions Expo SDK 58 supports, built with Xcode 26 /
the iOS 26 SDK or later.

**Project Type**: mobile app (no backend), in a pnpm monorepo.

**Performance Goals**:
- The app opens to Learn in under 2 seconds on a mid-range phone.
- A lesson renders in under 1 second.
- A saved lesson opens in two taps or fewer.

**Constraints**:
- The whole app works offline.
- Nothing about the person leaves the phone.
- Lessons are never generated.
- Australian storefronts only.

**Scale/Scope**:
- About 32 lessons (41 verses, around 150–200 phrases).
- 2 tabs and about 6 screens: Learn, lesson, Journal, journal entry, support, settings.
- A bundle of lesson content under 1 MB.

All unknowns are resolved in [research.md](./research.md). None remain.

## Constitution Check

*Gate: checked before research and again after design. Both pass.*

| Principle | How v1 meets it | Pass |
|---|---|---|
| I. Reflection, not prescription | Lessons teach the text. Prompts are small reflections, reviewed, never advice about the person's life | ✅ |
| II. Do no harm, no negativity | Warm-tone writing rules for every lesson and screen; review before release | ✅ |
| III. Textual integrity | Sanskrit, meanings and verses are fixed, proofread and reviewed, never generated. Canonical verse text sits in separate fields from our notes and summaries. Every verse names its translation source. Copyrighted commentaries aren't quoted | ✅ |
| IV. Safety is a hard gate | Crisis check in code on every saved reply (on the phone), and help one tap away. There's no generation in v1, so the pre- and post-generation gates and the fallback start with Ask | ✅ (gates for generation deferred, see below) |
| V. Calm, faithful experience | Design tokens from the approved style; 44px targets; reduced motion; Ask and later features not built early | ✅ |
| VI. Privacy by default | Everything on the phone; no accounts, server, analytics or OTA updates; App Store label "Data Not Collected" | ✅ |
| VII. Evidence-based quality gates | The golden eval set tests generated responses, so it starts with Ask (v2). In v1, quality is gated by proofreading and tradition review (SC-003), plus the crisis test set in CI | ✅ (deferred, see below) |
| Tech: React Native client | Expo / React Native | ✅ |
| Tech: server API on PostgreSQL; hybrid retrieval; LLM via backend | Not needed in v1: no server, search or AI. Designed for v2 in `docs/ASK_V2_DESIGN.md` | ✅ (deferred) |
| Tech: canonical and commentary layers | Separate fields in lesson files (`fullVerse`, `phrases[].devanagari/iast` vs `note`, `summary`) | ✅ |

### Deferred to v2 (recorded as the constitution requires)

| Item | Why it's not in v1 | When it starts |
|---|---|---|
| Pre- and post-generation safety gates, built-in fallback (Principle IV) | v1 generates nothing | With Ask (v2) |
| Golden eval set (Principle VII) | It tests generated responses; there are none in v1 | With Ask (v2) |
| Server, Postgres and hybrid retrieval (tech constraints) | v1 has no server | With Ask (v2) |

## Project Structure

### Documentation (this feature)

```text
specs/001-turiya-learn/
├── spec.md              # What v1 does
├── plan.md              # This file
├── research.md          # Technology decisions (Phase 0)
├── data-model.md        # Content and phone data, rules, state changes (Phase 1)
├── quickstart.md        # How to run and validate v1 (Phase 1)
├── contracts/
│   └── lesson-file.md   # Lesson file format for authors and reviewers
├── checklists/
│   └── requirements.md  # Spec quality checklist
└── tasks.md             # Created by /speckit-tasks (not yet)
```

### Source Code (repository root)

```text
apps/
└── mobile/                      # Expo app
    ├── app/                     # Expo Router screens
    │   ├── (tabs)/learn.tsx, journal.tsx
    │   ├── lesson/[id].tsx
    │   ├── journal/[id].tsx
    │   ├── support.tsx
    │   └── settings.tsx
    ├── src/
    │   ├── features/learn/      # Path, progress, unlock rule, lesson + phrase views
    │   ├── features/saved/      # Save/unsave, Saved list
    │   ├── features/journal/    # Replies, list, relative times
    │   ├── features/safety/     # Support screen, crisis matcher, help link
    │   ├── features/reminders/  # 7-day local scheduling (research R6)
    │   ├── storage/             # MMKV wrapper, versioned keys
    │   └── theme/               # Design tokens, fonts
    └── e2e/                     # Maestro flows (quickstart scenarios)
packages/
└── shared/                      # Zod lesson schema, crisis word lists,
                                 # AU support services, content build script
content/
├── README.md                    # Pronunciation scheme, writing guide
└── path/karika-ch1/             # ~32 lesson files (contracts/lesson-file.md)
```

**Structure Decision**: a pnpm monorepo with one app, one shared package and top-level `content/`.
Content sits at the top level because non-developers edit it. v2 adds `apps/api`, `db/` and
`evals/` alongside, without moving anything.

## Complexity Tracking

None. Nothing is added beyond the spec; the deferrals above remove complexity rather than add it.
