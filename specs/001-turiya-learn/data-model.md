# Data model: Turiya v1 — Learn

Phase 1 output of `/speckit-plan`. Entities come from the spec's Key Entities; rules cite its
requirements. v1 has **no server**, so data lives in two places:

| Where | What | Personal? |
|---|---|---|
| **Bundled in the app** | Lessons, support services | No — the same for everyone |
| **Phone storage (MMKV)** | Progress, saved lessons, Journal replies, settings | **Yes — never leaves the phone** (FR-027) |

---

## 1. Bundled content (read-only)

Built from `content/path/karika-ch1/*.md` (format: [contracts/lesson-file.md](./contracts/lesson-file.md)).

### Lesson
| Field | Type | Rules |
|---|---|---|
| `id` | string, e.g. `"07b"` | Unique |
| `order` | integer | 1…N, no gaps; sets the Path order (FR-001, FR-007) |
| `title` | string | Required |
| `verse` | string, e.g. `"Māṇḍūkya 7"` | The verse this lesson teaches |
| `part` | `{ n, of, heading }` | Required when a verse spans several lessons (FR-003, FR-004) |
| `phrases[]` | Phrase | At least 1, in the verse's order (FR-002) |
| `ending` | Ending | **Required on the verse's last part only** (FR-005) |
| `reviewedBy`, `reviewedOn` | string, date | Required for a release build (FR-006, SC-003) |

### Phrase
| Field | Rules |
|---|---|
| `devanagari` | Required |
| `iast` | Required (standard transliteration) |
| `pronunciation` | Required; follows the scheme in `content/README.md` |
| `meaning` | Required; a few words |
| `note` | Required; a short plain-words explanation (our own commentary) |

### Ending (last part of a verse)
| Field | Rules |
|---|---|
| `fullVerse` | Devanagari, IAST, pronunciation, translation, `ref` and `source` — all required |
| `summary` | Required; a few plain sentences |
| `state` | Optional: `jagrat` \| `svapna` \| `sushupti` \| `turiya` |
| `prompt` | Required; a small reflection for the day, never advice about the person's life (FR-025) |

**Build checks**:
- no gaps in `order`;
- every multi-part verse has exactly one `ending`, on its last part;
- `part.n` runs 1…`of`;
- every required field is present;
- a release build fails if `reviewedBy` is missing.

### Support service
`{ region: "AU", name, contacts: { call?, text?, chat?, web? }, hours, lastVerified: date }`.
At least 000, Lifeline, a youth service and an anxiety/depression service (FR-030). A test fails if
any `lastVerified` is older than 90 days (FR-031).

---

## 2. On the phone (MMKV, JSON)

Every read and write is wrapped. If storage fails, the app shows empty states and keeps working.

| Key | Holds |
|---|---|
| `turiya_progress_v1` | Progress |
| `turiya_saved_v1` | Saved lessons |
| `turiya_journal_v1` | Journal replies |
| `turiya_settings_v1` | Reminder setting |

```ts
type Progress = {
  pathId: "karika-ch1";
  contentVersion: string;                          // lesson IDs are kept stable across content fixes
  completed: { lessonId: string; at: string }[];  // in order, no gaps
  lastReadOn?: string;                             // local date; used by the reminder (FR-022)
};

type Saved = { lessonId: string; savedAt: string }[];   // newest first (FR-009, FR-010)

type JournalEntry = {
  kind: "lesson";          // a field kept so v2 can add "ask" entries
  id: number;              // creation time in ms
  lessonId: string;        // at most one entry per lesson
  response: string;        // non-empty; an empty reply saves nothing
  createdAt: string;
  updatedAt: string;       // changes when edited from the lesson (FR-014)
};

type Settings = { reminder: { on: boolean; time?: "HH:mm" } };
```

- Lesson titles, verses and prompts are **not** copied into the Journal. They are looked up from the
  bundled content, so a proofreading fix shows everywhere.
- Relative times are worked out at display time (FR-020).

---

## 3. State changes

### A lesson (FR-007, FR-008)
```
locked ──(previous lesson completed)──▶ available ──(person finishes it)──▶ completed
```
- Lesson 1 starts as **available**. Only the next lesson in order can be available.
- **Completed** lessons can be re-read and saved at any time.
- When the last lesson is completed, the Path is **finished**. The person sees the completion
  moment. Starting again clears `completed`, but keeps Saved and the Journal.

### Saving (FR-009, FR-010)
```
not saved ──tap "Save"──▶ saved ──tap "Saved"──▶ not saved
```
Allowed only for completed lessons, or the lesson being read.

### Crisis check (FR-023)
```
reply saved ──matches crisis lists──▶ support screen shown (reply kept)
            └─no match──────────────▶ nothing else happens
```

### Reminder (FR-022; research R6)
- **Turned on**: schedule the next 7 days at the chosen time.
- **A lesson read or finished today**: cancel today's reminder.
- **App opened**: top the schedule back up to 7 days.
- **Turned off**: cancel all reminders.
