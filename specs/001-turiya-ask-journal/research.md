# Research: Turiya v1

Phase 0 output of `/speckit-plan`. Each item: **Decision**, **Why**, **Alternatives considered**.

Already decided in [docs/DECISIONS.md](../../docs/DECISIONS.md) and not re-researched here:
- React Native + Expo (DEC-2);
- on-phone storage (DEC-1, DEC-3);
- a TypeScript server on the Vercel AI SDK (DEC-4);
- Postgres + pgvector (DEC-5);
- Anthropic Claude (DEC-6);
- RAG grounding (DEC-7);
- crisis safety (DEC-8);
- Australia first (DEC-10);
- learning-first (DEC-11).

---

## R1 — Structured AI output

- **Decision**: AI SDK 6 `generateText` with `output: Output.object({ schema })`, using the shared
  Zod answer schema.
- **Why**: in AI SDK 6, `generateObject` is deprecated in favour of `generateText` with an `output`
  setting. It still works, but new code should not depend on it.
- **Alternatives**: `generateObject` (deprecated); the Anthropic SDK directly with structured
  outputs (works, but loses the provider-swap ability DEC-4 relies on).

## R2 — Embeddings for search

- **Decision**: run a small open embedding model **on our own server**: `bge-small-en-v1.5` via
  `@huggingface/transformers` (384 dimensions, mean pooling, normalised, with the model's
  recommended query prefix).
- **Why**:
  - Anthropic has no embeddings API.
  - A hosted embeddings API (e.g. Voyage, OpenAI) would send every question to a **second
    company**. That widens the privacy story, and the consent screen (FR-036) would have to name
    them too.
  - Self-hosting keeps Anthropic as the only outside party, and costs nothing per call.
  - The corpus is small (a few hundred passages), so a small model is enough.
- **Alternatives**:
  - Voyage AI: better quality, but a second processor of user questions.
  - OpenAI embeddings: same problem.
  - Keyword-only search: breaks the constitution's hybrid-search rule.

## R3 — Hybrid retrieval

- **Decision**: run two searches and merge them with reciprocal rank fusion:
  - pgvector cosine search (HNSW index);
  - Postgres full-text search (`tsvector`).
  - A small synonym list maps spellings: *turiya / turīya / the fourth*, *jagrat / jāgrat /
    waking*, and so on.
- **Why**: the constitution requires vector **plus** keyword search, so exact Sanskrit terms always
  find their verses. Reciprocal rank fusion is simple, needs no tuning data, and suits a small
  corpus.
- **Alternatives**: weighted score blending (needs tuning); a re-ranking model (overkill at this
  size).

## R4 — App runtime and storage

- **Decision**: Expo SDK 58 (React Native 0.88, New Architecture), or the latest stable SDK at M0.
  `react-native-mmkv` for storage, with an **Expo development build** (EAS) from M1.
- **Why**: SDK 58 is current (October 2026), and since SDK 55, Expo runs only on the New
  Architecture. MMKV is a native module, so it doesn't run in Expo Go. Development builds are the
  normal workflow anyway.
- **Alternatives**: `AsyncStorage`, which runs in Expo Go but is slower and asynchronous. It's
  kept as the fallback (DEC-3).

## R5 — Navigation

- **Decision**: Expo Router with a tab layout (Learn, Ask, Journal), plus stack screens for a
  lesson, the support screen and the consent screen.
- **Why**: it's Expo's default router. File-based routes keep the screen list obvious, and deep
  links (from v1.1 notifications) work out of the box.
- **Alternatives**: React Navigation configured by hand (more boilerplate for the same result).

## R6 — Daily reminder that skips days

- **Decision**: use `expo-notifications` local notifications:
  - schedule **one-off** notifications for the next 7 days at the chosen time;
  - when a lesson is completed, cancel today's;
  - each time the app opens, top the schedule back up to 7 days;
  - when the Path is finished, cancel everything.
- **Why**: a repeating daily trigger can't skip a day, and local notifications can't run logic when
  they fire. Scheduling a week ahead covers people who don't open the app for a while.
- **Alternatives**:
  - a repeating daily trigger: can't honour "skip if a lesson was done today";
  - server push: needs push tokens and a server, which breaks device-only (DEC-1).

## R7 — Crisis detection

