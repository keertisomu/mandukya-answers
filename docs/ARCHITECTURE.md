# Architecture — v1

Decisions referenced here (`DEC-n`) live in [DECISIONS.md](./DECISIONS.md);
requirements (`FR-n`/`NFR-n`) in [REQUIREMENTS.md](./REQUIREMENTS.md).

---

## 1. System overview

```
┌───────────────────────────────────────────────────────────┐
│  React Native app (iOS + Android, Expo)         DEC-2       │
│                                                             │
│   Ask / Thinking / Answer / Journal screens                 │
│                                                             │
│   Journal → on-device only (MMKV, key turiya_journal_v1)    │
│             DEC-1, DEC-3 — never sent to the server         │
└───────────────┬─────────────────────────────────────────────┘
                │  POST /v1/answer  { question }   (HTTPS)
                ▼
┌───────────────────────────────────────────────────────────┐
│  Backend — thin TypeScript service              DEC-4       │
│                                                             │
│   1. crisis screen (fast model) → { kind: "crisis" }        │
│   2. embed(question), retrieve top-k Kārikā passages        │
│   3. generateObject(model, system+context, schema) ← Zod    │
│   4. output safety check (fail → error → client fallback)   │
│   5. return { kind: "reflection", levels[4], practice }     │
│                                                             │
│   Holds the LLM API key (NFR-3). Stores NO user data.       │
└───────────────┬───────────────────────────────┬─────────────┘
                │                                 │
                ▼                                 ▼
   ┌─────────────────────────┐        ┌──────────────────────┐
   │ LLM provider (Anthropic)│        │ Postgres + pgvector   │
   │ no training on inputs    │        │ Kārikā corpus+vectors │
   │ DEC-6                    │        │ verse map, config     │
   │                          │        │ NO user data  DEC-5   │
   └─────────────────────────┘        └──────────────────────┘
```

The user's problem text transits the backend to reach the LLM (so the key stays
off the client) and is **processed once, not persisted** (DEC-1 / NFR-1).

---

## 2. Client (React Native)

### 2.1 Screens & state
Mirror the prototype's state model:

- `tab`: `"ask" | "journal"`
- `phase` (Ask sub-state): `"idle" | "thinking" | "answer"`
- `question` (current input), `asked` (submitted text), `answer` (parsed result
  or fallback)
- `open` (answer card index → bool), `jopen` (journal entry id → bool)
- `journal` (array of entries), `thinkIdx` (status-line cycle index)

### 2.2 On-device persistence (DEC-3)
- Store: `react-native-mmkv` (fallback: `AsyncStorage`).
- Key: `turiya_journal_v1` — a JSON array, newest first.
- Entry shape (frozen, versioned):
  ```ts
  type Level = { name: string; sanskrit: string; tint: string;
                 heading: string; body: string; verseRef: string };
  type Entry = { id: number;        // ms timestamp
                 when: string;       // formatted at save time
                 question: string;
                 levels: Level[];    // length 4, fixed order
                 practice: string;
                 story?: Story;            // v1.1 (FR-36)
                 followUps?: FollowUp[] }; // v1.1 (FR-33)
  type Story = { key: string;      // taxonomy key, e.g. "not_respected"
                 phrase: string }; // first-person, ≤ 8 words
  type FollowUp = { at: number; text: string };
  ```
- The v1.1 fields are optional, so v1 entries stay valid and the storage key does
  not change. Crisis-routed questions never become entries (FR-27).
- All reads/writes wrapped in try/catch; the app renders correctly on empty or
  unavailable storage (NFR-4).

### 2.3 Fixed state → verse map (hard-coded on the client, DEC-7 / FR-9)
The four states, their tints, and their Kārikā verses are **client constants** —
never fetched, never model-generated — so citations cannot hallucinate:

| Order | key | Sanskrit | Label | Tint | Kārikā ref |
|---|---|---|---|---|---|
| 1 | jagrat | Jāgrat | Waking | `#C7A87E` | Kārikā 1.1 |
| 2 | svapna | Svapna | The story | `#A98BA8` | Kārikā 2.1 |
| 3 | sushupti | Suṣupti | Underneath | `#7E9AA8` | Kārikā 1.2 |
| 4 | turiya | Turīya | The witness | `#3F4F52` | Kārikā 1.10 |

Devanagari + translation for each come from the handoff; see [CR-1] — proofread
before launch. Keep this map and the backend's state definitions in sync (a
shared `packages/shared` module if we use a monorepo).

### 2.4 Answer flow (FR-3, FR-7–FR-13)
1. Trim input; empty → no-op.
2. `phase = "thinking"`, start the status-line interval.
3. `POST /v1/answer { question }`.
4. On `kind: "crisis"`: show the support screen (§2.6). Save nothing, schedule
   nothing; `phase = "idle"` once the user leaves the screen (FR-26, FR-27).
