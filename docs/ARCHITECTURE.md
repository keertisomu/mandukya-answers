# Architecture — v1

How v1 is built. **What** it does is in the
[spec](../specs/001-turiya-ask-journal/spec.md) (`FR-`/`SC-` IDs); **why** each choice was made is
in [DECISIONS.md](./DECISIONS.md) (`DEC-` IDs).

## 1. The big picture

```
Phone (React Native app)
  ├─ Learn ............ lessons are bundled in the app; works offline
  ├─ Journal, progress  stored on the phone only; never sent anywhere
  └─ Ask ──HTTPS──▶ Our server (TypeScript)
                      1. crisis check
                      2. find relevant Kārikā passages ──▶ Postgres + pgvector
                      3. write the reflection ───────────▶ Anthropic (Claude)
                      4. safety-check it, then reply
```

Three rules shape everything:

- **Only Ask needs the server.** Learn and Journal work with no network.
- **Only the Ask question leaves the phone**, after the person agrees (FR-036). The server uses it
  once and stores nothing (DEC-1).
- **No AI-written scripture.** Verses and lessons are fixed, reviewed content. AI writes only the
  Ask reflection text.

## 2. The app (React Native + Expo, DEC-2)

### Screens
- **Tabs**: Learn (opens first), Ask, Journal.
- **Ask** has three states: idle → thinking → answer.
- **Shared**: the support screen and the consent screen.
- **Designs**: Ask and Journal follow the design handoff (`design/`). Learn, the reminder setting and
  the consent screen still need designs.

### Data on the phone (MMKV, DEC-3)
All stored as JSON on the device only. Every read and write is wrapped so the app still works if
storage fails.

| Key | Holds |
|---|---|
| `turiya_journal_v1` | Journal entries, newest first |
| `turiya_path_v1` | Path progress |
| `turiya_settings_v1` | Reminder time, consent choice |

```ts
type Entry = AskEntry | LessonEntry;

type AskEntry = {
  kind: "ask"; id: number; when: string; question: string;
  levels: Level[];          // always 4, fixed order
  practice: string;
};
type Level = { name: string; sanskrit: string; tint: string;
               heading: string; body: string; verseRef: string };

type LessonEntry = {
  kind: "lesson"; id: number; when: string;
  lessonId: string;         // e.g. "1-07"
  lessonTitle: string; verseRef: string;
  prompt: string; response: string;   // one entry per lesson; editing updates it
};

type PathProgress = {
  pathId: "karika-ch1";
  contentVersion: string;   // lets corrected lessons keep progress
  completed: { lessonId: string; at: number }[];
};
```

### Lessons
- **Source**: one file per lesson in `content/path/karika-ch1/`. Each holds:
  - the verse(s): Devanagari, IAST transliteration, pronunciation guide, translation, reference
    and translation source;
  - the explanation;
  - the related state;
  - the "live it today" prompt.
- **Checked and bundled at build time.** A schema in `packages/shared` rejects missing fields or
  broken order. Lessons ship inside the app, so they work offline and are never generated.
- **Canonical text and commentary stay separate.** Verse and translation are canonical; the
  explanation and prompt are our commentary. They are never mixed (constitution, Principle III).
- **Changes go through pull requests** reviewed for accuracy.
- **Pronunciation guide**: one simple respelling scheme for every verse. Syllables are split by
  hyphens and long sounds doubled, e.g. *MAAN-doo-kya*. The scheme is written down in
  `content/README.md`, so every lesson reads the same way.
- **Order rule**: the next lesson opens as soon as the previous one is done. No daily limit, no
  streaks, no missed-day state.
- **Ask ↔ Learn links**: a fixed map from verse reference to lesson. An Ask verse links to its
  lesson only once the person has reached it.

### The four Ask states (fixed in the app)
Verses are app constants, never fetched or generated, so citations can't be invented.

