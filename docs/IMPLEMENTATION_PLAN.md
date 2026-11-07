# Implementation plan — v1

The order of work for v1. What to build is in the
[spec](../specs/001-turiya-ask-journal/spec.md); how is in [ARCHITECTURE.md](./ARCHITECTURE.md).

## The approach in three lines

1. **Content is the critical path, not code.** Writing and reviewing about 30 lessons takes longest,
   so it starts on day one.
2. **Build Learn first.** It needs no server and is the core of the product.
3. **Then Ask**: first on the built-in fallback, then the real server, safety checks and RAG.

## Parallel tracks (start with M0)

### Content — gates release
| Step | Work |
|---|---|
| C1 Translation | Choose and license (or confirm public domain) an English translation of Kārikā chapter 1. Record the source. |
| C2 Text | Devanagari, IAST and an easy pronunciation guide for the 12 mantras and 29 kārikās, proofread against a Sanskrit source. Also correct the four Ask verse references (see ARCHITECTURE). |
| C3 Lessons | Group about 41 verses into about 30 lessons. Write each explanation (3–5 minutes to read) and its "live it today" prompt, in the app's warm, non-advising voice. |
| C4 Review | Someone grounded in the tradition reviews every lesson. Fixes go in as pull requests to `content/`. |
| C5 Helplines | Verify the Australian support services. Repeat every 90 days. |

Finish 2–3 lessons end-to-end early. They become the real test content and set the tone.

### Design — needed before M2 finishes
Designs for Learn (today's lesson, progress, completed lessons, completion moment), the reminder
setting, the consent screen and the three-tab bar, in the style of the existing handoff.

## Milestones

| # | Milestone | Done when |
|---|---|---|
| M0 | **Setup**: monorepo (`apps/mobile`, `apps/api`, `packages/shared`, `content/`, `db/`), TypeScript, lint, tests; shared schemas (answer, lesson) and crisis word lists; a CI check that every lesson file is valid | Install, typecheck, lint and content check pass |
| M1 | **App shell**: Expo app opening to Learn; design tokens and base components; three tabs; reduced-motion support | All tabs navigable; Ask matches the handoff |
| M2 | **Learn, offline** (FR-001–009): lessons bundled; lesson screen; progress and in-order unlock; responses to the Journal; support screen and help link on every tab; on-device crisis check; daily reminder | Lessons work in order, offline; responses reach the Journal; help is one tap away (SC-003, SC-012) |
| M3 | **Ask on the fallback** (FR-010–017, FR-033): Ask, thinking and answer screens; Ask entries in the Journal; Learn ↔ Ask links; consent screen (FR-036) | Ask works end-to-end with the built-in reflection; no question is sent without consent (SC-011, SC-014) |
| M4 | **Server**: `/v1/answer` and `/v1/health`; crisis check (word lists, then model); output safety check; crisis test set in CI | Real reflections; failures fall back; crisis test set passes (SC-007, SC-010) |
| M5 | **Database**: Postgres + pgvector; migrations; seed the corpus (chapter 1 from `content/`) with embeddings and keyword index | Hybrid search returns sensible passages for sample questions |
| M6 | **Grounded answers**: hybrid retrieval in the pipeline; build the golden eval set (30–40 situations) and run it in CI | Answers reflect Gauḍapāda; golden eval passes (SC-008) |
| M7 | **Privacy and polish**: confirm nothing leaves the phone except consented questions; no text in logs; swap in the reviewed lessons; accessibility pass | All success criteria in the spec pass (SC-001–SC-017) |
| M8 | **Release**: see the checklist below | Builds submitted; server live |

### Release checklist (M8)
- **Store listing**:
  - presented as learning the Upaniṣad;
  - includes the attribution line;
  - makes no therapy or medical claims.
- **Apple App Store**:
  - privacy label: question text collected, not linked to the person, not used for tracking;
  - age questionnaire answered honestly, then 18+ chosen.
- **Australian storefronts only**: check both stores exclude every other country (FR-038).
- **Privacy policy**:
  - names Anthropic as the AI provider;
  - covers processing outside Australia;
  - has had a legal review.
- **Build**: with Expo SDK 54 or later (Xcode 26 / iOS 26 SDK), as Apple requires since April 2026.
- **Server**: live during app review. Review notes explain the crisis flow.
- **Order**: iOS first, if review capacity is limited.

## Risks

| Risk | What we do |
|---|---|
| Lessons take longer than planned | Start on day one. If needed, launch a shorter first Path (the 12 mantras with key kārikās) rather than delay. |
| No qualified reviewer | Line one up at M0, e.g. through Vedanta societies or teachers in Australia. |
| Translation licensing | Start early and keep a public-domain fallback in mind. |
| Sanskrit errors | Proofread before launch. Lessons are files, so fixes are quick. |
| AI drifts into advice | Schema checks, the safety check, the golden eval set in CI, and the app's fallback. |
| Missed crisis | Word lists on both paths, the model check for Ask, the test set in CI, and help on every tab. |
| Too many false crisis alarms | Acceptable by design. Watch the anonymous count after prompt changes. |
| Notification fatigue | Reminder off by default, at most one a day, skipped on days a lesson was already done. |

After launch: v1.1 and beyond are in [ROADMAP.md](./ROADMAP.md).
