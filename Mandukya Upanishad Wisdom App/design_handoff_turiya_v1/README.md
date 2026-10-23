# Handoff: Turiya — Māṇḍūkya guidance app (v1)

## Overview
A calm mobile app where a user types any everyday problem (relationship, stress, anger, a decision) and gets an answer that looks at it through the **four states of consciousness** of the Māṇḍūkya Upanishad, with a verse from **Gauḍapāda's Māṇḍūkya Kārikā** attached to each state. Every answer is auto-saved to a read-only **Journal** so the user can scroll back through their history. The experience goal is that the app feels quieting the moment it opens.

v1 scope is deliberately minimal: **Ask** and **Journal**, nothing else. (Śaṅkara's commentary, a verse library, daily practice, streaks and community are explicitly deferred to v2 — do not build them.)

## About the Design Files
The file in this bundle (`Turiya.dc.html`) is a **design reference created in HTML** — a working prototype showing the intended look and behavior. It is **not production code to copy directly**. The task is to **recreate this design in the target codebase's environment** (React Native / Swift / Flutter / etc.) using that project's established patterns, component library, and networking. If no codebase exists yet, pick the most appropriate mobile framework and implement there.

`Turiya.dc.html` uses a proprietary custom runtime (`<x-dc>`, `support.js`, `ios-frame.jsx`); **none of that runtime should be reproduced**. Only the design — layout, type, color, copy, flow, and the AI contract below — carries over. The iPhone bezel is a preview device frame, not part of the app.

## Fidelity
**High-fidelity.** Final colors, typography, spacing, copy, and interactions. Recreate the UI faithfully using the codebase's existing libraries. Exact tokens are listed under Design Tokens.

## LLM contract (the core of the app)
The answer is produced by one LLM call. The four states are **fixed** and always rendered in this order; the model fills only heading + body per state, plus one practice line. The Kārikā verse for each state is **hard-coded in the app** (never model-generated) so citations can't hallucinate.

- **System prompt** (verbatim from the prototype):
  > You are a warm, plain-spoken friend grounded in Gauḍapāda's Māṇḍūkya Kārikā (the commentary on the Mandukya Upanishad). You never use jargon without explaining it, never moralize, never say 'as an AI'. Answer the person's everyday problem through the four states of consciousness: jagrat (waking — the bare observable facts), svapna (dream — the story/projection they are running on top of the facts), sushupti (deep sleep — the rest and wholeness that remains when the problem is absent), turiya (the witness that is aware of all three and is untouched). Be concrete about THEIR specific situation, never generic. 45-70 words per level. Reply with ONLY valid JSON: {"levels":[{"heading":string,"body":string} x4 in that order],"practice":string}. heading: a short plain-English line, max 8 words. practice: one small concrete thing to do today, max 25 words.
- **User message**: the raw problem text.
- **Model**: any capable chat model; `max_tokens` ≈ 1400.
- **Parsing**: extract the first `{...}` JSON block from the response; require `levels` to be an array of length ≥ 4 or treat as failure.
- **Failure fallback**: if the call fails or JSON is invalid, show a built-in generic four-level answer (see `fallback()` in the prototype source) so the user never hits a dead end.
- **API key**: the prototype uses a sandbox helper; in production call your own LLM backend — never ship a key in the client.

### Fixed state → verse map (hard-coded)
| Order | Key | Sanskrit | Label | Kārikā ref | Devanagari | English |
|---|---|---|---|---|---|---|
| 1 | jagrat | Jāgrat | Waking | Gauḍapāda · Kārikā 1.1 | विश्वो हि स्थूलभुङ् नित्यं तैजसः प्रविविक्तभुक् । | Vishva, the waking self, feeds on the gross; Taijasa on the subtle. |
| 2 | svapna | Svapna | The story | Gauḍapāda · Kārikā 2.1 | वैतथ्यं सर्वभावानां स्वप्न आहुर्मनीषिणः । | The wise call everything seen in dream unreal — it lives only within. |
| 3 | sushupti | Suṣupti | Underneath | Gauḍapāda · Kārikā 1.2 | आनन्दभुक्तथा प्राज्ञस्त्रिधा भोगं निबोधत । | Prajna, in deep sleep, feeds on bliss. Know experience to be threefold. |
| 4 | turiya | Turīya | The witness | Gauḍapāda · Kārikā 1.10 | निवृत्तेः सर्वदुःखानामीशानः प्रभुरव्ययः । | Turiya: the end of all sorrow, the lord, imperishable, non-dual, shining. |

State accent dots, in order: `#C7A87E`, `#A98BA8`, `#7E9AA8`, `#3F4F52`.

> Verify the Devanagari and references with a Sanskrit source before launch — they are plausible but should be proof-read by someone who reads the text.

## Screens / Views

### 1. Ask (home, default tab)
- **Purpose**: the user states a problem and submits it.
- **Layout**: full-height column. Scrollable content area, padding `58px 22px 12px` (top padding leaves room under the status bar); fixed bottom tab bar. A soft radial "glow" blob sits top-left behind content (decorative, `pointer-events:none`).
- **Components (top to bottom)**:
  - **Greeting** eyebrow — time-based ("Good morning/afternoon/evening", "Late night"). 11px, uppercase, letter-spacing .14em, color `#9A9186`.
  - **Title** — "What's sitting with you today?" Newsreader 300, 34px, line-height 1.18, letter-spacing -.01em, color `#23201C`.
  - **Subhead** — explains the four-state framing. 13.5px, line-height 1.55, color `#7A7268`, max-width 300px.
  - **Input card** — bg `#FFFDF9`, 1px border `rgba(35,32,28,.09)`, radius 20px, padding `16px 16px 12px`, shadow `0 1px 2px rgba(35,32,28,.04), 0 12px 30px -22px rgba(35,32,28,.4)`. Contains a 4-row transparent textarea (15.5px, placeholder example) and a footer row: left note "Saved only on this device" (11px `#A79D90`), right primary button **"Look at it"** (bg `#23201C`, text `#F7F4EE`, 13.5px/600, padding `11px 18px`, radius 999px, min-height 44px).
  - **Suggestion chips** — flex-wrap, gap 8px: "A relationship", "Stress", "Anger", "A decision", "Loneliness". Each: 1px border `rgba(35,32,28,.12)`, bg `rgba(255,253,249,.7)`, 12.5px `#5C554C`, padding `9px 13px`, radius 999px. Tapping a chip pre-fills the textarea with a first-person sentence (see `chips` in source).
  - **Four-states primer** — section under a top divider. Eyebrow "The four states". Four rows, each an accent dot + "<Sanskrit> — <gloss>". Glosses: Jāgrat = "the waking self, meeting the solid world of facts"; Svapna = "the inward dream-mind, building a world from memory"; Suṣupti = "deep sleep, where all of it is gathered and quiet"; Turīya = "the fourth — aware of the other three, untouched by them".

### 2. Thinking (transient, within Ask)
- Shown while the LLM call is in flight. Centered column, min-height ~640px: a breathing radial halo (132px, `breathe` 6s ease-in-out infinite) with a `ॐ` (Noto Serif Devanagari, 40px, `#8A6A52`), and a rotating italic status line (Newsreader italic, 17px, `#7A7268`) cycling every 1.6s through: "sitting with it…", "looking at the surface…", "listening underneath…", "finding the verse…".

### 3. Answer (within Ask, after response)
- **Purpose**: present the four-state reading; it auto-saves to Journal.
- **Components**:
  - Back link "← Ask something else" (13px `#8A6A52`) — returns to a cleared Ask screen.
  - The user's question echoed — Newsreader 21px, line-height 1.35.
  - Caption "Four levels, from the surface inward" (12px `#A79D90`).
  - **Four accordion cards** (one per state), gap 10px. Card: bg `#FFFDF9`, 1px border `rgba(35,32,28,.09)`, radius 18px. Header button (min-height 44px): accent dot + eyebrow "<Sanskrit> · <Label>" (10.5px uppercase `#A79D90`) + model heading (Newsreader 17.5px `#23201C`) + caret ("–"/"+"). Expanded body: paragraph (14.5px, line-height 1.65, `#4A443D`) then a verse chip — bg `#F2EEE6`, radius 12px, padding `12px 14px`: Devanagari line (Noto Serif Devanagari 14.5px `#3A342D`), italic English (Newsreader italic 13.5px `#7A7268`), then the ref eyebrow (10.5px uppercase `#A79D90`). **First card open by default.**
  - **Practice card** — dark `#2E3A3C`, text `#EDE9E1`, radius 18px, padding 18px: eyebrow "One thing to try" + practice line (Newsreader 18px).
  - Footer note "Saved to your journal" (centered, 12px `#A79D90`).
  - Entry animates in with `fadeup` (.5s).

### 4. Journal (second tab)
- **Purpose**: read-only history of every past interaction, newest first. No re-asking.
- **Components**: title "Journal" (Newsreader 300, 30px) + count line ("N entries · yours only", or "Your history"). **Empty state**: faded `ॐ` + "Nothing here yet. Whatever you ask will be kept here for you to return to." Otherwise a column of accordion cards (gap 10px): header shows the timestamp eyebrow + the question (Newsreader 17px) + caret. Expanded: the four saved levels (dot + eyebrow + heading + body + ref, each separated by a 1px top divider) and the dark practice card at the bottom.

## Interactions & Behavior
- **Submit**: trim input; if empty, no-op. Go to Thinking, start status cycle, call LLM. On resolve/fail, stop cycle, build an entry, prepend to journal, persist, show Answer.
- **Navigation**: two bottom tabs, Ask and Journal. Switching to Ask lands on the last Answer if one exists, else the empty Ask form.
- **Accordions**: independent open/close per card (answer + journal). Caret toggles "+"/"–".
- **Timestamps**: "Today · h:mm", "Yesterday · h:mm", "N days ago" (<7), else "Mon D".
- **Animations**: `breathe` (halo), `fadeup` (answer entry), `driftglow` (background blob opacity). All subtle; respect `prefers-reduced-motion` in production.

## State Management
- `tab`: "ask" | "journal"
- `phase`: "idle" | "thinking" | "answer" (Ask sub-state)
- `question` (string), `asked` (submitted string), `answer` (parsed JSON or fallback)
- `open` (map of answer-card index → bool), `jopen` (map of entry id → bool)
- `journal` (array of entries), `thinkIdx` (status-line cycle)
- **Persistence**: journal is read from and written to local storage under key `turiya_journal_v1` (JSON array). Entry shape: `{ id (ms timestamp), when (string), question, levels:[{name,sanskrit,tint,heading,body,verseRef}], practice }`. In production prefer the platform's secure/local store; this is private, device-only data in v1 (no accounts, no server).

## Design Tokens
- **Fonts**: Newsreader (display/serif, weights 300/400/500, + italic), Karla (UI/sans, 400/500/600), Noto Serif Devanagari (Sanskrit, 400/500). All Google Fonts.
- **Colors**: app bg `#F7F4EE`; page/desk bg `#EDE9E1`; ink `#23201C`; body text `#4A443D`; muted `#7A7268`; faint `#9A9186` / `#A79D90`; card `#FFFDF9`; card border `rgba(35,32,28,.09)`; verse chip `#F2EEE6`; dark panel `#2E3A3C` on `#EDE9E1` text; accent/brand `#8A6A52` (hover `#6E5240`); state dots `#C7A87E` `#A98BA8` `#7E9AA8` `#3F4F52`.
- **Radius**: cards 18–20px; verse/practice inner 12px; chips/buttons 999px.
- **Shadow (input card)**: `0 1px 2px rgba(35,32,28,.04), 0 12px 30px -22px rgba(35,32,28,.4)`.
- **Hit targets**: ≥ 44px.
- **Type sizes**: title 34 / section title 30 / question echo 21 / card heading 17.5 / body 14.5–15.5 / eyebrow 10.5–11.

## Assets
None. No images or icon files — the only glyph is the Unicode `ॐ` (U+0950). Fonts load from Google Fonts.

## Files
- `Turiya.dc.html` — the full prototype (template markup + logic class). Read the logic class for the exact `fallback()` text, chip pre-fill sentences, and timestamp logic. Ignore the `<x-dc>`/`support.js`/`ios-frame.jsx` runtime — design reference only.