| # | State | Label | Colour | Verse ref (from handoff) |
|---|---|---|---|---|
| 1 | Jāgrat | Waking | `#C7A87E` | Kārikā 1.1 |
| 2 | Svapna | The story | `#A98BA8` | Kārikā 2.1 |
| 3 | Suṣupti | Underneath | `#7E9AA8` | Kārikā 1.2 |
| 4 | Turīya | The witness | `#3F4F52` | Kārikā 1.10 |

⚠️ **Check before launch**: the Devanagari given for 1.1 and 1.2 looks like the two halves of
**Kārikā 1.3** in standard editions. Confirm against the chosen edition. Devanagari and
translations are in `design/README.md`.

### Ask flow
1. Ignore empty input.
2. First time only: show the consent screen. If declined, stop.
3. Show the thinking state.
4. Call `POST /v1/answer`.
5. Handle the response:
   - `crisis` → show the support screen. Save nothing.
   - `reflection` → merge the text with the four fixed states, save it to the Journal, show it.
   - Any error → show the built-in fallback reflection (ported from the prototype) and save that.
6. Every answer shows the attribution line (FR-034) and the help link.

### Support screen and helplines
- **How people reach it**: three ways —
  - a `crisis` reply from Ask;
  - the on-device check on a lesson response;
  - the "Need to talk to someone?" link on every tab.
- **Bundled in the app**, so it works offline. Stored per region (`AU` in v1), so a new country
  adds data, not screens.
- **Verify every number before launch and every 90 days after.**

  | Service | Contact |
  |---|---|
  | Emergency | 000 |
  | Lifeline | 13 11 14 (also text and chat) |
  | Kids Helpline | 1800 55 1800 |
  | Beyond Blue | 1300 22 4636 |
  | Overseas | findahelpline.com |

### Crisis check on lesson responses
Lesson responses never leave the phone, so this check runs on the device:
- It matches words and phrases from shared lists in `packages/shared`.
- It handles negation and common idioms ("this job is killing me").
- It leans toward showing help when unsure.
- If it matches, the response is still saved and the support screen opens.

The server uses the **same lists** as its first gate, so both paths agree.

### Daily reminder
- A local notification scheduled on the phone. There's no push server.
- Off by default. Permission is asked for only when the person turns it on.
- Generic text only, e.g. "Your next lesson is ready."
- Skipped on days the person has already done a lesson, and once the Path is finished.

### Consent screen
- **Shown once**, before the first question is sent.
- **Says**:
  - the AI provider's name (Anthropic);
  - that only the question text is sent;
  - that it may be processed outside Australia and isn't stored;
  - that everything else stays on the phone.
- **Choice** is saved in `turiya_settings_v1` and can be changed in settings.
- **Purpose**: meets Australian privacy expectations and Apple Guideline 5.1.2(i).

## 3. The server (TypeScript, Vercel AI SDK, DEC-4)

Holds the AI key and turns a question into a safe, grounded reflection. It is stateless: no
accounts, no stored questions.

### API (versioned under `/v1`)

**`POST /v1/answer`** — request: `{ "question": string }`. The response is one of:

```json
{ "kind": "reflection",
  "levels": [ { "heading": "…", "body": "…" }, …4 ],
  "practice": "…" }
```
```json
{ "kind": "crisis" }
```

- On any failure, return an error. The app has its own fallback.
- The reply carries only generated text. Verses, order and colours come from the app, so the AI
  can't change them.
- Clients must ignore unknown fields, so later additions don't break older apps.

**`GET /v1/health`** — liveness check.

### Answer pipeline
1. **Crisis check**:
   - First, the shared word and phrase lists.
   - Then a fast model check (Claude Haiku), tuned to flag when unsure.
   - If either flags: return `crisis` and add 1 to the anonymous daily count.
2. **Find passages**: hybrid search over the Kārikā corpus — vector similarity *plus* keyword match.
   The constitution requires hybrid search, so a question that names "turīya" always finds the
   turīya verses.
3. **Write the reflection**: `generateObject` with the system prompt, the passages and a Zod schema
   that enforces exactly 4 levels plus a practice line. `max_tokens` ≈ 1400.
