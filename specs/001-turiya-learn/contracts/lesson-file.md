# Contract: Lesson file format

How lessons are written: the interface between the people who write and review lessons and the app.
A build check (`pnpm content:check`) rejects any file that breaks these rules (research R3, R4).

- **One file per lesson**: `content/path/karika-ch1/<order>-<slug>.md`, e.g.
  `09-mantra-7-not-the-knower.md`.
- **YAML front matter** holds the structured parts: phrases, verse, prompt. The **Markdown body is
  optional** and is used only for a longer introduction to the lesson.
- **Edits** go through pull requests, so every change can be seen and reviewed.

## Example — mantra 7, part 1 of 3

```markdown
---
id: "09"
order: 9
title: "Not the knower"
verse: "Māṇḍūkya 7"
part: { n: 1, of: 3, heading: "Not the knower" }
phrases:
  - devanagari: "नान्तःप्रज्ञं"
    iast: "nāntaḥprajñam"
    pronunciation: "naan-tah-prag-yam"
    meaning: "not aware of the inner world"
    note: >
      You are not the dreamer. The one who lives in a dream world is not
      what you most deeply are.
  - devanagari: "न बहिष्प्रज्ञं"
    iast: "na bahiṣprajñam"
    pronunciation: "na ba-hish-prag-yam"
    meaning: "not aware of the outer world"
    note: >
      You are not the waker either — the one busy with the world of objects.
reviewedBy: "Name of reviewer"
reviewedOn: 2027-01-15
---
```

*Text above is illustrative. Real Sanskrit, meanings and notes come from the authors and are
confirmed by the proofreader and reviewer.*

The **last part** of a verse (here, part 3 of 3) also carries the `ending`:

```yaml
ending:
  fullVerse:
    ref: "Māṇḍūkya 7"
    devanagari: "नान्तःप्रज्ञं न बहिष्प्रज्ञं … स आत्मा स विज्ञेयः ॥"
    iast: "nāntaḥprajñaṃ na bahiṣprajñaṃ … sa ātmā sa vijñeyaḥ"
    pronunciation: "naan-tah-prag-yam na ba-hish-prag-yam … sa aat-maa sa vig-yay-yah"
    translation: "…"          # from the chosen translation, word for word
    source: "Translator, title, year — licence"
  summary: >
    Not the dreamer, not the waker, not something in between, not the deep
    sleeper. Not something you can see, grasp or think. The one, peaceful,
    non-dual awareness in all three states. That is the Self; that is to be known.
  state: turiya
  prompt: "Once today, notice something you are aware of — then notice the noticing."
```

## Rules

| Field | Required | Rule |
|---|---|---|
| `id`, `order` | Yes | Unique; `order` has no gaps |
| `title` | Yes | Short, plain English |
| `verse` | Yes | e.g. `"Māṇḍūkya 7"` or `"Kārikā 1.10"` |
| `part` | If the verse spans lessons | `n` from 1 to `of`; `heading` is short and plain |
| `phrases` | Yes | One or more, in the verse's order |
| `phrases[].devanagari`, `.iast` | Yes | Proofread against a Sanskrit source |
| `phrases[].pronunciation` | Yes | Follows the scheme in `content/README.md` |
| `phrases[].meaning` | Yes | A few words |
| `phrases[].note` | Yes | Two or three plain sentences. **Our own words** — never copied from a copyrighted translation or commentary |
| `ending` | Last part only | Exactly one per verse |
| `ending.fullVerse.translation` | Yes | Quoted exactly from the chosen public-domain or permitted translation |
| `ending.fullVerse.source` | Yes | Translator, title, year and licence |
| `ending.summary` | Yes | A few plain sentences |
| `ending.state` | No | One of the four state keys |
| `ending.prompt` | Yes | A small reflection for the day. Never tells the person what to do about their life. |
| `reviewedBy`, `reviewedOn` | Yes for release | Set by the reviewer. A release build fails without them. |

## Writing guidance (for authors)

- **One idea per phrase.** Say what it points to, in everyday words. Explain any Sanskrit term the
  first time it appears.
- **Teachers' work as a guide only.** Gambhirananda, Nikhilananda, Chinmayananda and
  Sarvapriyananda can guide understanding, but their words are copyrighted. Write fresh notes; never
  paraphrase closely.
- **Tone**: warm and steady, never negative, preachy or prescriptive (constitution, Principles I
  and II).
- **No therapy or medical language.**
- **Australian English.**
- **Length**: about 3–5 minutes per lesson. If a verse runs longer, split it at its natural parts.
