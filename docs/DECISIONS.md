# Decisions (v1)

Short records of the choices that shape v1, why they were made, and what would
change them. Numbered like ADRs so we can reference them (`DEC-1`, etc.).

---

## DEC-1 — Journal stays on the device; the server never stores user problems

**Status:** Accepted

**Context.** The product lets people type private, often emotionally loaded
problems. The original design handoff stores the journal only on the device
(browser `localStorage`, key `turiya_journal_v1`) with no accounts and no
server. Separately, there was a wish for "a simple Postgres DB" and a stated
worry about user data sitting on centralized servers.

**Decision.** The journal (the user's questions + the generated answers) lives
**only on the device**. Nothing personal is sent to, or stored on, our server.
Postgres still exists, but it holds only **non-personal** content (see DEC-5).
The request that produces an answer is processed in-flight and not persisted
server-side.

**Why.** It makes the privacy promise true by construction rather than by
policy — there is no central store of user problems to leak, subpoena, or
mishandle. It also removes accounts/auth from v1 scope entirely.

**Consequences.**
- No cross-device sync and no cloud backup in v1. If the app is uninstalled, the
  journal is gone. The UI already sets this expectation ("Saved only on this
  device").
- The user's problem text *does* transit our backend to reach the LLM provider
  (needed to keep the API key off the client — DEC-4). It is used for that one
  request and not stored. This must be stated plainly in the privacy copy.

**Revisit if** users ask for sync/backup across devices. The likely v2 answer is
*local-first with opt-in, end-to-end-encrypted* backup, so the server still
never sees plaintext.

---

## DEC-2 — React Native (iOS + Android), one codebase

**Status:** Accepted

**Decision.** Build the client as a single React Native app targeting iOS and
Android. Use **Expo** unless a reason to go bare emerges.

**Why.** The design is mobile-first. One TypeScript codebase covers both stores
and pairs naturally with a TypeScript backend (DEC-4), so the whole project is
one language. Expo gets us to a running app fastest and handles the build/release
plumbing.

**Consequences.**
- `localStorage` does not exist in React Native. On-device persistence uses an RN
  store instead (see DEC-3).
- Faithful recreation of the design tokens is our responsibility — the handoff is
  a reference, not code to copy.

**Revisit if** we decide v1 is iOS-only (then native SwiftUI becomes an option)
or if a web/PWA version is wanted first.

---

## DEC-3 — On-device storage: MMKV (fast KV), journal as a JSON array

**Status:** Accepted (implementation detail of DEC-1)

**Decision.** Persist the journal with **react-native-mmkv**, storing the same
entry shape the prototype uses, under a versioned key (`turiya_journal_v1`).

**Why.** MMKV is fast, synchronous, and simple — a good match for a single
growing JSON array read at startup and written on each new entry. `AsyncStorage`
is an acceptable fallback; a device SQLite DB is more than v1 needs.

**Consequences.** All reads/writes wrapped in try/catch; the app renders
correctly when storage is empty or unavailable. Entry shape is frozen and
versioned so a future migration is possible.

---

## DEC-4 — Backend: start on the Vercel AI SDK + Postgres/pgvector; keep Mastra open for v2

**Status:** Accepted (with a documented alternative)

**Decision.** v1 backend is a thin **TypeScript** HTTP service built on the
**Vercel AI SDK** (`ai` + `@ai-sdk/anthropic`), talking to **Postgres with the
pgvector extension**. The LLM key lives only here. **Mastra** is explicitly kept
as the v2 option for workflows/evals/tracing — it is built on the same AI SDK, so
adopting it later is additive, not a rewrite.

**Why.**
- The RAG corpus (Gauḍapāda's Kārikā) is small and static → it does not need a
  heavy agent framework yet.
- The riskiest part of the app is the structured-JSON contract. The AI SDK's
  `generateObject` + a Zod schema *enforces* the `{levels[4], practice}` shape
  with validation, which is strictly better than the prototype's "regex the first
  `{…}` and hope" parsing.
- pgvector keeps the vector store inside the one Postgres we already wanted — no
  extra infra.
- Mastra is young and fast-moving; a stable, minimal base is worth more for a
  calm, long-lived app. The extend-later path is preserved.

**Alternative considered — adopt Mastra now.** Reasonable if we want its
opinionated scaffolding (agents, workflow steps, built-in evals, free-tier
tracing UI) from day one, and are comfortable tracking a newer framework. The v2
roadmap (Śaṅkara, verse library, multi-step practice) is workflow-shaped and is
where Mastra would shine. Mastra's framework is free/open-source (Apache-2.0) and
self-hostable; its Cloud has a free Starter tier (~100K observability events, 24
CPU-hours, 15-day retention — confirm current limits at signup).

**Revisit when** we start v2 workflows or want answer-quality evals/tracing —
re-evaluate adopting Mastra at that point.

---

## DEC-5 — What Postgres holds in v1 (and what it never holds)

**Status:** Accepted (follows from DEC-1 and DEC-4)

**Holds:** the Kārikā corpus + embeddings (pgvector) for RAG; the fixed
state→verse map; app config. Optionally, anonymous aggregate counters with no
PII and no problem text.

**Never holds:** user questions, generated answers, the journal, user identifiers,
or anything that could re-identify a person.

---

## DEC-6 — LLM provider: Anthropic Claude (no training on inputs)

**Status:** Accepted (recommended; swappable)

**Decision.** Use the Anthropic API via `@ai-sdk/anthropic`. Default model
**Claude Sonnet 5.5** (`claude-sonnet-5-5`) for the answer call — strong
instruction-following and the warm, plain-spoken tone this app needs, at
$2/$10 per 1M tokens in/out. **Claude Opus 5.5** (`claude-opus-5-5`, $4/$20) is
the higher-quality option; **Claude Haiku 5.5** (`claude-haiku-5-5`, $0.10/$0.50)
is the budget option. The model is a single config value, swappable without code
changes.

**Why.** Anthropic's **API tier does not train on inputs or outputs by default**
(unlike consumer chat apps), which is exactly what "no LLM model training" asks
for. The AI SDK keeps the provider swappable if that ever changes.

**Consequences.** State the no-training fact in the privacy copy. Keep the model
ID in env/config, not hard-coded in call sites.

---

## DEC-7 — Answer fidelity: RAG-grounded in a curated Kārikā corpus

**Status:** Accepted

**Decision.** Every answer is grounded in **retrieved passages from a vetted text
of Gauḍapāda's Māṇḍūkya Kārikā** (plus a trusted translation), not the model's
loose recall. The four displayed verses stay **hard-coded** per state (citations
can't hallucinate); RAG shapes the *interpretation* in each level's body.

**Why.** The product's whole premise is answering through Gauḍapāda's commentary
rather than the bare Upaniṣad. Grounding in real passages makes answers more
faithful and defensible for a sacred-text app.

**Consequences.** Two non-code dependencies become real v1 work:
1. **Proofread** the Devanagari and Kārikā references against a Sanskrit source
   (the handoff itself flags them as "plausible but unverified").
2. **Source and license** a translation/commentary text for the corpus
   (public-domain options exist — verify copyright before shipping).

---

## Open questions (not blocking; defaults noted)

- **Exact LLM model** — default Claude Sonnet 5.5; confirm or override.
- **Translation source** for the corpus — do we have a preferred text, or should
  we research public-domain options (e.g. Nikhilananda) and confirm licensing?
- **Expo vs bare RN** — default Expo.
- **Analytics** — default: none in v1 (privacy purity). If we want anonymous
  counts later, they go in Postgres with no PII (DEC-5).
