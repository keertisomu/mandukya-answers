# Architecture — v1

How v1 is built. **What** it does is in the [spec](../specs/001-turiya-learn/spec.md) (`FR-`/`SC-`
IDs); **why** each choice was made is in [DECISIONS.md](./DECISIONS.md) (`DEC-` IDs). The detailed
design is in the spec folder: [plan](../specs/001-turiya-learn/plan.md),
[data model](../specs/001-turiya-learn/data-model.md),
[lesson file format](../specs/001-turiya-learn/contracts/lesson-file.md).

## 1. The big picture

```
Phone (React Native app) — no server, no network
  ├─ Learn ...... ~32 lessons, bundled in the app at build time
  │               saved lessons at the top
  ├─ Journal .... replies to lesson prompts
  └─ Storage .... progress, saved, journal, settings — on the phone only (MMKV)

content/ (reviewed lesson files) ──build check──▶ bundled into the app
```

Three rules shape everything:

- **Nothing leaves the phone.** No server, no accounts, no analytics, no over-the-air updates
  (DEC-1, DEC-12).
- **No generated text.** Every lesson is written, proofread and reviewed before release.
- **Everything works offline**, including fonts and the support screen.

Ask, the AI reflection feature, comes in v2 and adds a server. Its design is in
[ASK_V2_DESIGN.md](./ASK_V2_DESIGN.md).

## 2. The app (React Native + Expo, DEC-2)

### Screens
- **Tabs**: Learn (opens first) and Journal.
- **Learn home**: saved lessons at the top, then "continue" (the next lesson) and progress.
- **Lesson**: phrases one after another, under part headings where a verse has parts. A verse's
  last part ends with the full verse, summary and prompt.
- **Other screens**: a journal entry, the support screen and settings (the reminder).
- **Designs**: the visual style (colours, type, calm feel) comes from the handoff in `design/`. The
  Learn screens, lesson layout, Saved list and settings need new designs in that style.

### Data on the phone (MMKV, DEC-3)

| Key | Holds |
|---|---|
| `turiya_progress_v1` | Completed lessons |
| `turiya_saved_v1` | Saved lessons |
| `turiya_journal_v1` | Replies to lesson prompts |
| `turiya_settings_v1` | Reminder on/off and time |

Every read and write is wrapped, so the app still works if storage fails. Shapes and state changes
are in the [data model](../specs/001-turiya-learn/data-model.md).

### Lessons
- **Source**: one Markdown file per lesson in `content/path/karika-ch1/`, in the
  [lesson file format](../specs/001-turiya-learn/contracts/lesson-file.md).
- **Phrase by phrase**: each phrase has Devanagari, IAST, an easy pronunciation guide, a meaning
  and a short note.
  - Long verses (such as mantra 7) span several lessons, one per part.
  - A verse's last part adds the full verse, summary, related state and prompt.
- **Checked at build time**: a Zod schema in `packages/shared` rejects missing fields, gaps in
  order or a missing ending. A release build also fails if any lesson isn't reviewed.
- **Bundled**: notes and summaries become simple text blocks, rendered with the app's own
  components.
- **Canonical text and our commentary stay in separate fields** (constitution, Principle III).
- **Pronunciation guide**: one respelling scheme, documented in `content/README.md`, e.g.
  *naan-tah-prag-yam*.
- **Order rule**: the next lesson opens as soon as the previous one is done. No daily limit, no
  streaks.
- **Changes** go through pull requests, reviewed for accuracy.

### Saved lessons
A list of lesson IDs on the phone, newest first, shown at the top of Learn. A saved lesson opens in
two taps or fewer.

### Support screen and helplines
- **How people reach it**:
  - the crisis check on a journal reply;
  - the "Need to talk to someone?" link on both tabs.
- **Bundled in the app**, so it works offline. Stored per region (`AU` in v1), so a new country adds
  data, not screens.
- **Verify every number before launch and every 90 days after.**

  | Service | Contact |
  |---|---|
  | Emergency | 000 |
  | Lifeline | 13 11 14 (also text and chat) |
  | Kids Helpline | 1800 55 1800 |
  | Beyond Blue | 1300 22 4636 |
  | Overseas | findahelpline.com |

### Crisis check on journal replies
- **Runs on the phone**, matching word and phrase lists from `packages/shared`.
- **Handles** negation and common idioms ("this job is killing me").
- **Leans toward showing help** when unsure.
- **On a match**, the reply is still saved and the support screen opens.
- **Tested** by a crisis test set in CI. In v2, the same lists become Ask's first gate.

### Daily reminder
- **Local notifications** (`expo-notifications`) for the next 7 days at the chosen time, with no
  push server.
- **Today's reminder is cancelled** once a lesson is read; the week is topped up each time the app
  opens.
- **Off by default.** Permission is asked for only when the person turns it on.
- **Generic text** only, e.g. "A quiet moment to read?"

## 3. Everything else

- **No network code.** No analytics, crash-reporting or `expo-updates`. Crash reports come only from
  App Store Connect and Play Console. The App Store privacy label is **"Data Not Collected"**.
- **Fonts**: Newsreader, Karla and Noto Serif Devanagari are bundled, so Sanskrit renders offline.
- **Versions**: storage keys end in `_v1`, and lesson IDs stay stable across content fixes.
- **Repo layout**:

  | Folder | Holds |
  |---|---|
  | `apps/mobile` | The React Native app |
  | `packages/shared` | Lesson schema, crisis word lists, support services, content build |
  | `content/` | Reviewed lesson files and the pronunciation scheme |

- **Built to extend**:
  - more Paths (chapters 2–4) are new `content/path/` folders;
  - new regions are new helpline data;
  - v2 adds `apps/api`, `db/` and `evals/` for Ask, without moving anything.
