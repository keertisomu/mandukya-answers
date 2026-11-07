# Quickstart: run and validate Turiya v1

How to run v1 locally and prove it meets the spec. Commands refer to scripts created in M0 (see
[plan.md](./plan.md)); they don't exist yet.

## Prerequisites

- **Node.js** 24 LTS and **pnpm**.
- **Xcode 26** (iOS simulator) and/or **Android Studio** (emulator).
- **An Expo development build** of the app. MMKV doesn't run in Expo Go (research R4).
- **Postgres with pgvector**: local (Docker), or a Neon development branch.
- **An Anthropic API key**, set on the **server only** as `ANTHROPIC_API_KEY`. It never goes in the
  app.
- **Maestro** for the end-to-end flows.

## Setup

```bash
pnpm install
pnpm content:check                   # every lesson file is valid (contracts/lesson-file.md)
pnpm --filter db migrate && pnpm --filter db seed
pnpm --filter api dev                # server on http://localhost:8787
pnpm --filter mobile ios             # or: android
```

## Automated checks

```bash
pnpm test                            # unit + component tests (Vitest, Jest)
pnpm --filter api test:contract      # /v1/answer matches contracts/answer-api.md
pnpm evals                           # golden eval set + crisis test set (real model calls)
pnpm e2e                             # Maestro flows below
```

All must pass. An eval regression blocks release (constitution, Principle VII).

## Validation scenarios

Each scenario maps to the spec's success criteria (SC) and requirements (FR). The data model and
API details are in [data-model.md](./data-model.md) and
[contracts/answer-api.md](./contracts/answer-api.md).

| # | Do this | Expect | Proves |
|---|---|---|---|
| 1 | Fresh install, airplane mode on, open the app | Lesson 1 opens. Verse shows Devanagari, IAST, pronunciation guide, translation and source | FR-001–003, FR-008, SC-003 |
| 2 | Finish lesson 1 with a response, tap "next" | Lesson 2 opens straight away; progress "2 of 30"; lesson 3 locked | FR-006, FR-007 |
| 3 | Finish lesson 2, close the app, reopen days later | Lesson 3 is next; no missed-day message | FR-006 |
| 4 | Open Journal | Lesson responses listed newest first; each opens and links to its lesson | FR-018–021, SC-012 |
| 5 | Force-quit and reopen | Journal and progress unchanged | FR-022, SC-012 |
| 6 | Write a crisis phrase as a lesson response (offline) | Response saved; support screen shows 000 and Lifeline; nothing sent (check with a proxy) | FR-029, FR-039, SC-010, SC-013 |
| 7 | Tap "Need to talk to someone?" on each tab (offline) | Support screen opens every time | FR-030 |
| 8 | Online, first Ask, **decline** consent | No request leaves the phone; Ask explains it needs consent; Learn still works | FR-036, SC-014 |
| 9 | Ask again, **agree**, submit a real problem | Thinking state, then 4 states in order with correct verses, one practice line, attribution line; saved to Journal | FR-012–015, FR-034, SC-006, SC-007 |
| 10 | Tap a verse in the reflection whose lesson you've reached | That lesson opens | FR-017 |
| 11 | Ask with crisis wording | Support screen, no reflection, nothing added to Journal | FR-028, SC-010 |
| 12 | Stop the server, then Ask | Built-in reflection shown and saved; no error screen | FR-033, SC-011 |
| 13 | Turn the reminder on for 2 minutes from now; don't do a lesson | Reminder arrives with generic text | FR-009 |
| 14 | Do a lesson, then wait for the reminder time | No reminder that day | FR-009 |
| 15 | Inspect outbound traffic (e.g. Proxyman) across scenarios 1–12 | Only `POST /v1/answer` bodies leave the phone, and only after consent | FR-035, SC-013 |
| 16 | Inspect server logs and the database after scenario 9 | No question or reflection text anywhere; only counts | FR-035 |
| 17 | Set the device to another language and region | App stays in Australian English with Australian services | FR-027 |

## Before release (manual)

- **Content**: every lesson has `reviewedBy` and `reviewedOn` (SC-002).
- **Support services**: every one has `lastVerified` within the last 90 days (SC-016).
- **Stores**: both list the app in Australia only (SC-017).
- **First-use testing** with 5–8 people: SC-001, SC-005, SC-008, SC-009 and SC-015.
