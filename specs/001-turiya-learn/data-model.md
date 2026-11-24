# Data model: Turiya v1 — Learn

Phase 1 output of `/speckit-plan`. Entities come from the spec's Key Entities; rules cite its
requirements. v1 has **no server**, so data lives in two places:

| Where | What | Personal? |
|---|---|---|
| **Bundled in the app** | Lessons, support services | No — the same for everyone |
| **Phone storage (MMKV)** | Progress, saved lessons, Journal replies, settings | **Yes — never leaves the phone** (FR-027) |

---

## 1. Bundled content (read-only)

Built from `content/path/mandukya/*.md` (format: [contracts/lesson-file.md](./contracts/lesson-file.md)).

### Lesson
| Field | Type | Rules |
|---|---|---|
| `id` | string, e.g. `"07b"` | Unique |
| `order` | integer | 1…N, no gaps; sets the Path order (FR-001, FR-007) |
| `title` | string | Required |
| `mantra` | integer 1–12 | The mantra this lesson teaches (FR-001) |
| `part` | `{ n, of, heading }` | Required when a mantra spans several lessons (FR-002, FR-003) |
| `phrases[]` | Phrase | At least 1, in the mantra's order (FR-002) |
| `karikas[]` | Kārikā | On the mantra's last part only; may be empty (FR-004) |
| `ending` | Ending | **Required on the mantra's last part only** (FR-005) |
| `reviewedBy`, `reviewedOn` | string, date | Required for a release build (FR-006, SC-003) |

### Phrase
| Field | Rules |
|---|---|
| `devanagari` | Required |
| `iast` | Required (standard transliteration) |
| `pronunciation` | Required; follows the scheme in `content/README.md` |
| `meaning` | Required; a few words |
| `note` | Required; a short plain-words explanation (our own commentary) |

### Kārikā (Gauḍapāda's verse on the mantra)
| Field | Rules |
|---|---|
| `ref` | `"Kārikā 1.1"` … `"Kārikā 1.29"` |
| `devanagari`, `iast`, `pronunciation` | Required |
| `translation`, `source` | Required; public-domain or permitted translation |
| `note` | Required; plain words on how it helps understand the mantra (our own commentary) |
| `featured` | `true` for exactly one kārikā per mantra, shown under "What Gauḍapāda says"; the rest go under "More from Gauḍapāda", shown one at a time (FR-004) |

### Ending (last part of a mantra)
| Field | Rules |
|---|---|
| `fullMantra` | Devanagari, IAST, pronunciation, translation, `ref` and `source` — all required |
| `summary` | Required; a few plain sentences |
| `state` | Optional: `jagrat` \| `svapna` \| `sushupti` \| `turiya` |
| `prompt` | Required; a small reflection for the day, never advice about the person's life (FR-025) |

**Build checks**:
- no gaps in `order`, and mantras 1–12 all present, in order;
- every mantra has exactly one `ending`, on its last part;
- **every kārikā 1.1–1.29 appears exactly once** across the Path (FR-004, SC-003);
- a mantra with kārikās has exactly one marked `featured`;
- `part.n` runs 1…`of`;
- every required field is present;
- a release build fails if `reviewedBy` is missing.

### Support service
`{ region: "AU" | "IN", name, contacts: { call?, text?, chat?, web? }, hours, emergency?: true, lastVerified: date }`.
- **Australia**: at least 000, Lifeline, a youth service and an anxiety/depression service.
- **India**: at least 112, Tele-MANAS and one counselling line.

A test fails if either list is short of these, or if any `lastVerified` is older than 90 days (FR-030,
FR-031).

### Feedback
`{ address, subject: "Turiya feedback" }`, bundled. The email body is filled in at the moment of
sending with the app version, device type and current lesson ID only (FR-032). Nothing is stored.

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
  pathId: "mandukya";
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

type Settings = {
  reminder: { on: boolean; time?: "HH:mm" };
  supportRegion?: "AU" | "IN";   // set only if the phone's region is neither, or the person switches (FR-030)
};
```

- Lesson titles, mantras and prompts are **not** copied into the Journal. They are looked up from the
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
reply saved ──matches crisis lists──▶ mental health support opens (reply kept)
            └─no match──────────────▶ nothing else happens
```

### Support (FR-024, FR-030, FR-032)
```
header "Support" ──▶ menu ──"Mental health support"──▶ services for region (switch AU ⇄ IN)
                         └─"Give feedback"──────────▶ explain ──"Open email"──▶ person's email app
crisis match on a reply ─────────────────────────────▶ services for region (skips the menu)
```
Region: the phone's region if it is AU or IN; otherwise `supportRegion`, asked once.

### Reminder (FR-022; research R6)
- **Turned on**: schedule the next 7 days at the chosen time.
- **A lesson read or finished today**: cancel today's reminder.
- **App opened**: top the schedule back up to 7 days.
- **Turned off**: cancel all reminders.