4. **Safety check** (fast model): reject any reflection that:
   - endorses violence or self-harm;
   - gives medical or therapy directions;
   - demeans any caste, gender or group;
   - speaks as a deity or guru.
5. Return the reflection or an error.

**Limits** (enforced by the schema and the prompt):

| Part | Limit |
|---|---|
| Body | 45–70 words per state |
| Heading | 8 words or fewer |
| Practice | 25 words or fewer |

### System prompt (from the handoff)
> You are a warm, plain-spoken friend grounded in Gauḍapāda's Māṇḍūkya Kārikā (the commentary on
> the Māṇḍūkya Upaniṣad). You never use jargon without explaining it, never moralize, never say 'as
> an AI'. Answer the person's everyday problem through the four states of consciousness: jagrat
> (waking — the bare observable facts), svapna (dream — the story/projection they are running on
> top of the facts), sushupti (deep sleep — the rest and wholeness that remains when the problem is
> absent), turiya (the witness that is aware of all three and is untouched). Be concrete about
> THEIR specific situation, never generic. 45–70 words per level. heading: a short plain-English
> line, max 8 words. practice: one small concrete thing to do today, max 25 words.

The retrieved passages are added as context, with an instruction to ground each state in them.

### Models (DEC-6)
- **Reflection**: `claude-sonnet-5-5`. Use `claude-opus-5-5` for higher quality or
  `claude-haiku-5-5` for lower cost.
- **Safety checks**: `claude-haiku-5-5`.
- **Model IDs** are set in config, never hard-coded.

### Testing the AI (constitution, Principle VII)
- **Golden eval set**: 30–40 realistic situations, about 5 per theme. Each has the expected
  state(s), the expected verse(s) and a pass/fail check that no verdict or advice was given.
  Starting themes:
  - relationship conflict;
  - anger;
  - grief;
  - decision paralysis;
  - work stress;
  - comparison and envy;
  - loneliness.
- **Crisis test set**: crisis and non-crisis wording for both Ask and lesson responses, including
  idioms like "this job is killing me". Missing a real crisis fails; a false alarm is acceptable.
- Both sets run in CI on every prompt, model or content change. A regression blocks release.

## 4. Database (Postgres + pgvector, DEC-5)

Holds only shared content and anonymous counts. No table refers to a person.

```sql
-- Kārikā text, chunked for search
CREATE TABLE karika_passage (
  id          BIGSERIAL PRIMARY KEY,
  chapter     INT  NOT NULL,
  verse       INT,
  state_key   TEXT,              -- jagrat | svapna | sushupti | turiya
  devanagari  TEXT,
  translation TEXT NOT NULL,
  source      TEXT NOT NULL,     -- where the translation came from
  embedding   VECTOR(1536),
  search_text TSVECTOR           -- for keyword search
);
CREATE INDEX ON karika_passage USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON karika_passage USING gin (search_text);

-- anonymous daily counts only
CREATE TABLE daily_counter (
  day                DATE PRIMARY KEY,
  answers            INT NOT NULL DEFAULT 0,
  crisis_routed      INT NOT NULL DEFAULT 0,
  safety_check_fails INT NOT NULL DEFAULT 0
);
```

**Seeding**: a script loads the corpus, embeds it and fills `search_text`. Chapter 1 verses come
from the same `content/path/` files the app bundles, so Learn and Ask never disagree on the text.

## 5. Everything else

- **Secrets**: the AI key and model IDs live in server environment variables only.
- **Logs**: never contain question or reflection text. Crisis routing is a count only.
- **Versions**: API under `/v1`; storage keys end in `_v1`; database migrations are tracked.
- **Repo layout**:

  | Folder | Holds |
  |---|---|
  | `apps/mobile` | The React Native app |
  | `apps/api` | The server |
  | `packages/shared` | Schemas, the four states, crisis word lists |
  | `content/` | Reviewed lessons |
  | `db/` | Migrations and seed script |

- **Built to extend**:
  - more Paths are new `content/path/` folders;
  - new regions are new helpline data;
  - Mastra can wrap the pipeline later without a rewrite (DEC-4).
