# Decisions

Why v1 is the way it is. Each entry: the decision, why, what it costs, and when to revisit.

| # | Decision | Status |
|---|---|---|
| DEC-1 | Personal data stays on the phone | Accepted |
| DEC-2 | React Native + Expo, one codebase | Accepted |
| DEC-3 | MMKV for on-phone storage | Accepted |
| DEC-4 | Server on the Vercel AI SDK; Mastra kept for later | Accepted |
| DEC-5 | Postgres holds only shared content | Accepted |
| DEC-6 | Anthropic Claude as the AI provider | Accepted |
| DEC-7 | Ask answers grounded in the Kārikā (RAG) | Accepted |
| DEC-8 | Crisis safety is in v1 and blocks launch | Accepted |
| DEC-9 | Retention through daily practice, not gamification | Accepted (updated by DEC-11) |
| DEC-10 | Launch in Australia only; India next | Accepted |
| DEC-11 | v1 centres on learning; Ask applies it | Accepted |

---

## DEC-1 — Personal data stays on the phone

**Decision**: the Journal, lesson responses and progress live only on the phone. There are no
accounts. The server never stores anything personal.

**Why**: people write their rawest problems here. With no central store, there is nothing to leak.
That makes the privacy promise true by design, not just by policy.

**Trade-off**:
- No sync, and no backup across devices. Losing the phone loses the Journal.
- The Ask question does pass through the server to reach the AI. It is used once and not stored,
  and the app says so.

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
- **Shape is enforced**: `generateObject` with a schema guarantees the four-state answer format.
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

**Cost**: the text needs a licensed translation and proofreading. This is the same work as the
lesson content (DEC-11).

## DEC-8 — Crisis safety is in v1 and blocks launch

**Decision**:
- A crisis check runs before any Ask reflection is written, and on lesson responses (on the phone).
- If it flags, the person sees Australian support services instead of a reflection.
- Every reflection gets an output safety check.
- Help is one tap away on every tab.
- The app makes no therapy claims, and states that it is not religious authority.

**Why**: people bring stress, anger and loneliness. Gita chatbots have been caught condoning
violence. Illinois banned AI therapy in 2025, and Apple rejects apps that read as medical advice.
Leaving safety out would be the biggest risk to the product, and adding it is cheap.

**Privacy**: crisis questions are never saved, logged or followed up. The server keeps only a
count.

## DEC-9 — Retention through daily practice, not gamification

**Decision**:
- **v1**: the daily lesson Path is the main reason to return, with an optional reminder (DEC-11).
- **v1.1**: follow-up check-ins and the pattern mirror deepen Ask (see ROADMAP).
- **Not used**: streaks, community and open-ended chat.

**Why**: meditation apps keep only about 6–8% of users after 30 days. The research found daily
practice with gentle reminders, and structured depth, work best. Streaks punish missed days, which
clashes with the teaching.

**Measure**: week-4 and day-30 retention from the app stores.

## DEC-10 — Launch in Australia only; India next

**Decision**: v1 is offered only in Australian app stores. India comes in v2, then other countries,
each with its own spec.

**Why**: one market means one set of crisis services, one privacy law (the Privacy Act 1988) and one
language. That keeps the first release's safety and compliance work manageable. Australia's Indian
community is one of its largest overseas-born communities.

**Trade-off**: the research pointed to the US, UK and Canada as bigger markets; Bible Chat earns over
95% of its revenue in the US. Australia is a deliberately careful first step.

**Consequences**:
- Australian support services, checked every 90 days.
- A notice and consent before the first Ask, because questions are processed overseas.
- Australian English.
- Helplines stored per region.

## DEC-11 — v1 centres on learning; Ask applies it

**Decision**:
- **Learn is the home tab**: a Path of about 30 daily lessons through chapter 1 of the Kārikā (the 12
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

---

## Open questions

| Question | Default if not decided |
|---|---|
| Which translation of chapter 1, and its licence? | Research public-domain options first. This is on the critical path. |
| Who writes the lesson explanations, and who reviews them? | Needs a person. Blocks release. |
| Which model for reflections? | `claude-sonnet-5-5` |
| Expo or bare React Native? | Expo |
| Any analytics in v1? | None. Only anonymous server counts. |
