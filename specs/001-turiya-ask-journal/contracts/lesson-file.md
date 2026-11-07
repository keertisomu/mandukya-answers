# Contract: Lesson file format

How lessons are written. This is the interface between the people who write and review lessons and
the app. A build check (`pnpm content:check`) rejects any file that breaks these rules (research
R10).

- **One file per lesson**: `content/path/karika-ch1/<order>-<slug>.md`, e.g.
  `07-the-fourth.md`.
- **Two parts**: YAML front matter for the fixed fields, then Markdown for the explanation.
- **Edits** go through pull requests, so every change is visible and can be reviewed.

## Example

```markdown
---
id: "1-07"
order: 7
title: "The fourth"
state: turiya              # jagrat | svapna | sushupti | turiya — or leave out
verses:
  - ref: "Māṇḍūkya 7"
    devanagari: "नान्तःप्रज्ञं न बहिष्प्रज्ञं …"
    iast: "nāntaḥprajñaṃ na bahiṣprajñaṃ …"
    pronunciation: "NAAN-tah-prag-yam na ba-hish-prag-yam …"
    translation: "Not inwardly aware, not outwardly aware …"
    source: "R. E. Hume, The Thirteen Principal Upanishads (1921), public domain"
prompt: "Once today, notice something you are aware of — and then notice the noticing."
reviewedBy: "Name of reviewer"
reviewedOn: 2027-01-15
---

The Upaniṣad has described waking, dreaming and deep sleep. Now it points at what is
present in all three …

(Plain-language explanation, about 3–5 minutes to read with the verse.)
```

*Verse text above is illustrative only; real text comes from the chosen translation after
proofreading.*

## Rules

| Field | Required | Rule |
|---|---|---|
| `id` | Yes | `"<chapter>-<order, 2 digits>"`; unique |
| `order` | Yes | Whole number; the Path has no gaps or repeats |
| `title` | Yes | Short, plain English |
| `state` | No | One of the four state keys |
| `verses` | Yes | One or more |
| `verses[].ref` | Yes | Unique across the Path |
| `verses[].devanagari` | Yes | Proofread against a Sanskrit source |
| `verses[].iast` | Yes | Standard IAST |
| `verses[].pronunciation` | Yes | Follows the scheme in `content/README.md` (research R11) |
| `verses[].translation` | Yes | From the chosen translation, word for word |
| `verses[].source` | Yes | Translation name, year and licence |
| `prompt` | Yes | A small reflection for the day. Never tells the person what to do about their life. |
| `reviewedBy`, `reviewedOn` | Yes for release | Set by the tradition reviewer. A release build fails if either is missing. |
| Body (Markdown) | Yes | The explanation. Our own commentary — never mixed into `translation`. |

## Writing guidance (for authors)
- Explain what the verse says **as Gauḍapāda reads it**, in plain words, and explain any Sanskrit
  term the first time it's used.
- Warm and steady; never negative, preachy or prescriptive (constitution, Principles I and II).
- No therapy or medical language.
- Australian English.
