# Decisions

Why Turiya is built the way it is. Each entry: the decision, why, what it costs, and when to revisit.

| # | Decision | Status |
|---|---|---|
| DEC-1 | Personal data stays on the phone | Accepted |
| DEC-2 | React Native + Expo, one codebase | Accepted |
| DEC-3 | MMKV for on-phone storage | Accepted |
| DEC-4 | Server on the Vercel AI SDK; Mastra kept for later | Accepted — applies from v2 (Ask) |
| DEC-5 | Postgres holds only shared content | Accepted — applies from v2 (Ask) |
| DEC-6 | Anthropic Claude as the AI provider | Accepted — applies from v2 (Ask) |
| DEC-7 | Ask answers grounded in the Kārikā (RAG) | Accepted — applies from v2 (Ask) |
| DEC-8 | Crisis safety is in v1 and blocks launch | Accepted (updated by DEC-12) |
| DEC-9 | Retention through daily practice, not gamification | Accepted (updated by DEC-12) |
| DEC-10 | Launch v1 in Australia and India | Accepted (updated 2026-11-24) |
| DEC-11 | v1 centres on learning; Ask applies it | Superseded by DEC-12 |
| DEC-12 | v1 is learning only; Ask moves to v2 | Accepted |
| DEC-13 | Lesson and Support choices from the prototype | Accepted |

---

## DEC-1 — Personal data stays on the phone

**Decision**: the Journal, lesson responses and progress live only on the phone. There are no
accounts. The server never stores anything personal.

**Why**: people write their rawest problems here. With no central store, there is nothing to leak.
That makes the privacy promise true by design, not just by policy.

**Trade-off**:
- No sync, and no backup across devices. Losing the phone loses the Journal.
- In v1 nothing leaves the phone at all. From v2, the Ask question passes through the server to
  reach the AI. It is used once and not stored, and the app asks for consent first.

**Revisit if** users ask for backup. The preferred answer is an opt-in encrypted backup to their own
iCloud or Google Drive, which still needs no account with us.

## DEC-2 — React Native + Expo, one codebase

**Decision**: one React Native app for iOS and Android, built with Expo.

**Why**: the design is mobile-first. A single TypeScript codebase serves both stores and matches the
TypeScript server. Expo is the fastest route to a running app.

**Revisit if** v1 becomes iOS-only (SwiftUI is then an option) or a web version is wanted first.

## DEC-3 — MMKV for on-phone storage

**Decision**: store the Journal, progress and settings as JSON in `react-native-mmkv`, under
versioned keys.

**Why**: it's fast, simple and synchronous — a good fit for a few small JSON documents.
`AsyncStorage` is an acceptable fallback.

## DEC-4 — Server on the Vercel AI SDK; Mastra kept for later

**Decision**: a thin TypeScript server using the Vercel AI SDK with Anthropic, over Postgres with
pgvector.

**Why**:
- **Small corpus**: the Kārikā is small and doesn't change, so a full agent framework isn't needed
  yet.
- **Shape is enforced**: structured output with a schema (AI SDK 6 `generateText` +
  `Output.object`) guarantees the four-state answer format.
- **One database**: pgvector keeps search inside the Postgres we already have.

**Alternative**: Mastra, which is free, open source and self-hostable. It adds workflows, evals and
tracing, and is built on the same SDK, so adopting it later doesn't mean a rewrite.

**Revisit when** v2 brings multi-step flows, or we want built-in tracing and evals.

## DEC-5 — Postgres holds only shared content

**Holds**: the Kārikā corpus with its search indexes, and anonymous daily counts.

**Never holds**: questions, reflections, journal entries, or anything that identifies a person.

## DEC-6 — Anthropic Claude as the AI provider

**Decision**: Anthropic's API through `@ai-sdk/anthropic`.

| Use | Model | Price per 1M tokens (in / out) |
|---|---|---|
| Reflections | `claude-sonnet-5-5` | $2 / $10 |
| Higher quality (option) | `claude-opus-5-5` | $4 / $20 |
| Safety checks | `claude-haiku-5-5` | $0.10 / $0.50 |

**Why**: Anthropic's API does not train on customer inputs, which is what "no model training"
requires. Claude also follows the warm, non-advising tone well.

