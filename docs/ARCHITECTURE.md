# Architecture — v1

How v1 is built. **What** it does is in the [spec](../specs/001-turiya-learn/spec.md) (`FR-`/`SC-`
IDs); **why** each choice was made is in [DECISIONS.md](./DECISIONS.md) (`DEC-` IDs). The detailed
design is in the spec folder: [plan](../specs/001-turiya-learn/plan.md),
[data model](../specs/001-turiya-learn/data-model.md),
[lesson file format](../specs/001-turiya-learn/contracts/lesson-file.md).

## 1. The big picture

```
Phone (React Native app) — no server, no network
  ├─ Learn ...... ~15 lessons on the 12 mantras, with Gauḍapāda's kārikās,
  │               bundled in the app at build time
  │               saved lessons at the top
  ├─ Journal .... replies to lesson prompts
  └─ Storage .... progress, saved, journal, settings — on the phone only (MMKV)

content/ (reviewed lesson files) ──build check──▶ bundled into the app
```

Three rules shape everything:

- **Nothing leaves the phone.** No server, no accounts, no analytics, no over-the-air updates
  (DEC-1, DEC-12).
- **No generated text.** Every lesson is written, proofread and reviewed before release.
- **Everything works offline**, including fonts and mental health support.

Ask, the AI reflection feature, comes in v2 and adds a server. Its design is in
[ASK_V2_DESIGN.md](./ASK_V2_DESIGN.md).

## 2. The app (React Native + Expo, DEC-2)

### Screens
- **Tabs**: Learn (opens first) and Journal.
- **Learn home**: saved lessons at the top, then "continue" (the next lesson) and progress.
- **Lesson**: the mantra's phrases one after another, under part headings where it has parts. A
  mantra's last part then shows "What Gauḍapāda says" (one featured kārikā; the rest under an
  expandable "More from Gauḍapāda", one at a time), the full mantra, summary and prompt.
- **Other screens**: a journal entry, the Support sheet and settings (the reminder).
- **Header**: every screen has a small, muted **Support** item next to its other actions.
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
- **Source**: one Markdown file per lesson in `content/path/mandukya/`, in the
  [lesson file format](../specs/001-turiya-learn/contracts/lesson-file.md).
- **Phrase by phrase**: each phrase has Devanagari, IAST, an easy pronunciation guide, a meaning
  and a short note.
  - Long mantras (such as mantra 7) span several lessons, one per part.
  - A mantra's last part adds Gauḍapāda's kārikās on it, then the full mantra, summary, related
    state and prompt.
- **Checked at build time**: a Zod schema in `packages/shared` rejects missing fields, gaps in
  order, a missing ending, or a kārikā (1.1–1.29) that is missing or appears twice. A release build also fails if any lesson isn't reviewed.
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

### Support sheet
Opened from the **Support** item in every header. Two options:

**1. Mental health support**
- Also opens **directly**, skipping the menu, when the crisis check matches a journal reply.
- Shows the services for the phone's region, read offline from `expo-localization`. If the region is
  neither Australia nor India, the person picks once. A switch moves between the two lists.
- Bundled in the app, so it works offline. Stored per region, so a new country adds data, not
  screens.
- **Verify every number before launch and every 90 days after.**

  | Region | Service | Contact |
  |---|---|---|
  | Australia | Emergency | 000 |
  | Australia | Lifeline | 13 11 14 (also text and chat) |
  | Australia | Kids Helpline | 1800 55 1800 |
  | Australia | Beyond Blue | 1300 22 4636 |
  | India | Emergency | 112 |
  | India | Tele-MANAS | 14416 or 1-800-891-4416 |
  | India | Counselling line | e.g. iCall (to confirm) |
  | Anywhere | Overseas | findahelpline.com |

**2. Give feedback**
- Opens the person's **own email app** (`mailto:` via `Linking`), pre-addressed with the subject
  "Turiya feedback" and a body holding only the app version, device type and current lesson ID.
- Turiya sends nothing itself, and no journal text is ever included.
- If no email app opens, the address is shown with a copy button.

### Crisis check on journal replies
- **Runs on the phone**, matching word and phrase lists from `packages/shared`.
- **Handles** negation and common idioms ("this job is killing me").
- **Leans toward showing help** when unsure.
- **On a match**, the reply is still saved and mental health support opens straight away.
- **Tested** by a crisis test set in CI. In v2, the same lists become Ask's first gate.

### Daily reminder
- **Local notifications** (`expo-notifications`) for the next 7 days at the chosen time, with no
  push server.
- **Today's reminder is cancelled** once a lesson is read; the week is topped up each time the app
  opens.
- **Off by default.** Permission is asked for only when the person turns it on.
- **Generic text** only, e.g. "A quiet moment to read?"

## 3. Everything else

- **No network code.** No analytics, crash-reporting or `expo-updates`. Feedback goes through the
  person's own email app, never from Turiya. Crash reports come only from
  App Store Connect and Play Console. The App Store privacy label is **"Data Not Collected"**.
- **Fonts**: Newsreader, Karla and Noto Serif Devanagari are bundled, so Sanskrit renders offline.
- **Versions**: storage keys end in `_v1`, and lesson IDs stay stable across content fixes.
- **Repo layout**:

  | Folder | Holds |
  |---|---|
  | `apps/mobile` | The React Native app |
  | `packages/shared` | Lesson schema, crisis word lists, support services (AU, IN), content build |
  | `content/` | Reviewed lesson files and the pronunciation scheme |

- **Built to extend**:
  - more Paths (chapters 2–4) are new `content/path/` folders;
  - new regions are new helpline data;
  - v2 adds `apps/api`, `db/` and `evals/` for Ask, without moving anything.
