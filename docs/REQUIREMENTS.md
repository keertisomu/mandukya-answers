# Requirements — v1

**Product:** Turiya — a calm mobile app that answers an everyday problem through
the four states of consciousness of the Māṇḍūkya Upaniṣad, grounded in
**Gauḍapāda's Māṇḍūkya Kārikā** (the commentary), with one verse attached per
state.

**v1 scope is deliberately minimal: two screens — Ask and Journal — and nothing
else.** Śaṅkara's commentary, a verse library, daily practice, streaks, and
community are **explicitly deferred to v2** (see Non-goals).

Decisions referenced below (`DEC-n`) are recorded in [DECISIONS.md](./DECISIONS.md).

---

## 1. Users & principles

- A single person, using the app privately, with no account and no sign-in.
- **Privacy by construction** (DEC-1): their problems and answers never leave the
  device except to be processed once by the LLM, and are never stored on our
  server.
- **Calm, never clinical.** The app should feel quieting the moment it opens. It
  offers reflection, not advice or therapy.
- **Faithful to the commentary** (DEC-7): answers interpret the user's situation
  through Gauḍapāda's Kārikā, not the bare Upaniṣad and not the model's free
  association.

---

## 2. Functional requirements

### 2.1 Ask (home tab)
- **FR-1** A time-based greeting eyebrow ("Good morning/afternoon/evening", "Late
  night").
- **FR-2** A multi-line text input where the user types any everyday problem,
  with the prototype's placeholder example.
- **FR-3** A primary **"Look at it"** action. On submit: trim the input; if empty,
  no-op. Otherwise move to the Thinking state and start the answer request.
- **FR-4** Suggestion chips ("A relationship", "Stress", "Anger", "A decision",
  "Loneliness") that pre-fill the input with a first-person sentence (exact
  sentences per the prototype logic).
- **FR-5** A "four states" primer: four rows, each an accent dot + Sanskrit name +
  plain-English gloss.
- **FR-6** Copy on the input card states the data stays on the device ("Saved only
  on this device").

### 2.2 Thinking (transient sub-state of Ask)
- **FR-7** While the request is in flight, show a centered breathing halo with `ॐ`
  and a status line that cycles every ~1.6s through: "sitting with it…",
  "looking at the surface…", "listening underneath…", "finding the verse…".

### 2.3 Answer (sub-state of Ask)
- **FR-8** Render the four states **in fixed order** — jāgrat, svapna, suṣupti,
  turīya — each as an accordion card. The model fills only a short heading and a
  body per state. **First card open by default.**
- **FR-9** Each card shows its hard-coded Kārikā verse (Devanagari + English
  translation + reference). Verses are **never model-generated** (DEC-7).
- **FR-10** Show one **practice** line ("One thing to try") in a dark card.
- **FR-11** Echo the user's question above the cards; provide a back link that
  returns to a cleared Ask screen.
- **FR-12** On producing an answer, **auto-save** it as a Journal entry (newest
  first) and persist to device storage.
- **FR-13 (fallback)** If the request fails or returns invalid data, show a
  built-in generic four-level answer so the user never hits a dead end. The entry
  is still saved.

### 2.4 Journal (second tab)
- **FR-14** Read-only history of every past interaction, newest first. No
  re-asking from here.
- **FR-15** Title + count line ("N entries · yours only", or "Your history").
- **FR-16** Empty state: faded `ॐ` + the prototype's empty-state copy.
- **FR-17** Each entry is an accordion: timestamp eyebrow + question in the header;
  expanded body shows the four saved levels (dot, eyebrow, heading, body,
  reference) and the practice card.
- **FR-18** Timestamp formatting: "Today · h:mm", "Yesterday · h:mm", "N days ago"
  (<7), else "Mon D".

### 2.5 Navigation
- **FR-19** Two bottom tabs: Ask and Journal. Switching to Ask lands on the last
  Answer if one exists this session, else the empty Ask form.
- **FR-20** Accordions open/close independently (both on Answer and Journal).

### 2.6 The answer request (core behavior)
- **FR-21** One request to our backend per "Look at it". The backend, not the
  client, calls the LLM (DEC-4).
- **FR-22** The backend grounds the answer in retrieved Kārikā passages (DEC-7)
  and returns schema-validated JSON: `{ levels: [{heading, body} × 4], practice }`.
- **FR-23** Per-level constraints: body 45–70 words, concrete to *this* person's
  situation; heading ≤ 8 words; practice = one small concrete thing to do today,
  ≤ 25 words.
- **FR-24** The client merges the returned text with the hard-coded state→verse
  map to build the displayed cards and the saved entry.

---

## 3. Non-functional requirements

- **NFR-1 (privacy)** No user questions/answers/journal stored server-side; no
  accounts; no device identifiers collected (DEC-1, DEC-5). Problem text used for
  a single LLM request and not persisted.
- **NFR-2 (no training)** Use an LLM provider/tier that does not train on inputs;
  state it in-app (DEC-6).
- **NFR-3 (secrets)** The LLM API key never ships in the client; it lives only on
  the backend.
- **NFR-4 (resilience)** Any backend/LLM failure degrades to the fallback answer
  (FR-13); the app is fully usable offline for reading past Journal entries.
- **NFR-5 (accessibility)** Respect reduce-motion for all animations; hit targets
  ≥ 44px; readable contrast per the design tokens.
- **NFR-6 (fidelity)** Recreate the design tokens (fonts, colors, spacing, radii,
  type scale) faithfully from the handoff.
- **NFR-7 (extensibility)** The backend API is versioned (`/v1/...`) and
  structured so v2 features can be added without breaking v1 clients.
- **NFR-8 (performance)** Thinking state appears immediately on submit; the app
  opens straight to a calm screen with no blocking network call on launch.
- **NFR-9 (cost control)** `max_tokens` bounded (~1400 for the answer call); model
  ID configurable to trade cost/quality (DEC-6).

---

## 4. Content & scholarship (non-code, but required for v1)

- **CR-1** Proofread all Devanagari lines and Kārikā references against a Sanskrit
  source before launch (handoff flags them as unverified).
- **CR-2** Source and license a trustworthy translation/commentary text for the
  RAG corpus; confirm copyright (DEC-7). Record the source in the repo.
- **CR-3** Keep the four hard-coded verses and the state→verse map in one
  reviewed place (see architecture).

---

## 5. Non-goals (explicitly out of v1)

Śaṅkara's commentary · a browsable verse library · daily practice / reminders ·
streaks · community / sharing · user accounts · cross-device sync or cloud backup
· editing or deleting individual journal entries from the UI · push
notifications · analytics dashboards.

These are noted so v1 stays small; several are natural v2 candidates.

---

## 6. Acceptance criteria (v1 "done")

1. On a fresh install, the app opens to a calm Ask screen with no network call.
2. Submitting a typed problem shows the Thinking state, then a four-state Answer
   with correct verses and a practice line, within a reasonable wait.
3. Pulling the device offline (or forcing a backend error) still yields a complete
   fallback answer, saved to the Journal.
4. The answer is saved and appears, newest first, in the Journal across app
   restarts.
5. No user problem/answer text is present anywhere server-side (verified by
   inspection of logs and DB).
6. Reduce-motion disables the halo/entry animations; all tap targets are ≥ 44px.
7. The four states always render in the fixed order with the correct hard-coded
   verses, even when model text is missing.