**Note**: model IDs are config, so switching models needs no code change.

## DEC-7 — Ask answers grounded in the Kārikā (RAG)

**Decision**: every Ask reflection is grounded in passages retrieved from a vetted text of
Gauḍapāda's Kārikā. Retrieval is hybrid: vector plus keyword, as the constitution requires. The
four displayed verses are fixed in the app.

**Why**: the product's premise is Gauḍapāda's commentary, not the model's loose recall. Fixed verses
mean citations can't be invented.

**Privacy**: search embeddings are computed on our own server with a small open model, so Anthropic
is the only outside company that sees a question (ASK_V2_DESIGN.md §4).

**Cost**: the text needs a licensed translation and proofreading. This is the same work as the
lesson content (DEC-11).

## DEC-8 — Crisis safety is in v1 and blocks launch

**Decision**:
- **v1**: journal replies are checked for crisis wording on the phone. If it's found, the reply is
  kept and mental health support for the phone's region opens. A small "Support" item in every
  header keeps it one tap away. The app makes
  no therapy claims and states that it is not religious authority.
- **v2 (Ask)**: a crisis check before any reflection is written, and an output safety check on every
  reflection (see ASK_V2_DESIGN.md).

**Why**: people bring stress, anger and loneliness. Gita chatbots have been caught condoning
violence. Illinois banned AI therapy in 2025, and Apple rejects apps that read as medical advice.
Leaving safety out would be the biggest risk to the product, and adding it is cheap.

**Privacy**: in v1 the check happens entirely on the phone. In v2, crisis questions are never
saved, logged or followed up; the server keeps only a count.

## DEC-9 — Retention through daily practice, not gamification

**Decision**:
- **v1**: the lesson Path and saved lessons are the reasons to return. A lesson a day is suggested
  and helped by an optional reminder, but people can go at their own pace (DEC-12).
- **After Ask ships (v2)**: follow-up check-ins and the pattern mirror deepen Ask (see ROADMAP).
- **Not used**: streaks, community and open-ended chat.

**Why**: meditation apps keep only about 6–8% of users after 30 days. The research found daily
practice with gentle reminders, and structured depth, work best. Streaks punish missed days, which
clashes with the teaching.

**Measure**: week-4 and day-30 retention from the app stores.

## DEC-10 — Launch v1 in Australia and India

**Decision**: v1 is offered in the Australian and Indian app stores only. Other countries come
later, each with its own spec.

**History**: first decided as Australia only (2026-10-24), when v1 included Ask. Each country then
needed a review of AI data leaving the country, plus its own consent screen. Changed on 2026-11-24.

**Why both now**:
- With no server, AI or network in v1 (DEC-12), a second country adds very little: its own support
  services, store availability and privacy-policy wording.
- India is the Upaniṣad's home audience, and the most likely place to find the lesson reviewer and,
  later, a chanter.
- Australia's Indian community is one of its largest overseas-born communities.

**Trade-off**: the research pointed to the US, UK and Canada as the biggest paying markets. v1 is
free, so that matters less now.

**Consequences**:
- Support services for both countries, chosen by the phone's region and switchable, checked every 90
  days.
- English with British/Commonwealth spelling, which both countries share. Indian languages come
  later.
- Testing on low-cost Android phones.
- The privacy policy covers both countries. v1 processes no personal data, so the Privacy Act 1988
  and the DPDP Act 2023 have little to apply to.
- With v2, a notice and consent before the first Ask, because questions are processed overseas.

## DEC-11 — v1 centres on learning; Ask applies it

**Status**: superseded by DEC-12 (Ask moved out of v1). Kept for the record.

**Decision**:
- **Learn is the home tab**: a Path of about 30 short lessons through chapter 1 of the Kārikā (the 12
  mantras with 29 kārikās).
- **Lesson content is fixed and reviewed**, never generated.
- **Ask stays** as the second tab and links to the lessons.
- **The Journal** holds both lesson responses and Ask reflections.

**Why**:
- As a problem-answering app alone, Turiya would be opened only on bad days.
- Structured programs and daily practice drive retention and paid conversion.
- A learning-only app would compete with excellent free teaching online. Applying the text to *my*
  situation is what those don't offer.

