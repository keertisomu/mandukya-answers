# Implementation plan — v1

A milestone-based plan. Each milestone is independently demoable. References:
[REQUIREMENTS.md](./REQUIREMENTS.md), [ARCHITECTURE.md](./ARCHITECTURE.md),
[DECISIONS.md](./DECISIONS.md), and the product spec
[specs/001-turiya-ask-journal/spec.md](../specs/001-turiya-ask-journal/spec.md).

v1 centres on the daily lesson Path (DEC-11). The Path needs **no backend**, so
the order is:
1. Build Learn and Journal first, fully offline. This is the product's core and
   carries the least technical risk.
2. Add Ask on the built-in fallback.
3. Add the backend, safety pipeline and RAG last.

**Content is the critical path, not code.** Writing and reviewing about 30
lessons takes longer than building the screens, so the content track starts on
day one, in parallel with M0.

---

## Content track (C1–C4) — starts with M0, gates release

- **C1 Translation.** Choose and license (or confirm public domain) an English
  translation of Kārikā chapter 1; record the source (CR-2, CR-6).
- **C2 Text.** Devanagari and IAST for all 12 mantras and 29 kārikās, proofread
  against a Sanskrit source. This includes correcting the four Ask verse
  references (CR-1; see ARCHITECTURE §2.3).
- **C3 Lessons.** Group the ~41 verses into ~30 lessons; write each explanation
  (3–5 minutes to read) and its "live it today" prompt in the app's warm,
  non-prescriptive voice.
- **C4 Review.** Every lesson reviewed by someone grounded in the tradition;
  corrections merged through pull requests on `content/path/karika-ch1/`.
- **Exit:** all ~30 lesson files merged, schema-valid, reviewed (acceptance
  criterion 11).

Get two or three finished lessons through C1–C4 early. They become the real
content for building and testing M2, and they set the tone for the rest.

## Design track (D1)
- **D1** Designs for Learn (today's lesson, progress, completed lessons,
  completion moment), the reminder setting, the AI-consent screen and the
  three-tab bar, in the handoff's visual language (CR-7). Needed before M2's UI
  work finishes.

---

## M0 — Project setup
- Monorepo scaffold: `apps/mobile`, `apps/api`, `packages/shared`, `content/`,
  `db/`.
- TypeScript, lint/format, and a SessionStart-friendly test setup.
- `packages/shared`:
  - the state→verse constants;
  - the Zod **answer schema** (`{ levels[4]{heading,body}, practice }`);
  - the **lesson schema**;
  - the crisis rule lists.
- A content validation script: every lesson file in `content/path/` must match
  the schema; run it in CI.
- **Exit:** `pnpm install`, typecheck, lint and content validation pass.

## M1 — Client shell & design system (NFR-6)
- Expo app boots to the **Learn** tab.
- Encode design tokens (colors, Newsreader/Karla/Noto Serif Devanagari, radii,
  type scale, input-card shadow) and base components.
- Three-tab nav (Learn / Ask / Journal); reduce-motion plumbing (NFR-5).
- **Exit:** all three tabs navigable; Ask matches the handoff visually on a device.

## M2 — Learn: the Path, offline (spec FR-033–FR-040, FR-043)
- Bundle `content/path/karika-ch1/` into the app at build time.
- Lesson screen: verse (Devanagari, IAST, translation, reference, source),
  explanation, related state, "live it today" prompt, optional response.
- Path progress in MMKV (`turiya_path_v1`); unlock rule (one new lesson per
  calendar day after the last completion); progress display; re-reading
  completed lessons; completion moment.
- Lesson responses saved as `kind: "lesson"` Journal entries; Journal lists both
  kinds.
- Safety UI (DEC-8):
  - the support screen with bundled Australian helplines;
  - the "Need to talk to someone?" link on Learn, Ask and Journal;
  - the on-device crisis check on lesson responses.
- Optional daily reminder (local notification), off by default.
- **Exit:** a person can complete lessons day by day, offline, see responses in
  the Journal, and reach help from every tab. Acceptance criteria 1, 12, 13, 15
  pass (with placeholder content until C4 finishes).

## M3 — Ask + Answer with the built-in fallback (FR-1–FR-20)
- Build Ask (greeting, input, "Look at it", chips, four-states primer),
  Thinking (breathing halo + cycling status), Answer (four accordion cards with
  hard-coded verses + practice card).
- Ask entries in the Journal; links between Ask verses and Path lessons (spec
  FR-042).
- AI-consent screen before the first Ask (spec FR-041); declining leaves Learn and
  Journal usable.
- Wire the full flow using **only** the ported `fallback()`, with no network yet.
- **Exit:** a person can ask, get a (fallback) four-state answer, see it saved, and
  reopen it from the Journal across restarts. Acceptance criteria 3, 4, 6, 7, 14
  pass.

## M4 — Backend skeleton + `/v1/answer` (ungrounded) (FR-21–FR-24, NFR-3/7/9)
- Thin TS service (Vercel AI SDK + `@ai-sdk/anthropic`), key in env only.
- `POST /v1/answer` → `generateObject` with the system prompt and the shared Zod
  schema (no RAG yet). `GET /v1/health`.
- Safety pipeline (DEC-8):
  - shared rule lists as a first gate, then the model crisis screen;
  - an output safety check after generation;
  - the `kind: "reflection" | "crisis"` response;
  - anonymous counters.
- Build the **crisis test set**, covering both Ask questions and lesson responses
  and including everyday idioms like "this job is killing me", and run it in CI.
