# Implementation plan — v1

The order of work for v1. What to build is in the [spec](../specs/001-turiya-learn/spec.md); how is
in [ARCHITECTURE.md](./ARCHITECTURE.md) and the [plan](../specs/001-turiya-learn/plan.md).

## The approach in three lines

1. **Content is the critical path, not code.** Writing and reviewing about 32 phrase-by-phrase
   lessons takes longest, so it starts on day one.
2. **The app is simple.** There's no server, AI or network, just a well-made offline reader with a
   journal.
3. **Build with real lessons early.** Finish mantra 7 first; it's the heart of the text and the
   hardest lesson to lay out.

## Parallel tracks (start with M0)

### Content — gates release
| Step | Work |
|---|---|
| C1 Translation | Choose and license (or confirm public domain) an English translation of Kārikā chapter 1 for the full-verse text. Record the source. |
| C2 Sanskrit | Devanagari, IAST and pronunciation for every phrase and full verse (12 mantras, 29 kārikās), proofread against a Sanskrit source. Write the pronunciation scheme in `content/README.md`. |
| C3 Lessons | Split each verse into phrases. Write each phrase's meaning and note, then each verse's summary and prompt, in our own words. Split long verses into parts (mantra 7 into 3). |
| C4 Review | Someone grounded in the tradition reviews every lesson. Fixes go in as pull requests to `content/`. |
| C5 Helplines | Verify the Australian support services. Repeat every 90 days. |

**Order**: mantra 7 first (lessons for all 3 parts), then mantras 1–6, then the rest in order.

### Design — needed before M2 finishes
Designs in the existing visual style for:
- the Learn home (Saved, continue, progress);
- the lesson layout (phrase cards, part headings, the verse ending);
- the Journal;
- the support screen;
- settings (reminder);
- the two-tab bar.

## Milestones

| # | Milestone | Done when |
|---|---|---|
| M0 | **Setup**: monorepo (`apps/mobile`, `packages/shared`, `content/`), TypeScript, lint, tests; lesson schema and content build; crisis word lists; CI | Install, typecheck, lint and `content:check` pass |
| M1 | **App shell**: Expo development build opening to Learn; design tokens, bundled fonts; two tabs; reduced-motion support | Both tabs navigable offline |
| M2 | **Lessons** (FR-001–FR-008): lesson screen with phrases, parts and verse ending; progress and in-order unlock; completion moment | Mantra 7 reads well across its 3 parts, offline (SC-009) |
| M3 | **Saved and Journal** (FR-009–FR-014): save/unsave and the Saved list; replies and Journal screens | A saved lesson opens in 2 taps or fewer; replies persist (SC-004, SC-008) |
| M4 | **Safety and reminder** (FR-022–FR-024): crisis matcher on replies; support screen; help link; crisis test set in CI; daily reminder | Crisis wording always shows support; the reminder skips days with reading (SC-007) |
| M5 | **Polish and privacy**: swap in the reviewed lessons; accessibility pass; confirm no network calls (traffic inspection) | All quickstart scenarios pass (SC-001–SC-013, except store checks) |
| M6 | **Release**: see the checklist below | Builds submitted |

### Release checklist (M6)
- **Store listing**:
  - presented as learning the Māṇḍūkya Upaniṣad through Gauḍapāda;
  - makes no therapy or medical claims.
- **Apple App Store**:
  - privacy label: **Data Not Collected**;
  - age questionnaire answered honestly, then 18+ chosen.
- **Australian storefronts only**: check both stores exclude every other country (FR-029).
- **Privacy policy**: short, explaining that everything stays on the phone and that deleting the
  app deletes it.
- **Build**: with Expo SDK 58 or later (Xcode 26 / iOS 26 SDK).
- **Content**: every lesson reviewed (`content:check --release`), and every helpline verified in
  the last 90 days.

## Risks

| Risk | What we do |
|---|---|
| Lessons take longer than planned | Start on day one with mantra 7. If needed, launch with the 12 mantras and their key kārikās, and add the rest in updates. |
| No qualified reviewer | Line one up at M0, e.g. through Vedanta societies or teachers in Australia. |
| Copyright: meanings drift too close to modern commentaries | Writing guidance (lesson-file contract); reviewer checks; never paste from copyrighted translations. |
| Translation licensing | Start early and keep a public-domain fallback. |
| Sanskrit or pronunciation errors | Proofread before launch. Lessons are files, so fixes ship in the next update. |
| Missed crisis wording | Reviewed word lists, the crisis test set in CI, and help on every tab. |
| Too many false alarms | Acceptable by design; tune the idiom allow-list. |
| Notification fatigue | Reminder off by default, at most one a day, skipped on reading days. |

After launch: Ask (v2) and later plans are in [ROADMAP.md](./ROADMAP.md).
