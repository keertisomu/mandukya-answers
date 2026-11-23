---
description: "Task list for Turiya v1 — Learn the Māṇḍūkya"
---

# Tasks: Turiya v1 — Learn the Māṇḍūkya

**Input**: design documents in `specs/001-turiya-learn/`:
- [spec.md](./spec.md) (user stories, FR/SC);
- [plan.md](./plan.md) (stack, structure);
- [research.md](./research.md) (decisions R1–R10);
- [data-model.md](./data-model.md);
- [contracts/lesson-file.md](./contracts/lesson-file.md);
- [quickstart.md](./quickstart.md) (validation scenarios).

**Tests**: included only where the constitution or the plan needs them:
- content integrity (Principle III);
- crisis checking (Principle IV: "safety-gate logic MUST have explicit tests");
- the lesson-order rule and phone storage (data the person can't afford to lose);
- end-to-end flows matching the quickstart scenarios.

Write the safety tests before the code they test.

**Organization**: one phase per user story, in priority order. Lesson writing is people's work, so
it's a separate **content track** (Phase 8) that starts on day one and runs alongside everything
else.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: can run in parallel (different files, no dependency on an unfinished task).
- **[USn]**: the user story the task belongs to.

## Path conventions (from plan.md)

| Path | Holds |
|---|---|
| `apps/mobile/` | The Expo app. Screens in `app/` (Expo Router); code in `src/`; end-to-end flows in `e2e/` |
| `packages/shared/` | Schemas, content build, crisis word lists, support services. Tests in `packages/shared/test/` |
| `content/` | Lesson files (`content/path/mandukya/`) and the authors' guide |

---

## Phase 1: Setup (shared infrastructure)

**Purpose**: an empty monorepo that installs, typechecks, lints and tests.

- [ ] T001 Create the pnpm monorepo root with workspaces `apps/*` and `packages/*`.
  - Files: `package.json` (scripts `typecheck`, `lint`, `test`, `content:check`, `test:crisis`, `e2e`), `pnpm-workspace.yaml`, `tsconfig.base.json` (strict) and `.nvmrc` (Node 24).
- [ ] T002 Scaffold the Expo app in `apps/mobile/` with Expo SDK 58 (or the latest stable), TypeScript and Expo Router.
  - Use the tabs template, trimmed to nothing.
  - Add `react-native-mmkv`, `expo-notifications` and `@expo-google-fonts/{newsreader,karla,noto-serif-devanagari}`.
  - Do **not** add `expo-updates` or any analytics or crash SDK (research R8).
- [ ] T003 [P] Scaffold `packages/shared/` as a TypeScript library with Zod, `gray-matter` and Vitest.
  - Files: `packages/shared/package.json`, `packages/shared/tsconfig.json`, `packages/shared/vitest.config.ts`, `packages/shared/src/index.ts`.
- [ ] T004 [P] Configure ESLint and Prettier for the whole repo in `eslint.config.mjs` and `.prettierrc`, with `lint` and `format` scripts in the root `package.json`.
- [ ] T005 [P] Configure Jest with `jest-expo` and React Native Testing Library in `apps/mobile/jest.config.js`, with an MMKV mock in `apps/mobile/test/setup.ts`.
- [ ] T006 [P] Configure an Expo development build in `apps/mobile/eas.json` (profiles `development`, `preview`, `production`) and `apps/mobile/app.json`.
  - Set the app name "Turiya", bundle IDs and `newArchEnabled`.
  - Leave out `updates`.
- [ ] T007 [P] Add CI in `.github/workflows/ci.yml`: on push and PR, run `pnpm install`, `typecheck`, `lint`, `test`, `content:check` and `test:crisis` on Node 24.

**Checkpoint**: `pnpm install && pnpm typecheck && pnpm lint && pnpm test` pass on an empty project.

---

## Phase 2: Foundational (blocks every user story)

**Purpose**: the lesson schema and content build, storage, theme and navigation shell that every
story uses.

**⚠️ No user story work starts until this phase is done.**

### Content schema and build (contracts/lesson-file.md, data-model §1)

- [ ] T008 [P] Write failing tests for single-file validation in `packages/shared/test/content/schema.test.ts`. Cover:
  - a valid mantra 7 part 1 file (from the contract example);
  - a missing `pronunciation`;
  - `karikas` or `ending` on a non-last part;
  - `featured` missing;
  - a bad `mantra` number.
- [ ] T009 [P] Write failing tests for whole-Path validation in `packages/shared/test/content/validate.test.ts`. Cover:
  - a gap in `order`;
  - `part.n` out of range;
  - two endings for one mantra;
  - a kārikā appearing twice;
  - a mantra with kārikās but none `featured`;
  - in **release** mode: mantras 1–12 not all present, any of Kārikā 1.1–1.29 missing, or `reviewedBy` missing.
- [ ] T010 Implement the Zod schemas `Phrase`, `Karika`, `Ending` and `Lesson` in `packages/shared/src/content/schema.ts`, exactly as in data-model §1, so that T008 passes.
- [ ] T011 Implement `validatePath(lessons, { release })` in `packages/shared/src/content/validate.ts`, with the data-model §1 build checks, so that T009 passes.
  - **Development mode**: structure and order.
  - **Release mode**: also completeness (12 mantras, all 29 kārikās) and review fields.
- [ ] T012 Implement the content build CLI in `packages/shared/scripts/build-content.ts`:
  - read `content/path/mandukya/*.md` with `gray-matter`;
  - validate with T010 and T011;
  - turn `note`, `summary` and any Markdown body into simple text blocks (`{ type: "p", spans: [{ text, em? }] }`);
  - write `apps/mobile/src/content/generated/path.json` with a `contentVersion` (content hash);
  - wire up `pnpm content:check`, which validates only, and `pnpm content:check --release`.
- [ ] T013 Add development-only draft lessons in `content/path/mandukya/`, so the app can be built before real content arrives.
  - Files: `content/path/mandukya/01-mantra-1.md`, `content/path/mandukya/07-mantra-7-not-the-knower.md`, `content/path/mandukya/08-mantra-7-not-the-known.md`, `content/path/mandukya/09-mantra-7-what-it-is.md`.
  - Use the contract example. Mark every `note`, `meaning` and `summary` as `"DRAFT — not reviewed"`, and give no `reviewedBy`, so a release build can never ship them.

### App foundations

- [ ] T014 [P] Implement the storage wrapper in `apps/mobile/src/storage/storage.ts`:
  - typed `read(key, fallback)` and `write(key, value)` over MMKV, for the keys `turiya_progress_v1`, `turiya_saved_v1`, `turiya_journal_v1` and `turiya_settings_v1`;
  - every call wrapped in try/catch, returning the fallback on failure (data-model §2).
- [ ] T015 [P] Write tests for the storage wrapper in `apps/mobile/test/storage.test.ts`: round trip, a corrupt value returns the fallback, and a throwing MMKV returns the fallback.
- [ ] T016 [P] Encode the design tokens in `apps/mobile/src/theme/tokens.ts`: colours, radii, spacing and type scale from `design/README.md` → "Design Tokens".
- [ ] T017 [P] Load the bundled fonts (Newsreader, Karla, Noto Serif Devanagari) in `apps/mobile/src/theme/fonts.ts`, with no network font loading.
- [ ] T018 [P] Add `useReducedMotion()` in `apps/mobile/src/theme/motion.ts`. It returns the OS setting, and is used to turn off the `breathe`, `fadeup` and `driftglow` animations (FR-019).
- [ ] T019 [P] Create the base components in `apps/mobile/src/components/`: `Screen.tsx`, `Card.tsx`, `Button.tsx` (minimum 44px target, FR-018), `SanskritText.tsx` (Noto Serif Devanagari) and `Eyebrow.tsx`.
- [ ] T020 Create the app shell in `apps/mobile/app/`, with the tabs labelled in Australian English (FR-021):
  - root layout in `apps/mobile/app/_layout.tsx`: load fonts, hide the splash screen;
  - two-tab layout in `apps/mobile/app/(tabs)/_layout.tsx`: Learn, then Journal; the app opens to Learn (FR-015);
  - placeholder screens `apps/mobile/app/(tabs)/learn.tsx` and `apps/mobile/app/(tabs)/journal.tsx`.
- [ ] T021 [P] Implement the content access module in `apps/mobile/src/content/path.ts`, which loads `generated/path.json`. It provides `getLessons()`, `getLesson(id)`, `getLessonByOrder(n)` and `lessonCount()`.

**Checkpoint**: `pnpm content:check` passes on the draft lessons; the app boots offline to an empty
Learn tab; storage tests pass.

---

## Phase 3: User Story 1 — Learn a mantra phrase by phrase, with Gauḍapāda's help (P1) 🎯 MVP

**Goal**: a person can read lessons in order, offline. Each lesson has phrases, the Gauḍapāda
section (featured kārikās plus "More from Gauḍapāda"), then the full mantra, summary and prompt.
They can continue or stop, and see their progress.

**Independent Test**: quickstart scenarios 1–6. Open lesson 1 offline, read every phrase, check the
Gauḍapāda section and the mantra ending, continue to lesson 2, and see "2 of N". Mantra 7 spans 3
lessons.

### Tests for User Story 1

- [ ] T022 [P] [US1] Write failing tests for the order rule in `apps/mobile/test/progress.test.ts`. Cover:
  - lesson 1 available at first;
  - only the next lesson becomes available after completing one;
  - no skipping ahead;
  - completed lessons stay re-readable;
  - finishing the last lesson marks the Path finished;
  - restarting clears `completed` but not Saved or the Journal (data-model §3).

### Implementation for User Story 1

- [ ] T023 [US1] Implement the progress store in `apps/mobile/src/features/learn/progress.ts`, on the T014 storage, so that T022 passes.
  - Functions: `getStatus(lessonId)` (`locked`, `available` or `completed`), `complete(lessonId)` (also sets `lastReadOn` to today's local date), `nextLesson()`, `isFinished()` and `restart()`.
- [ ] T024 [P] [US1] Build `PhraseCard` in `apps/mobile/src/features/learn/components/PhraseCard.tsx`. It shows Devanagari, IAST, pronunciation, meaning and note (FR-002).
- [ ] T025 [P] [US1] Build `PartHeading` in `apps/mobile/src/features/learn/components/PartHeading.tsx`. It shows "Part n of m · heading" (FR-002, FR-003).
- [ ] T026 [P] [US1] Build `GaudapadaSection` in `apps/mobile/src/features/learn/components/GaudapadaSection.tsx` (FR-004).
  - A "What Gauḍapāda says" heading.
  - `featured` kārikās as cards: Devanagari, IAST, pronunciation, translation, source, note.
  - A "More from Gauḍapāda" expander, collapsed by default, holding the rest.
  - The section is left out when there are no kārikās.
- [ ] T027 [P] [US1] Build `MantraEnding` in `apps/mobile/src/features/learn/components/MantraEnding.tsx` (FR-005): the full mantra (all fields and source), the summary, the related state with its tint, and the prompt text.
- [ ] T028 [US1] Build the lesson screen in `apps/mobile/app/lesson/[id].tsx`, using T024–T027:
  - the title, then phrases in order under a part heading where there is one;
  - on the mantra's last part: `GaudapadaSection` then `MantraEnding`;
  - a **Finish** button that calls `complete()`, then offers "Next lesson" or shows "The next lesson is ready whenever you are." (FR-007);
  - locked lessons can't be opened.
- [ ] T029 [US1] Build the Learn home in `apps/mobile/app/(tabs)/learn.tsx`:
  - a "Continue" card for `nextLesson()`;
  - progress "Lesson n of N" (FR-008);
  - a list of completed lessons that can be re-read;
  - no missed-day messages.
- [ ] T030 [P] [US1] Build `PathComplete` in `apps/mobile/src/features/learn/components/PathComplete.tsx`: a gentle completion moment with "Read again from the start" (calls `restart()`) and re-read access. Show it on the Learn home when `isFinished()`.
- [ ] T031 [US1] Add a Maestro flow in `apps/mobile/e2e/learn.yaml` for quickstart scenarios 1–6 (airplane mode, phrases, Gauḍapāda section, mantra 7 parts, continue, re-open).

**Checkpoint**: US1 works end to end offline with the draft lessons. This is the MVP demo.

---

## Phase 4: User Story 2 — Return to the mantras that matter (P1)

**Goal**: save and unsave lessons; Saved sits at the top of Learn; any saved lesson opens in two
taps or fewer.

**Independent Test**: quickstart scenarios 7–8. Save a mantra 7 lesson, restart, and open it from
Saved in two taps or fewer; then unsave it.

- [ ] T032 [P] [US2] Write tests for the saved store in `apps/mobile/test/saved.test.ts`. Cover:
  - save, then unsave;
  - newest first;
  - no duplicates;
  - can't save a locked lesson;
  - survives a reload of storage.
- [ ] T033 [US2] Implement the saved store in `apps/mobile/src/features/saved/saved.ts` (`isSaved`, `toggle(lessonId)`, `list()`), allowing only completed lessons or the one being read (FR-009), so that T032 passes.
- [ ] T034 [P] [US2] Build `SaveButton` in `apps/mobile/src/features/saved/SaveButton.tsx`: a "Save" / "Saved" toggle, with an accessibility label and a 44px target.
- [ ] T035 [P] [US2] Build `SavedList` in `apps/mobile/src/features/saved/SavedList.tsx`: saved lessons with title and mantra, each opening the lesson in one tap.
- [ ] T036 [US2] Add `SaveButton` to the lesson screen header in `apps/mobile/app/lesson/[id].tsx`, and `SavedList` to the top of `apps/mobile/app/(tabs)/learn.tsx`, so a saved lesson is reachable in two taps or fewer (FR-010, SC-004).
- [ ] T037 [US2] Add a Maestro flow in `apps/mobile/e2e/saved.yaml` for quickstart scenarios 7–8.

**Checkpoint**: US1 and US2 both work on their own.

---

## Phase 5: User Story 3 — Keep a private journal of reflections (P1)

**Goal**: an optional reply to each lesson's prompt, kept on the phone and listed in a read-only
Journal.

**Independent Test**: quickstart scenarios 9–11. Write a reply, find it in the Journal with title
and time, open it, follow the link to the lesson, edit it from the lesson, restart, and it's still
there.

- [ ] T038 [P] [US3] Write tests for the journal store in `apps/mobile/test/journal.test.ts`. Cover:
  - at most one entry per lesson (upsert);
  - an empty reply saves nothing;
  - editing changes `updatedAt`;
  - newest first by `updatedAt`;
  - survives a reload.
- [ ] T039 [P] [US3] Write tests for relative times in `apps/mobile/test/relativeTime.test.ts`: "Today · h:mm", "Yesterday · h:mm", "N days ago" within 7 days, then "24 Oct" style, in an Australian locale (FR-020).
- [ ] T040 [US3] Implement the journal store in `apps/mobile/src/features/journal/journal.ts` (`getForLesson`, `saveReply(lessonId, text)`, `list()`) with `kind: "lesson"` entries (data-model §2), so that T038 passes.
- [ ] T041 [P] [US3] Implement `formatRelativeTime(date, now)` in `apps/mobile/src/lib/relativeTime.ts`, so that T039 passes.
- [ ] T042 [US3] Build `ReplyInput` in `apps/mobile/src/features/journal/ReplyInput.tsx`: a multi-line input under the prompt in `MantraEnding`, prefilled with any existing reply, with a "Save to journal" action that calls `saveReply` (FR-011). Add it to `apps/mobile/src/features/learn/components/MantraEnding.tsx`.
- [ ] T043 [US3] Build the Journal list in `apps/mobile/app/(tabs)/journal.tsx` (FR-012):
  - entries newest first, with relative time and lesson title, looked up from content, not stored;
  - the empty state "Nothing here yet. Begin a lesson, and what you write will be kept here."
- [ ] T044 [US3] Build the journal entry screen in `apps/mobile/app/journal/[id].tsx` (FR-013, FR-014): read-only, showing the reply, the lesson's prompt and an "Open lesson" link. There are no edit controls on this screen.
- [ ] T045 [US3] Add a Maestro flow in `apps/mobile/e2e/journal.yaml` for quickstart scenarios 9–11.

**Checkpoint**: US1–US3 work on their own.

---

## Phase 6: User Story 4 — Feel safe, private and never harmed (P1)

**Goal**: a crisis check on journal replies (on the phone), support services one tap away, privacy
copy, and proof that the app sends nothing. **Required for release.**

**Independent Test**: quickstart scenarios 12, 13, 16 and 17. Crisis wording in a reply (offline)
shows the support screen and keeps the reply; help is reachable from both tabs; traffic inspection
shows no requests.

### Tests for User Story 4 (write first — constitution, Principle IV)

- [ ] T046 [P] [US4] Write the crisis test set in `packages/shared/test/safety/crisis-cases.json`: 40 or more cases labelled `crisis: true|false`.
  - Include direct wording, indirect wording ("I don't see the point anymore"), negation ("I'm not going to hurt myself", which still shows support), and everyday idioms ("this job is killing me", "I could kill for a coffee", which are not crisis).
  - Mark the file "draft — to be reviewed with someone experienced in crisis support".
- [ ] T047 [P] [US4] Write failing tests in `packages/shared/test/safety/match.test.ts`, run by `pnpm test:crisis`:
  - every `crisis: true` case MUST match — any miss fails the build;
  - false alarms on `crisis: false` cases are reported but don't fail (research R7).
- [ ] T048 [P] [US4] Write a test in `packages/shared/test/safety/support-services.test.ts`:
  - the AU list includes 000, Lifeline, a youth service and an anxiety/depression service;
  - every entry has contact details and hours;
  - it fails if any `lastVerified` is more than 90 days old (FR-030, FR-031).

### Implementation for User Story 4

- [ ] T049 [US4] Write the crisis word and phrase lists in `packages/shared/src/safety/crisis-lexicon.ts`: phrases, an idiom allow-list and negation cues, with a header noting expert review is pending.
- [ ] T050 [US4] Implement `matchesCrisis(text)` in `packages/shared/src/safety/match.ts`, leaning toward a match when unsure, so that T047 passes.
- [ ] T051 [P] [US4] Add the AU support services in `packages/shared/src/safety/support-services.ts`, so that T048 passes:
  - 000; Lifeline 13 11 14, also text and chat; Kids Helpline 1800 55 1800; Beyond Blue 1300 22 4636;
  - an overseas entry for findahelpline.com;
  - each with `lastVerified`, keyed by region `AU`.
- [ ] T052 [US4] Build the support screen in `apps/mobile/app/support.tsx`:
  - a warm, brief heading;
  - each service with tap-to-call or text, plus hours;
  - a note that 000 and Australian lines work only in Australia, with the overseas link;
  - no verse and no lesson content; works offline.
- [ ] T053 [P] [US4] Build `HelpLink` ("Need to talk to someone?") in `apps/mobile/src/features/safety/HelpLink.tsx`. It opens `/support`. Add it to the headers of `apps/mobile/app/(tabs)/learn.tsx` and `apps/mobile/app/(tabs)/journal.tsx` (FR-024).
- [ ] T054 [US4] Call `matchesCrisis` when a reply is saved, in `apps/mobile/src/features/journal/ReplyInput.tsx`. On a match, keep the reply and open `/support` (FR-023).
- [ ] T055 [US4] Build the settings screen in `apps/mobile/app/settings.tsx`, reached from the Learn header, with an "About and privacy" section:
  - teaching the Upaniṣad with the help of Gauḍapāda's Kārikā, not therapy or religious authority (FR-026);
  - "Everything is saved only on this phone. Deleting the app deletes it." (FR-028);
  - translation sources and credits.
- [ ] T056 [P] [US4] Add a no-network guard in `scripts/check-no-network.mjs`, run in CI:
  - fail if `apps/mobile/package.json` depends on `expo-updates` or any analytics or crash SDK;
  - fail if `apps/mobile/src` or `apps/mobile/app` calls `fetch(`, `XMLHttpRequest` or `WebSocket` (FR-027).
- [ ] T057 [US4] Add a Maestro flow in `apps/mobile/e2e/safety.yaml` for quickstart scenarios 12–13. Document the manual traffic inspection (scenario 16) in `apps/mobile/e2e/README.md`.

**Checkpoint**: all four P1 stories work. v1 is functionally releasable once content is reviewed.

---

## Phase 7: User Story 5 — Get a gentle daily reminder (P3)

**Goal**: an optional daily reminder, off by default, skipped on days a lesson was read.

**Independent Test**: quickstart scenarios 14–15.

- [ ] T058 [P] [US5] Write tests for the scheduler in `apps/mobile/test/reminders.test.ts`, with `expo-notifications` mocked:
  - turning on schedules the next 7 days at HH:mm;
  - reading a lesson today cancels today's reminder;
  - opening the app tops up to 7 days;
  - turning off cancels everything;
  - the reminder text contains no journal content.
- [ ] T059 [US5] Implement the scheduler in `apps/mobile/src/features/reminders/scheduler.ts` (`enable(time)`, `disable()`, `onLessonRead()`, `topUp()`), with the generic text "A quiet moment to read?" (research R6), so that T058 passes.
- [ ] T060 [US5] Add a reminder toggle and time picker to `apps/mobile/app/settings.tsx`. Ask for notification permission only when the toggle is turned on, and save to `turiya_settings_v1` (FR-022).
- [ ] T061 [US5] Wire up the reminder calls in `apps/mobile/app/_layout.tsx` and `apps/mobile/app/lesson/[id].tsx`:
  - `topUp()` on app foreground, in `apps/mobile/app/_layout.tsx`;
  - `onLessonRead()` after `complete()`, in `apps/mobile/app/lesson/[id].tsx`;
  - tapping a notification opens the Learn tab.

**Checkpoint**: all user stories work.

---

## Phase 8: Content track (people — starts with Phase 1, gates release)

**Purpose**: the real lessons. These are done by authors, a Sanskrit proofreader and a reviewer
grounded in the tradition, through pull requests to `content/`. The tasks can run in parallel with
Phases 1–7.

- [ ] T062 [P] Write the authors' guide in `content/README.md`:
  - the pronunciation scheme (research R5: hyphenated syllables, doubled long vowels, *h* for aspirates, *jña* → *gya*) with 10 worked examples;
  - the writing guidance from `contracts/lesson-file.md`.
- [ ] T063 [P] Choose the translations of the 12 mantras and of Kārikā chapter 1, confirm public-domain status or get permission, and record each source and licence in `content/SOURCES.md` (DECISIONS open question).
- [ ] T064 Write mantra 7 first in `content/path/mandukya/07-…09-*.md`, replacing the T013 drafts. Do the Gauḍapāda section first, since it's the hardest layout.
  - `content/path/mandukya/07-mantra-7-not-the-knower.md`
  - `content/path/mandukya/08-mantra-7-not-the-known.md`
  - `content/path/mandukya/09-mantra-7-what-it-is.md`, with kārikās 1.10–1.18 (1–2 featured) and the ending.
- [ ] T065 Write mantras 1–6 in `content/path/mandukya/01-mantra-1.md` … `06-mantra-6.md`. Kārikās 1.1–1.9 go with mantra 6.
- [ ] T066 Write mantras 8–12 in `content/path/mandukya/10-mantra-8.md` … `14-mantra-12.md`. Kārikās 1.19–1.23 go with mantra 11, and 1.24–1.29 with mantra 12.
- [ ] T067 Proofread every Devanagari, IAST and pronunciation field against a Sanskrit source, in all `content/path/mandukya/*.md` (SC-003).
- [ ] T068 Tradition review of every meaning, note, kārikā note, summary and prompt. Set `reviewedBy` and `reviewedOn` in each `content/path/mandukya/*.md`; `pnpm content:check --release` must pass.
- [ ] T069 [P] Review the crisis lists and test set with someone experienced in crisis support. Update `packages/shared/src/safety/crisis-lexicon.ts` and `packages/shared/test/safety/crisis-cases.json`, and remove the "pending review" notes.
- [ ] T070 [P] Verify every support service (number, hours, region) and update `lastVerified` in `packages/shared/src/safety/support-services.ts`. Repeat every 90 days (FR-031).

---

## Phase 9: Polish and release

**Purpose**: quality across all stories, and the release checklist (docs/IMPLEMENTATION_PLAN.md →
M6).

- [ ] T071 [P] Accessibility pass across `apps/mobile/app/` and `apps/mobile/src/`:
  - every touch target is 44px or more;
  - screen-reader labels for Devanagari (read the IAST and meaning);
  - Dynamic Type works;
  - reduced motion is respected.
- [ ] T072 [P] Performance check: cold start to Learn in under 2 seconds and a lesson in under 1 second on a mid-range Android phone and an iPhone. Record the results in `apps/mobile/e2e/README.md`.
- [ ] T073 [P] Write the privacy policy in `docs/PRIVACY_POLICY.md`: nothing is collected; everything stays on the phone; deleting the app deletes it; no analytics.
- [ ] T074 [P] Write the store listing copy in `docs/STORE_LISTING.md`:
  - learning the Upaniṣad with Gauḍapāda's Kārikā;
  - no therapy or medical claims;
  - privacy label **Data Not Collected**;
  - age rating chosen as 18+;
  - Australia only.
- [ ] T075 Swap in the reviewed lessons in `content/path/mandukya/`: remove all T013 drafts, run `pnpm content:check --release`, and rebuild.
- [ ] T076 Run every quickstart scenario (1–17) on iOS and Android, including traffic inspection (scenario 16) and another locale (scenario 17). Record the results in `apps/mobile/e2e/README.md`.
- [ ] T077 Build production releases with EAS, limit both stores to **Australia**, and submit (SC-013). Follow `docs/IMPLEMENTATION_PLAN.md` → "Release checklist".

---

## Dependencies and execution order

### Phases
- **Setup (1)**: no dependencies.
- **Foundational (2)**: after Setup. **Blocks all stories.**
- **US1 (3)**: after Foundational. The MVP.
- **US2 (4)**: after Foundational. It uses the lesson screen and Learn home from US1 (T028, T029).
- **US3 (5)**: after Foundational. It adds the reply input to `MantraEnding` (T027).
- **US4 (6)**:
  - The support screen, help link, settings and no-network guard (T046–T053, T055–T056) need only Foundational.
  - The reply crisis hook (T054) needs US3 (T042).
- **US5 (7)**: after Foundational. Its settings toggle needs the settings screen (T055).
- **Content track (8)**: starts with Setup. T064–T068 need the schema (T010–T012) for `content:check`.
- **Polish (9)**: after US1–US4 (US5 is optional), and T075 after T068.

### Within each story
Tests first (they should fail), then stores and logic, then components, then screens, then the
Maestro flow.

### Release gate
US1–US4 done, plus T064–T070 (reviewed content, crisis review, verified helplines), plus T071–T077.
US5 may ship in v1 or follow in an update.

---

## Parallel examples

```text
# Phase 2, after T010/T011 are started:
T014 storage wrapper        T016 tokens        T017 fonts
T018 reduced motion         T019 base components    T021 content access

# US1 components, together:
T024 PhraseCard    T025 PartHeading    T026 GaudapadaSection    T027 MantraEnding

# US4 tests, together:
T046 crisis test set    T047 matcher tests    T048 support-services test

# Content track, alongside all coding:
T062 authors' guide    T063 translations    T069 crisis review    T070 helplines
```

---

## Implementation strategy

### MVP first (User Story 1)
1. Phase 1, then Phase 2.
2. Phase 3 (US1) with the draft mantra 7 lessons.
3. **Stop and validate** against quickstart scenarios 1–6. Demo mantra 7 end to end.

### Incremental delivery
1. Add US2 (Saved), then demo the morning re-read of mantra 7.
2. Add US3 (Journal), then US4 (Safety). US1–US4 together are releasable once content is reviewed.
3. Add US5 (Reminder) for v1, or in the first update.

### Content in parallel
The content track decides the launch date, not the code. Start T062–T064 on day one. Mantra 7 then
doubles as the app's real test content.

---

## Notes

- **[P]** = different files and no unfinished dependencies.
- **Drafts never ship**: lessons marked "DRAFT" have no `reviewedBy`, so the release build fails
  while they remain.
- **Commit** after each task or logical group, and stop at any checkpoint to validate.
- **Ask (v2)** has no tasks here; its design is in `docs/ASK_V2_DESIGN.md`.
