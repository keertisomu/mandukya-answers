# Ask — v2 design (not built in v1)

Everything designed for **Ask** before it moved to v2 (DEC-12), kept in one place. When v2 starts,
turn this into spec 002 with `/speckit-specify`, and add the server to the architecture.

**What Ask is**: the person describes something weighing on them and gets a calm reflection through
the four states of consciousness, with one fixed verse per state and one small practice. It applies
what the Learn lessons teach to everyday life.

## 1. Behaviour

- **Input**: a typed problem, with suggestion chips that pre-fill a first-person sentence ("A
  relationship", "Stress", "Anger", "A decision", "Loneliness"). Empty input is ignored.
- **While waiting**: a calming state (breathing ॐ, cycling lines like "sitting with it…").
- **Reflection**: exactly four states, in this order, the first one open:

  | # | State | Label | Colour | Verse ref (from handoff) |
  |---|---|---|---|---|
  | 1 | Jāgrat | Waking | `#C7A87E` | Kārikā 1.1 |
  | 2 | Svapna | The story | `#A98BA8` | Kārikā 2.1 |
  | 3 | Suṣupti | Underneath | `#7E9AA8` | Kārikā 1.2 |
  | 4 | Turīya | The witness | `#3F4F52` | Kārikā 1.10 |

  - Each state has a short heading and a body about the person's own situation, plus its fixed
    verse.
  - ⚠️ The Devanagari given for 1.1 and 1.2 looks like the two halves of **Kārikā 1.3**. Confirm
    against the chosen edition.
- **Practice**: one "one thing to try" line — a small reflective action, never an instruction.
- **Links to Learn**: each verse links to its lesson once the person has reached it.
- **Saved**: each reflection goes to the Journal. A crisis question is never saved.
- **Built-in fallback**: if anything fails, a steadying four-state reflection (from the prototype)
  is shown instead of an error.
- **Design**: the screens follow the design handoff in `design/`.

## 2. Safety and consent

- **Crisis check before writing**: first the shared word lists, then a fast model check, leaning
  toward flagging. If either flags, show the support screen. Nothing is saved, logged or followed
  up.
- **Output safety check**: reject any reflection that:
  - endorses violence or self-harm;
  - gives medical or therapy directions;
  - demeans any caste, gender or group;
  - speaks as a deity or guru.
- **Attribution on every reflection**: "A reflection drawing on Gauḍapāda's Kārikā — not religious
  authority, therapy, or medical advice."
- **Consent before the first question**: name the AI provider (Anthropic), say only the question
  text is sent, that it may be processed outside Australia and isn't stored, and that everything
  else stays on the phone. Ask doesn't run until the person agrees.
  - Apple App Review Guideline 5.1.2(i) (November 2025) requires this.
  - It also meets Australian privacy expectations.
  - **Note**: adding Ask changes v1's "nothing leaves the phone" promise. Update the privacy policy
    and the App Store privacy label when it ships.

## 3. Server

A thin, stateless TypeScript service. It holds the AI key and stores nothing personal.

### API — `POST /v1/answer`

Request `{ "question": string }`, 1–2000 characters. The response is one of:

```json
{ "kind": "reflection",
  "levels": [ { "heading": "…", "body": "…" }, …4 ],
  "practice": "…" }
```
```json
{ "kind": "crisis" }
```

| Field | Rule |
|---|---|
| `levels` | Exactly 4, in state order |
| `heading` | 1–8 words |
| `body` | 45–70 words |
| `practice` | 1–25 words |

- **Errors**: `400`, `429` or `5xx` (with a 25-second app timeout) → the app shows the built-in
  reflection. The error body never echoes the question.
- **Reply contents**: only generated text. Verses, order and colours come from the app.
- **Compatibility**: clients ignore unknown fields.
- **Health**: `GET /v1/health` is a liveness check.

### Pipeline
1. **Crisis check**: word lists, then `claude-haiku-5-5`. If either flags → `crisis`.
2. **Hybrid search** over the Kārikā:
   - pgvector cosine search plus Postgres full-text search, merged by reciprocal rank fusion;
   - a synonym list (turiya / turīya / the fourth).
   - The constitution requires hybrid search.
3. **Write the reflection**: AI SDK 6 `generateText` with `Output.object` and the shared Zod schema.
   `max_tokens` ≈ 1400.
4. **Output safety check** (`claude-haiku-5-5`). If it fails → error → the app falls back.

### System prompt (from the handoff)
> You are a warm, plain-spoken friend grounded in Gauḍapāda's Māṇḍūkya Kārikā (the commentary on
> the Māṇḍūkya Upaniṣad). You never use jargon without explaining it, never moralize, never say 'as
> an AI'. Answer the person's everyday problem through the four states of consciousness: jagrat
> (waking — the bare observable facts), svapna (dream — the story/projection they are running on
> top of the facts), sushupti (deep sleep — the rest and wholeness that remains when the problem is
> absent), turiya (the witness that is aware of all three and is untouched). Be concrete about
> THEIR specific situation, never generic. 45–70 words per level. heading: a short plain-English
> line, max 8 words. practice: one small concrete thing to do today, max 25 words.