- **Decision**: two layers.
  1. **Word and phrase lists** in `packages/shared`, used on the phone (lesson responses) and as
     the server's first gate (Ask). They handle negation and an idiom allow-list.
  2. **Server only**: a model check with `claude-haiku-5-5` that returns
     `{ crisis: boolean, category }`, tuned to flag when unsure.
- **Building the lists**: draft them, then review them with someone experienced in crisis
  support. Test against the crisis test set, which includes idioms such as "this job is killing me"
  and "I could kill for a coffee".
- **Why**: lesson responses must never leave the phone (FR-029), so their check has to be local.
  For Ask, the model check catches indirect wording that word lists miss.
- **Alternatives**:
  - an on-device ML classifier: adds app size, and no reliable small model was found for this;
  - the model check alone: fails when offline or when the server is down.

## R8 — Server framework and hosting

- **Server decision**: Hono on Node.js 24 LTS, in a container.
- **Hosting decision**, made at M4 between two candidates:
  - **Fly.io** (Sydney region `syd`) + **Neon** Postgres (Sydney, `aws-ap-southeast-2`, with
    pgvector);
  - **AWS** or **Google Cloud** Sydney regions, with their managed Postgres.
- **Why**:
  - Hono is small and TypeScript-first, and runs on any host.
  - The database holds no personal data, and Anthropic processes questions outside Australia
    anyway. So the Sydney region is about speed for Australian users, not compliance.
  - Neon's Sydney region supports Postgres, and pgvector is a standard extension there.
  - Fly.io's Sydney region had several networking incidents in 2026, mostly between regions. A
    single-region setup is less exposed, but check its status history before choosing.
- **Alternatives**: Fastify or Express (both fine; Hono is lighter); serverless functions (cold
  starts while the embedding model loads, which is a poor fit).

## R9 — Testing

- **Decision**:
  - **Vitest** for `packages/shared` and the server;
  - **Jest** with React Native Testing Library for the app;
  - **Maestro** for end-to-end flows on simulators;
  - an **eval runner** script for the golden and crisis sets, run in CI on every prompt, model or
    content change.
- **Why**: each is the standard choice for its layer. Maestro flows are plain YAML and easy to read.
  The eval runner is what the constitution's quality gate requires (Principle VII).
- **Cost note**: a full eval run is about 40 golden plus a similar number of crisis calls, a few
  cents per run.
- **Alternatives**: Detox, which is heavier to set up than Maestro.

## R10 — Lesson file format

- **Decision**: one Markdown file per lesson, with YAML front matter for the structured fields and
  Markdown for the explanation. A build step validates each file with Zod and bundles them into a
  JSON module. The format is in [contracts/lesson-file.md](./contracts/lesson-file.md).
- **Why**: the authors and reviewers aren't developers. Markdown is readable in a GitHub pull
  request and easy to comment on. The build step catches mistakes before they reach the app.
- **Alternatives**:
  - JSON files: hard for non-developers to edit;
  - a headless CMS: another service and login, which is overkill for about 30 lessons.

## R11 — Pronunciation guide scheme

- **Decision**: one simple English respelling, documented in `content/README.md`:
  - syllables split by hyphens;
  - long vowels doubled (*aa*, *ee*, *oo*);
  - aspirated consonants written with *h* (*bh*, *dh*).
- **Why**: beginners can read it without learning IAST. One scheme keeps every lesson consistent,
  and the Sanskrit proofreader checks it with the verse (SC-002).
- **Alternatives**: IAST only (unreadable to most beginners); audio only (planned for later, see
  ROADMAP).

## R12 — Fonts offline

- **Decision**: bundle Newsreader, Karla and Noto Serif Devanagari with `@expo-google-fonts/*`.
- **Why**: fonts load from the app bundle, so lessons render correctly offline (FR-008).
- **Alternatives**: loading fonts from Google at runtime, which fails offline.

## R13 — Choosing the reflection model

- **Decision**: start with `claude-sonnet-5-5`. Run the golden eval set on `claude-haiku-5-5` too,
  and use the **cheapest model that passes**.
- **Why**: the constitution favours the cost-effective tier, with "reliably stays reflective, not
  advising" as the deciding test.
- **Alternatives**: fixing on one model without measuring.