5. On `kind: "reflection"`: validate shape client-side too; build the `Entry` by
   merging returned `levels`/`practice` (and `story` in v1.1) with the verse map;
   prepend to `journal`; persist; `phase = "answer"`. In v1.1, schedule the
   follow-up notification (§2.7).
6. On any failure: use the built-in `fallback(question)` text (ported verbatim
   from the prototype); build and save the entry the same way; `phase = "answer"`.
7. Stop the interval in every case. The Answer screen always shows the
   attribution line (FR-30) and the help link (FR-28).

### 2.5 Design system (NFR-6)
Encode the handoff's tokens once (colors, fonts — Newsreader / Karla / Noto Serif
Devanagari, radii, type scale, the input-card shadow) and build components from
them. Gate `breathe` / `fadeup` / `driftglow` animations on reduce-motion (NFR-5).

### 2.6 Support screen & helplines (DEC-8, FR-26/FR-28)
- One screen, reached two ways: a `crisis` response, or the "Need to talk to
  someone?" link on Answer and Journal.
- The helpline list is **bundled in the app** (works offline), chosen by device
  locale, with an international directory as the fallback. Starting list — **every
  entry must be verified before launch (CR-4):**

  | Region | Resource |
  |---|---|
  | United States, Canada | 988 Suicide & Crisis Lifeline — call or text 988 |
  | United Kingdom, Ireland | Samaritans — 116 123 |
  | India | Tele-MANAS — 14416 |
  | Elsewhere | findahelpline.com (international directory) |

- Tone matches the app: warm, brief, no verse, no four-states framing.

### 2.7 Follow-up notifications (v1.1, FR-31–FR-35)
- Local notifications via `expo-notifications`, scheduled on the device 3 days
  after an answer. No server push, no push token, nothing leaves the device.
- Payload carries only the entry `id`; the visible text is generic and never
  includes journal content (FR-32).
- Cancel the scheduled notification when the user adds a follow-up themselves, or
  when follow-ups are turned off in settings.
- Permission is requested after the first answer, never at launch (FR-34).

### 2.8 Pattern mirror (v1.1, FR-36–FR-39)
- Group journal entries by `story.key`, count, sort descending; show the section
  once 5+ entries carry a label. Pure on-device computation over the local
  journal.
- The taxonomy (keys + display names) lives in `packages/shared`, so the backend
  schema and the client grouping always agree (CR-5).

---

## 3. Backend (thin TypeScript service, DEC-4)

### 3.1 Responsibilities
- Hold the LLM key (NFR-3).
- Turn a question into a grounded, schema-valid four-state answer.
- Stay stateless with respect to users — no per-user storage, no auth in v1.

### 3.2 Endpoints (versioned, NFR-7)
- `POST /v1/answer`
  - **Request:** `{ "question": string }`
  - **Response (200)** is one of two shapes, discriminated by `kind`:
    - `{ "kind": "reflection", "levels": [{ "heading": string, "body": string }, ×4],
      "practice": string }` — in v1.1 also `"story": { "key": string,
      "phrase": string }`.
    - `{ "kind": "crisis" }` — no reflection was generated (FR-25).
  - **On screen/model/validation/safety-check failure:** return a 4xx/5xx with a
    small error body; the client falls back (it does **not** depend on the server
    for the fallback text).
  - Clients must ignore unknown fields, so v1.1's `story` doesn't break v1 builds.
- `GET /v1/health` — liveness for deploys.

The response intentionally carries only model-generated text. Verses/tints/order
are merged on the client from §2.3, so the contract can't be polluted by the
model.

### 3.3 Answer pipeline
1. **Crisis screen** (DEC-8, FR-25). A fast classifier call (`generateObject`
   with a small schema, e.g. `{ crisis: boolean, category }`) on the question. If
   `crisis`, return `{ kind: "crisis" }` immediately and increment the anonymous
   counter. Tuned for recall (NFR-11). To save latency it can run in parallel with
   step 2, discarding the retrieval result when it flags.
2. **Embed** the question and **retrieve** top-k relevant Kārikā passages from
   pgvector (the corpus is small, so k is small and latency is low).
3. **Generate** with `generateObject` (Vercel AI SDK), passing:
   - the system prompt (below),
   - the retrieved passages as grounding context,
   - a **Zod schema** that enforces `{ levels: [{heading, body}].length == 4,
     practice }` (plus `story` in v1.1, with `key` constrained to the taxonomy
     enum).
4. **Output safety check** (FR-29). A fast classifier call over the generated
   text against the FR-29 rules. Fail → error → client fallback.
