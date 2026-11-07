# Data model: Turiya v1

Phase 1 output of `/speckit-plan`. Entities come from the spec's Key Entities; rules cite its
requirements.

Data lives in three places:

| Where | What | Personal? |
|---|---|---|
| **Bundled in the app** | Lessons, the four Ask states, support services | No — the same for everyone |
| **Phone storage (MMKV)** | Journal, Path progress, settings | **Yes — never leaves the phone** (FR-035) |
| **Server database** | Kārikā corpus for search, anonymous daily counts | No |

---

## 1. Bundled content (read-only)

### Lesson
The source is one Markdown file per lesson; the format is in
[contracts/lesson-file.md](./contracts/lesson-file.md).

| Field | Type | Rules |
|---|---|---|
| `id` | string, e.g. `"1-07"` | Unique; `chapter-order` |
| `order` | integer | 1…N, no gaps; sets the Path order (FR-001) |
| `title` | string | Required |
| `verses[]` | Verse | At least 1 |
| `state` | `jagrat` \| `svapna` \| `sushupti` \| `turiya` \| none | Optional (FR-003) |
| `explanation` | Markdown | Required; about 3–5 minutes to read together with the verse (FR-003) |
| `prompt` | string | Required; the "live it today" prompt; reflective, never a directive (FR-031) |
| `reviewedBy`, `reviewedOn` | string, date | Required before release (FR-004, SC-002) |

**Verse**

| Field | Rules |
|---|---|
| `ref` | e.g. `"Māṇḍūkya 7"` or `"Kārikā 1.10"`; unique across the Path |
| `devanagari` | Required |
| `iast` | Required (standard transliteration) |
| `pronunciation` | Required; follows the scheme in `content/README.md` (FR-002) |
| `translation` | Required |
| `source` | Required; the translation's source and licence (FR-002) |

### Path
`{ id: "karika-ch1", contentVersion: string, lessons: Lesson[] }`. The build fails if lesson order
has gaps, a verse reference is duplicated, or a required field is missing.

### Ask state (four fixed constants)
`{ key, order 1–4, sanskrit, label, tint, verseRef, devanagari, translation }` for jāgrat,
svapna, suṣupti and turīya, in that order (FR-013, FR-014). Never fetched or generated.
`verseRef` links to a lesson where that verse is taught (FR-017).

### Support service
`{ region: "AU", name, contacts: { call?, text?, chat?, web? }, hours, lastVerified: date }`.
At least 000, Lifeline, a youth service and an anxiety/depression service (FR-039). A test fails
the build if any `lastVerified` is older than 90 days (FR-040).

---

## 2. On the phone (MMKV, JSON)

Every read and write is wrapped; if storage fails, the app falls back to empty state and keeps
working.

### Journal entry — key `turiya_journal_v1`, a list, newest first

```ts
type JournalEntry = AskEntry | LessonEntry;

type AskEntry = {
  kind: "ask";
  id: number;              // creation time in ms; unique
  createdAt: string;       // ISO date-time
  question: string;        // non-empty after trimming (FR-010)
  levels: Level[];         // exactly 4, fixed order (FR-013)
  practice: string;        // exactly one line (FR-015)
  fallback: boolean;       // true if the built-in reflection was used (FR-033)
};
type Level = { stateKey: string; heading: string; body: string };

type LessonEntry = {
  kind: "lesson";
  id: number;
  createdAt: string;
  updatedAt: string;       // changes when edited from the lesson (FR-021)
  lessonId: string;        // one entry per lesson at most
  response: string;        // non-empty; an empty response saves nothing
};
```

- Labels, verses and colours are **not** stored. They are looked up from the bundled content, so a
  proofreading fix shows everywhere. Relative times are worked out when the entry is displayed
  (FR-026).
- Ask entries are created only for `reflection` or fallback results. A **crisis** result creates
  nothing.

### Path progress — key `turiya_path_v1`

```ts
type PathProgress = {
  pathId: "karika-ch1";
  contentVersion: string;
  completed: { lessonId: string; at: string }[];  // in order, no gaps
};
```

### Settings — key `turiya_settings_v1`

```ts
type Settings = {
  aiConsent: "unasked" | "granted" | "declined";   // FR-036
  aiConsentAt?: string;
  reminder: { on: boolean; time?: "HH:mm" };       // FR-009
};
```

---

## 3. State changes

### A lesson (FR-006, FR-007)
```
locked ──(previous lesson completed)──▶ available ──(person completes it)──▶ completed
```
- Lesson 1 starts as **available**.
- Only the next lesson in order can be available. There is no daily limit and no skipping ahead.
- **Completed** lessons can be re-read at any time.
- When the last lesson is completed, the Path is **finished**. The person sees the completion
  moment, and can re-read or start again. Starting again clears `completed` but keeps Journal
  entries.

### An Ask (FR-010–FR-016, FR-028, FR-033, FR-036)
```
idle ──submit──▶ [consent "unasked"] ──▶ consent screen ──declined──▶ idle (explains why)
                                                        └─granted──▶ thinking
idle ──submit (consent granted)──▶ thinking
thinking ──reply "reflection"──▶ answer (saved)
thinking ──reply "crisis"──────▶ support screen (nothing saved)
thinking ──error / timeout─────▶ answer using the built-in fallback (saved, fallback: true)
answer ──"ask something else"──▶ idle (cleared)
```

### Reminder notifications (FR-009; see research R6)
- **Turned on**: schedule the next 7 days at the chosen time.
- **Lesson completed**: cancel today's reminder.
- **App opened**: top the schedule back up to 7 days.
- **Turned off, or the Path finished**: cancel all reminders.

---

## 4. Server database (Postgres + pgvector)

No table refers to a person (DEC-5).

```sql
CREATE TABLE karika_passage (
  id          BIGSERIAL PRIMARY KEY,
  ref         TEXT NOT NULL,         -- e.g. 'Kārikā 1.10'
  chapter     INT  NOT NULL,
  verse       INT,
  state_key   TEXT,                  -- jagrat | svapna | sushupti | turiya
  devanagari  TEXT,
  translation TEXT NOT NULL,
  source      TEXT NOT NULL,
  embedding   VECTOR(384) NOT NULL,  -- bge-small-en-v1.5 (research R2)
  search_text TSVECTOR NOT NULL      -- keyword search (research R3)
);
CREATE INDEX ON karika_passage USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON karika_passage USING gin (search_text);

CREATE TABLE daily_counter (
  day                DATE PRIMARY KEY,
  answers            INT NOT NULL DEFAULT 0,
  crisis_routed      INT NOT NULL DEFAULT 0,
  safety_check_fails INT NOT NULL DEFAULT 0
);
```

**Seeding**: chapter 1 rows come from the same `content/path/` files the app bundles; chapters 2–4
come from the licensed corpus. Embeddings and `search_text` are computed at seed time.
