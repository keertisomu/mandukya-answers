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

## DEC-8 — Crisis safety is part of v1 and blocks launch

**Status:** Accepted

**Context.** People bring stress, anger and loneliness, which sit close to mental
health. Market research (MARKET_VALIDATION.md §6) found: Gita chatbots condoning
violence; Illinois banning AI therapy (Aug 2025), with Nevada and Utah
restricting it; and a reported App Store rejection of an AI chat app under
Guideline 1.4.1 (medical advice).

**Decision.**
- A **crisis screen** runs before any reflection is generated. If it flags, the
  user sees a calm support screen with regional helplines instead of an answer.
- An **output safety check** runs on every reflection; failures fall back to the
  built-in answer.
- A **help link** is always visible on Answer and Journal, so a missed detection
  still leads somewhere.
- **Attribution** on every answer: a reflection drawing on Gauḍapāda, not
  religious authority, therapy, or medical advice. No clinical claims anywhere.

**Why in v1, not v1.1.** Shipping a problem-answering app without it is the
largest liability and app-review risk we found, and it's cheap: two fast
classifier calls and one screen.

**Privacy.** Crisis-routed questions are never saved, never logged, and never
followed up. The server keeps only an anonymous daily count.

**Consequences.** Adds latency (mitigated by running the screen in parallel with
retrieval) and a small cost per answer. Requires a maintained crisis test set and
verified helpline numbers (CR-4).

---

## DEC-9 — Retention: attract with problems, keep with practice

**Status:** Accepted; partly superseded by DEC-11. The lesson path and the daily
reminder moved into v1, and Ask is no longer the front door. The v1.1 slice and
the "avoid" list below still stand.

**Context.** v1 is opened when something goes wrong, which is episodic.
Meditation apps keep ~6–8% of users at 30 days. Research ranked daily practice
with gentle reminders as the strongest retention lever and structured depth
second.

**Decision.** Ask stays the front door. Retention comes in two steps:
- **v1.1 (soon after launch):** follow-up check-ins on past questions via
  on-device local notifications, and story labels feeding an on-device "pattern
  mirror" in the Journal. Both keep all personal data on the device.
- **v2:** the daily four-states rhythm, which the text itself suggests — morning
  (jāgrat: the facts), evening (svapna: the story I ran), bedtime (suṣupti:
  letting go) — plus a finite "40 days with Gauḍapāda" path (the natural paid
  tier) and a bedtime wind-down.

**Avoid.** Loss-aversion streaks (they clash with the teaching), community (cost
and privacy risk, no evidence), and open-ended chat (drifts toward a persona).

**How we'll know.** Compare week-4 retention for v1 vs v1.1 cohorts in App Store
Connect / Play Console before committing to v2 or a business model.

---

## DEC-10 — v1 launches in Australia only; India next, then other countries

**Status:** Accepted

**Decision.** v1 is released only through the Australian App Store and Google
Play storefronts. India is the v2 region; other countries follow.

**Why.** One English-speaking market means one set of crisis services, one
privacy law (Privacy Act 1988 / Australian Privacy Principles) and one language,
which keeps the safety and compliance work for a first release manageable.
Australia's Indian community is one of its largest overseas-born communities, so
the core audience is there.

**Trade-off.** The market research (MARKET_VALIDATION.md §3) pointed at the
diaspora in the US, UK and Canada; Bible Chat, for example, earns over 95% of its
revenue in the US. Australia is a smaller first market — a deliberate choice to
launch carefully, not to maximise reach.

**Consequences.**
- Crisis resources are Australian (000, Lifeline, Kids Helpline, Beyond Blue),
  verified before launch and every 90 days.
- A data-handling notice before the first Ask says problem text is processed by a
  service that may be outside Australia and is not stored.
- Australian English and day-month dates.
- Helplines are stored per region so later regions add data, not screens.

**Revisit when** planning v2: India needs its own crisis resources, a review
under the Digital Personal Data Protection Act 2023, and possibly Indian
languages — each new region gets its own spec.

---

## DEC-11 — v1 centres on learning the Upaniṣad; Ask is kept for applying it

**Status:** Accepted

**Context.** As a problem-answering app, v1 would be opened mainly when something
goes wrong, which is episodic. The market research found structured programs and
daily practice drive retention and paid conversion, while excellent Māṇḍūkya
teaching is already free online. A learning-only app would compete with that
free teaching and lose the one thing it doesn't offer: applying the text to
*my* situation.

**Decision.** Combine the two, with learning at the centre:
- **Learn (home tab):** a Path of about 30 daily lessons through chapter 1 of
  Gauḍapāda's Kārikā (Āgama Prakaraṇa: the 12 mantras with 29 kārikās). Each
  lesson has the verse, a plain-language explanation, the related state, and a
  "live it today" prompt with an optional written response.
- **Pace:** one new lesson per calendar day; no streaks, no missed-day messages.
- **Lesson content is fixed and reviewed,** never generated. AI is used only in
  Ask.
- **Ask stays,** now secondary, and links to the lessons for its verses.
- **Journal** holds lesson responses and Ask reflections.
- **An optional daily reminder** (local, off by default) moves into v1.

**Why chapter 1.** It contains all twelve mantras of the Upaniṣad with
Gauḍapāda's commentary on them, so finishing it means having studied the whole
Upaniṣad through Gauḍapāda. It's about a month at one lesson a day. Chapters 2–4
(about 186 more kārikās) become follow-on Paths and the natural paid offering.

**Consequences.**
- **Content becomes the critical path:** a licensed translation of the chapter,
  proofreading of ~41 verses rather than 4, ~30 explanations written for the app,
  and review by someone grounded in the tradition (CR-6).
- **New designs** are needed for Learn, the reminder setting and the consent
  screen (CR-7).
- **Lower risk:** the Path is fully offline and sends nothing anywhere. Fewer
  raw-problem conversations means less overlap with mental-health topics.
- **Safety still covers what people write:** lesson responses get an on-device
  crisis check, and help is one tap away on every tab.
- **The audience leans further** toward Western Advaita seekers and diaspora
  learners, which fits the Australian launch (DEC-10).

**Revisit if** first-use testing shows people come for Ask and ignore the Path,
or the content cannot be reviewed in time. The fallback then is a shorter first
Path, not dropping learning.

---

## Open questions (not blocking; defaults noted)

- **Exact LLM model** — default Claude Sonnet 5.5; confirm or override.
- **Translation source** for chapter 1 and the corpus — now on the critical path
  (DEC-11). Do we have a preferred text, or should we research public-domain
  options (e.g. Nikhilananda) and confirm licensing?
- **Who writes and who reviews the lessons** (CR-6) — an author for the
  explanations and a reviewer grounded in the tradition.
- **Expo vs bare RN** — default Expo.
- **Analytics** — default: none in v1 (privacy purity). If we want anonymous
  counts later, they go in Postgres with no PII (DEC-5).