5. **Validate** against the schema; on invalid output, surface an error so the
   client falls back (FR-13). `max_tokens ≈ 1400` (NFR-9).

The crisis test set (NFR-11) runs against steps 1 and 4 in CI whenever the
prompts or model IDs change.

### 3.4 System prompt (from the handoff; grounding added)
> You are a warm, plain-spoken friend grounded in Gauḍapāda's Māṇḍūkya Kārikā
> (the commentary on the Māṇḍūkya Upaniṣad). You never use jargon without
> explaining it, never moralize, never say 'as an AI'. Answer the person's
> everyday problem through the four states of consciousness: jagrat (waking — the
> bare observable facts), svapna (dream — the story/projection they are running on
> top of the facts), sushupti (deep sleep — the rest and wholeness that remains
> when the problem is absent), turiya (the witness that is aware of all three and
> is untouched). Be concrete about THEIR specific situation, never generic. 45–70
> words per level. heading: a short plain-English line, max 8 words. practice: one
> small concrete thing to do today, max 25 words.

With RAG (DEC-7), the retrieved Kārikā passages are supplied as context and the
prompt instructs the model to ground its reading of each state in them. Output
shape is enforced by the schema, so we drop the prototype's "reply with ONLY
JSON / regex the first `{…}`" instructions.

### 3.5 Model & provider (DEC-6)
- Provider: Anthropic via `@ai-sdk/anthropic` (no training on API inputs).
- Default model: `claude-sonnet-5-5`; `claude-opus-5-5` for higher quality,
  `claude-haiku-5-5` for lowest cost. Set via env/config, never hard-coded.
- Safety classifiers (crisis screen, output check): `claude-haiku-5-5` by default
  — fast and cheap. Configured separately from the answer model.

---

## 4. Data model (Postgres + pgvector, DEC-5)

Holds only non-personal content.

```sql
-- the commentary corpus, chunked for retrieval
CREATE TABLE karika_passage (
  id          BIGSERIAL PRIMARY KEY,
  chapter     INT  NOT NULL,          -- Kārikā prakaraṇa / chapter
  verse       INT,                    -- verse number if applicable
  state_key   TEXT,                   -- optional tag: jagrat|svapna|sushupti|turiya
  devanagari  TEXT,
  translation TEXT NOT NULL,          -- licensed translation/commentary prose
  source      TEXT NOT NULL,          -- provenance of the text (CR-2)
  embedding   VECTOR(1536)            -- dim matches the embeddings model
);
CREATE INDEX ON karika_passage USING hnsw (embedding vector_cosine_ops);

-- the fixed display verses + ordering (single source of truth with the client)
CREATE TABLE state_verse (
  state_key   TEXT PRIMARY KEY,       -- jagrat|svapna|sushupti|turiya
  ord         INT  NOT NULL,
  sanskrit    TEXT NOT NULL,
  label       TEXT NOT NULL,
  tint        TEXT NOT NULL,
  verse_ref   TEXT NOT NULL,
  devanagari  TEXT NOT NULL,
  translation TEXT NOT NULL
);

-- optional, anonymous, no PII, no problem text
CREATE TABLE daily_counter (
  day                DATE PRIMARY KEY,
  answers            INT NOT NULL DEFAULT 0,
  crisis_routed      INT NOT NULL DEFAULT 0,   -- FR-27: count only, never text
  safety_check_fails INT NOT NULL DEFAULT 0    -- FR-29: watch for prompt drift
);
```

No table references a user. There is no `user`, `session`, or `journal` table by
design (DEC-1).

**Seeding.** A one-off script loads the corpus from the licensed source (CR-2),
chunks and embeds it, and populates `state_verse` from the proofread verse map
(CR-1). Embeddings are computed once at seed time.

---

## 5. Cross-cutting

- **Config/secrets:** LLM key and model ID via environment variables on the
  backend only.
- **Observability:** minimal request logging that **excludes** question/answer
  text (NFR-1), and records crisis routing only as a count (FR-27). If/when we want tracing, that's a reason to revisit Mastra (DEC-4).
- **Versioning:** API under `/v1`; storage key suffixed `_v1`; DB migrations
  tracked.
- **Monorepo (suggested):** `apps/mobile` (RN), `apps/api` (backend),
  `packages/shared` (the state/verse constants and the answer schema shared by
  both), `db/` (migrations + seed).

---

## 6. v2 extension points (not built now)

- Swap the thin pipeline for a **Mastra workflow** (multi-step practice, Śaṅkara
  layer, evals, tracing) — additive because Mastra sits on the same AI SDK.
- Add a **verse library** screen backed by `karika_passage`.
- **Local-first encrypted sync** for the journal if users ask for it (DEC-1),
  keeping plaintext off the server.