**Why chapter 1**: it holds the whole Upaniṣad with Gauḍapāda's commentary on it — about a month of
lessons. Chapters 2–4 become later Paths.

**Trade-offs**:
- Content becomes the critical path: a translation, proofreading of about 41 verses, about 30
  explanations, and review by someone grounded in the tradition.
- Learn needs new designs.

**Benefits**:
- Lower risk. Learn is offline and sends nothing anywhere.
- Fewer raw-problem conversations.

**Revisit if** testing shows people ignore the Path and only use Ask, or the content can't be
reviewed in time. The fallback is a shorter first Path, not dropping learning.

## DEC-12 — v1 is learning only; Ask moves to v2

**Decision**: v1 teaches the text and nothing else.
- **Lessons**: phrase-by-phrase lessons on the Upaniṣad's 12 mantras. Gauḍapāda's kārikās appear
  inside the lesson for the mantra they explain — all 29 from chapter 1, one featured (DEC-13), the
  rest under "More from Gauḍapāda". The Upaniṣad is the subject; the Kārikā is the guide.
- **Saved lessons**: kept at the top of Learn, for daily re-reading.
- **Journal**: replies to lesson prompts.
- **Ask moves to v2**, and its design is kept in [ASK_V2_DESIGN.md](./ASK_V2_DESIGN.md).

**Why**:
- The product owner's own practice is to read one key verse, such as mantra 7, every morning,
  phrase by phrase. That is the experience to perfect first.
- The purpose of the app is learning. Ask can come once the teaching is in place to ground it.

**Benefits**:
- **No server, no AI, no network.** v1 is far simpler to build and run, costs almost nothing to
  operate, and makes a strong privacy promise: nothing leaves the phone. The App Store label is
  "Data Not Collected".
- **No AI consent screen** and no golden eval set until v2.
- **Less mental-health exposure**: people aren't bringing problems to an AI.

**Trade-offs**:
- Lesson content is the whole product, so the critical path is even more about writing and review.
- The phrase-by-phrase format means more content per verse.
- Ask's appeal on hard days waits until v2.

**Lesson format**: modelled on a phrase-by-phrase study of mantra 7 (19 phrases in 3 parts). The
format is followed, but meanings and notes are written for the app, because modern translations and
teachers' commentaries are still in copyright.

**Revisit if** testing shows people want help applying the text sooner than v2.

## DEC-13 — Lesson and Support choices from the prototype

**Context**: a clickable prototype of a mantra 7 lesson (three layouts) was reviewed on 2026-11-24.

**Decisions**:
- **One kārikā at a time.** Exactly one featured kārikā per mantra under "What Gauḍapāda says".
  The rest are under "More from Gauḍapāda", shown one at a time with Previous and Next. Two kārikās
  side by side felt overwhelming.
- **"Support" in the header.** A small, muted item on every screen replaces the "Need to talk to
  someone?" link. It is still one tap away, but quieter on a calm lesson screen.
- **Support has two options**:
  - **Mental health support**, for the phone's region (Australia or India);
  - **Give feedback**, through the person's own email app.

  Turiya itself still sends nothing, so v1 keeps its "nothing leaves the phone" promise.

**Still open**: which of the three lesson layouts to build — A (study page), B (one phrase at a
time) or C (verse first) — or a mix.

---

## Open questions

| Question | Default if not decided |
|---|---|
| Which translations of the 12 mantras and of Kārikā chapter 1, and their licences? | Research public-domain options first. This is on the critical path. For the 12 mantras, Hume's *Thirteen Principal Upanishads* (1921) is a likely public-domain choice: Hume died in 1940, so it should be free in Australia and India too, but confirm. It does **not** include Gauḍapāda's Kārikā, which needs its own translation. Newer translations are usually still in copyright, even though the text is ancient. |
| Who writes the lesson meanings and notes, and who reviews them? | Needs a person. Blocks release. |
| Feedback email address? | Needs the product owner. |
| Lesson layout: A, B, C or a mix? | Not decided (DEC-13). |
| Which model for Ask reflections (v2)? | `claude-sonnet-5-5`, or the cheapest model that passes the golden eval |
| Expo or bare React Native? | Expo |
| Any analytics in v1? | None. v1 sends nothing. |