### Database (Postgres + pgvector)

```sql
CREATE TABLE karika_passage (
  id BIGSERIAL PRIMARY KEY, ref TEXT NOT NULL, chapter INT NOT NULL, verse INT,
  state_key TEXT, devanagari TEXT, translation TEXT NOT NULL, source TEXT NOT NULL,
  embedding VECTOR(384) NOT NULL, search_text TSVECTOR NOT NULL
);
CREATE INDEX ON karika_passage USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON karika_passage USING gin (search_text);

CREATE TABLE daily_counter (
  day DATE PRIMARY KEY, answers INT NOT NULL DEFAULT 0,
  crisis_routed INT NOT NULL DEFAULT 0, safety_check_fails INT NOT NULL DEFAULT 0
);
```

No table refers to a person. Chapter 1 rows are seeded from the same `content/` lesson files the app
bundles, so Learn and Ask always agree on the text.

## 4. Technology decisions (researched November 2026)

| Topic | Decision | Why |
|---|---|---|
| Structured output | AI SDK 6 `generateText` + `Output.object` | `generateObject` is deprecated in AI SDK 6 |
| Embeddings | `bge-small-en-v1.5` run on our own server (`@huggingface/transformers`, 384 dims) | Anthropic has no embeddings API; a hosted one would show every question to a second company |
| Search | Vector + full-text, merged by reciprocal rank fusion | The constitution requires hybrid search; it's simple and needs no tuning data |
| Server | Hono on Node.js 24 LTS | Small, TypeScript-first |
| Hosting | Sydney region. Candidates: Fly.io `syd` or AWS/Google Cloud Sydney, with Neon Postgres (Sydney, pgvector) | Speed for Australian users. Fly.io Sydney had cross-region incidents in 2026 |
| Models | `claude-sonnet-5-5` to start; use the cheapest model that passes the golden eval. `claude-haiku-5-5` for safety checks | The constitution favours the cost-effective tier |

## 5. Testing the AI (constitution, Principle VII)

- **Golden eval set**: 30–40 situations, about 5 per theme. Each checks the expected state, the
  expected verse, and that no verdict or advice was given. Themes:
  - relationship conflict;
  - anger;
  - grief;
  - decision paralysis;
  - work stress;
  - comparison and envy;
  - loneliness.
- **Crisis test set**: crisis and non-crisis wording, including idioms such as "this job is killing
  me".
- **When they run**: both run in CI on every prompt, model or content change. A regression blocks
  release.

## 6. After Ask ships — deepening it

- **Follow-up check-ins**: 3 days after an Ask, one local notification ("Something you were sitting
  with a few days ago — how does it look now?") that never contains the question. The reply is
  saved under the original. Never sent for crisis questions.
- **Pattern mirror**: each reflection names the "story" behind the problem from a fixed list (e.g.
  "not respected", "must fix it alone") plus a short personal phrase. After 5 or more entries, the
  Journal shows recurring stories, worded as noticing, never as diagnosis.
- **How we'll know**: compare week-4 retention before and after, using App Store Connect and Play
  Console.