- Point the client at it; keep fallback on any failure.
- **Exit:** real four-state answers for typed problems; invalid/failed responses
  still fall back cleanly; the crisis test set passes. Acceptance criteria 2, 8, 9
  pass.

## M5 — Postgres + pgvector + corpus seeding (DEC-5, CR-1/CR-2)
- Provision Postgres with pgvector; migrations for `karika_passage`,
  `state_verse`, `daily_counter`.
- Seed script: chapter 1 verses come from `content/path/`, the rest of the corpus
  from the licensed source → chunk → embed → insert; populate `state_verse` from
  the proofread map.
- **Exit:** corpus queryable; nearest-neighbour retrieval returns sensible
  passages for sample questions.

## M6 — RAG grounding in `/v1/answer` (DEC-7, FR-22)
- Pipeline: embed question → retrieve top-k → pass passages as grounding context
  → `generateObject`.
- Light eval pass: run ~10–15 representative problems, read answers for fidelity
  to the commentary and tone; tune prompt/k.
- **Exit:** answers visibly reflect Gauḍapāda's commentary; schema always honored;
  acceptance criteria 2 and 7 hold with real content.

## M7 — Privacy hardening & polish (NFR-1/2/4/5)
- Verify no question/answer text in logs or DB, that crisis routing is only ever
  a count, and that lesson responses are never transmitted (acceptance criteria
  5, 8, 13).
- Verify every helpline number and region (CR-4).
- Re-run the crisis test set against the final RAG prompt and models.
- Privacy copy in-app: device-only storage, consent screen, no-training provider
  statement (DEC-6).
- Swap in the reviewed lesson content from C4.
- Offline behavior: Learn and Journal fully offline; Ask without network →
  fallback.
- Accessibility pass: reduce-motion, 44px targets, contrast.
- **Exit:** all acceptance criteria (1–15) pass.

## M8 — Release prep
- App icons/splash, store metadata, privacy nutrition labels reflecting DEC-1/6.
- Store listing review: positioned as learning the Upaniṣad, attribution line
  present, no therapeutic, clinical or medical claims (NFR-10, acceptance
  criterion 10).
- App Store age rating: answer the questionnaire honestly, then choose 18+.
- Release to the **Australian storefronts only** (DEC-10); confirm both stores
  exclude every other country.
- Australian privacy check: consent screen and data-handling notice before the
  first Ask, privacy policy naming the AI provider and covering processing outside
  Australia, legal review (spec FR-031, FR-041).
- Launch iOS first if store review capacity is limited (market research: iOS
  converts ~2× better and fits the first audience).
- Backend deploy + health checks; secrets configured; basic non-PII request
  logging/metrics. Keep the backend live during app review.
- Build with Expo SDK 54 or later (Xcode 26 / iOS 26 SDK) and submit iOS +
  Android.
- **Exit:** builds submitted; backend live.

## M9 — v1.1 retention slice (REQUIREMENTS §7, DEC-9)
- **Follow-ups:** `expo-notifications` local scheduling 3 days after an answer;
  generic notification text; deep link to the entry; follow-up input saved on the
  entry; permission asked after the first answer (or reused from the reminder);
  settings toggle; never for crisis-routed questions.
- **Story labels:** taxonomy in `packages/shared` (reviewed, CR-5); backend schema
  adds `story` with `key` constrained to the enum; client saves it on the entry
  and shows the phrase on the svapna card.
- **Pattern mirror:** on-device grouping by `story.key`; Patterns section at 5+
  labelled entries.
- Back-compat: earlier entries (no `story`, no `followUps`) render unchanged.
- **Exit:** v1.1 acceptance criteria 1–4 pass. Start the week-4 retention
  comparison (FR-40).

---

## Critical path & parallelism
- **Content (C1–C4)** is the long-lead item and gates release. Start it at M0,
  and get a few finished lessons early. Helpline verification (CR-4) is short but
  also gates launch.
- **Design (D1)** must land before M2's UI is finished.
- M1–M3 (client, Path, Ask on fallback) need no backend and can proceed in
  parallel with M4–M6.
- M4 unblocks real answers; M5→M6 add fidelity.

## Risks & mitigations
- **Lesson content takes longer than expected** (C3/C4) → start on day one;
  build with a few real lessons; if needed, launch with a shorter first Path
  (e.g. the twelve mantras with their key kārikās) rather than delay.
- **Translation licensing** (CR-2) → start early; have a public-domain fallback
  text identified.
- **Sanskrit accuracy** (CR-1) → proofread with a qualified reader before launch;
  content lives in reviewed files, so errors are correctable in one place.
- **Finding a qualified reviewer** (C4) → line one up at M0; consider Vedanta
  societies or teachers in Australia.
- **Model output drift** → schema validation + client fallback make bad output
  non-fatal; the light eval in M6 catches tone/fidelity regressions.
- **Framework churn** → staying on the AI SDK (not Mastra) in v1 keeps the surface
  small; Mastra remains an additive v2 option (DEC-4).
- **Missed crisis** → shared rule lists on both paths, the recall-tuned model
  screen for Ask, a maintained test set in CI, and the always-visible help link
  as a backstop (DEC-8).
- **False crisis alarms annoy users** → acceptable by design (NFR-11); watch the
  anonymous `crisis_routed` count for sudden jumps after prompt changes.
- **Notification fatigue** → reminder off by default, at most one a day, skipped
  once the lesson is opened; one follow-up per entry in v1.1; easy off switches.

## Explicitly out of scope for this plan
Everything under Non-goals in REQUIREMENTS.md (Śaṅkara, chapters 2–4, a verse
library, the daily four-states rhythm, bedtime wind-down, streaks, community,
accounts, sync, server push, analytics dashboards).
