# Quickstart: run and validate Turiya v1

How to run v1 locally and prove it meets the spec. Commands refer to scripts created in M0 (see
[plan.md](./plan.md)); they don't exist yet. v1 has **no server**, so there's nothing to run besides
the app.

## Prerequisites

- **Node.js** 24 LTS and **pnpm**.
- **Xcode 26** (iOS simulator) and/or **Android Studio** (emulator).
- **An Expo development build** of the app. MMKV doesn't run in Expo Go (research R1).
- **Maestro** for the end-to-end flows.

## Setup

```bash
pnpm install
pnpm content:check            # every lesson file is valid; all 12 mantras and 29 kārikās present
pnpm --filter mobile ios      # or: android
```

## Automated checks

```bash
pnpm test                     # schemas, unlock rule, crisis matcher, screens (Vitest, Jest)
pnpm test:crisis              # crisis test set against the word lists
pnpm e2e                      # Maestro flows below
pnpm content:check --release  # also fails if any lesson lacks reviewedBy/reviewedOn
```

## Validation scenarios

Details are in [data-model.md](./data-model.md) and
[contracts/lesson-file.md](./contracts/lesson-file.md).

| # | Do this | Expect | Proves |
|---|---|---|---|
| 1 | Fresh install, **airplane mode on**, open the app | Lesson 1 opens | FR-001, FR-015, FR-017, SC-009 |
| 2 | Read lesson 1 | Each phrase shows Devanagari, IAST, pronunciation, meaning and note, in order | FR-001, FR-002 |
| 3 | Reach the end of a mantra's last lesson | "What Gauḍapāda says" (where it has kārikās), then the full mantra with translation and source, summary, related state, prompt | FR-004, FR-005 |
| 4 | Read mantra 7 | It spans several lessons with part headings. The last part features one kārikā (1.10); "More from Gauḍapāda" shows the other eight one at a time; the full mantra comes at the end | FR-002–FR-004 |
| 5 | Finish a lesson, tap "next" | The next lesson opens straight away; the one after is locked; progress updates | FR-007, FR-008 |
| 6 | Finish a lesson, close the app, reopen days later | Next lesson ready; no missed-day message | FR-007 |
| 7 | Save a lesson, force-quit, reopen | It's in Saved at the top of Learn; opens in 2 taps or fewer | FR-009, FR-010, SC-004 |
| 8 | Unsave it | It leaves Saved | FR-009 |
| 9 | Write a reply to a prompt, open the Journal | Entry shows with lesson title and time; opens in full; links to the lesson | FR-011–FR-013 |
| 10 | Edit the reply from the lesson | The Journal shows the new version; it can't be edited from the Journal | FR-014 |
| 11 | Force-quit and reopen | Progress, Saved and the Journal unchanged | FR-016, SC-008 |
| 12 | Write a crisis phrase as a reply (offline), with the phone set to Australia, then to India | Reply kept; mental health support opens straight away: 000 and Lifeline, then 112 and Tele-MANAS | FR-023, FR-030, SC-007 |
| 13 | Tap **Support** in the header on each screen (offline) | Menu with Mental health support and Give feedback; the services list works offline and can switch Australia ⇄ India | FR-024, FR-030 |
| 13b | Support → Give feedback → Open email | The email app opens with address, subject, app version, device and lesson; no journal text; nothing sent by the app | FR-032 |
| 14 | Turn the reminder on for 2 minutes from now; read nothing | A reminder with generic text arrives | FR-022 |
| 15 | Read a lesson, then wait for the reminder time | No reminder that day | FR-022 |
| 16 | Inspect outbound traffic (e.g. Proxyman) across scenarios 1–15 | No requests from the app | FR-027, SC-010 |
| 17 | Set the device to a region other than Australia or India | App stays in English; mental health support asks once which list to show, and shows the international directory | FR-021, FR-030 |

## Before release (manual)

- **Content**: every lesson has `reviewedBy` and `reviewedOn` (SC-003).
- **Support services**: every one has `lastVerified` within the last 90 days (SC-012).
- **Stores**: both list the app in Australia and India only (SC-013). The App Store privacy label is "Data Not
  Collected".
- **First-use testing** with 5–8 people: SC-001, SC-002, SC-006 and SC-011.
