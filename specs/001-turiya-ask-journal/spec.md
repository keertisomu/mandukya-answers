# Feature Specification: Turiya — Ask & Journal (v1)

**Feature Branch**: `001-turiya-ask-journal`

**Created**: 2026-10-17

**Status**: Draft

**Input**: User description: "based on the design handoff README and Turiya.dc.html prototype, write the specifications for this product idea"

## Overview

Turiya is a calm mobile app where a person types any everyday problem — a relationship strain,
stress, anger, a decision they're stuck on — and receives a reflection that looks at it through
the four states of consciousness of the Māṇḍūkya Upanishad (waking, dream, deep sleep, and the
witness), each paired with a fixed verse from Gauḍapāda's Māṇḍūkya Kārikā. Every reflection is
automatically saved to a private, read-only Journal the person can return to. The experience goal
is that the app feels quieting the moment it opens, and that it never advises, judges, or sounds
negative — it helps the person step back into a witnessing stance rather than resolve the problem
for them.

This specification covers the complete v1 scope: **Ask** and **Journal**. Everything else
(Śaṅkara's commentary, a browsable verse library, daily practice, streaks, community, accounts)
is explicitly out of scope for v1.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Ask about a problem and receive a four-state reflection (Priority: P1)

A person opens the app, types what's weighing on them (or taps a suggestion to pre-fill a
starting sentence), and submits it. After a brief, calming wait, they receive a reflection
organized into the four fixed states — each with a short plain-language heading, a few sentences
written specifically about their situation, and the hard-coded Kārikā verse for that state — plus
a single small "one thing to try" today. The reflection is warm, never prescriptive, and never
negative.

**Why this priority**: This is the core value of the product and the reason a person opens it.
Without it there is no product. It is independently shippable as a complete, demonstrable
experience on its own.

**Independent Test**: Launch the app, type a problem, submit, and confirm a four-state reflection
plus a practice line is returned and displayed in the fixed order with the correct verse per
state — and that it reads as warm and non-advisory.

**Acceptance Scenarios**:

1. **Given** an empty Ask screen, **When** the person types a problem and submits, **Then** a
   calming in-progress state is shown while the reflection is prepared, followed by a reflection
   with exactly four states in the fixed order (Waking, The story, Underneath, The witness).
2. **Given** a returned reflection, **When** the person views it, **Then** each state shows an
   eyebrow label (Sanskrit · English), a short heading, a body written about their specific
   situation, and the correct fixed verse (Devanagari, English translation, reference) for that
   state, with the first state expanded by default.
3. **Given** a returned reflection, **When** the person reads it, **Then** a single "one thing to
   try" practice line is shown, phrased as a small reflective action, not a directive to resolve
   the problem.
4. **Given** the Ask screen, **When** the person taps a suggestion chip (e.g., "A relationship"),
   **Then** the input is pre-filled with a first-person starting sentence they can edit before
   submitting.
5. **Given** an empty or whitespace-only input, **When** the person submits, **Then** nothing
   happens (no reflection is generated).
6. **Given** a returned reflection, **When** the person chooses to ask again, **Then** they return
   to a cleared Ask screen ready for a new problem.

---

### User Story 2 - Return to a private journal of past reflections (Priority: P1)

Every reflection the person receives is automatically saved. The person can open a Journal tab and
scroll their history, newest first, expanding any past entry to re-read the full four-state
reflection and practice for that question. The Journal is read-only — a place to return to, not to
re-ask from — and the data stays private to the person's device.

**Why this priority**: Auto-saved history is the second half of the product promise ("kept here
for you to return to") and is what turns a one-off answer into an ongoing contemplative practice.
It is independently testable and valuable even with only one saved entry.

**Independent Test**: Generate at least one reflection, open the Journal tab, and confirm the
entry appears with its timestamp and question, expands to show the full saved four-state
reflection and practice, and persists after the app is closed and reopened.

**Acceptance Scenarios**:

1. **Given** at least one past reflection, **When** the person opens the Journal, **Then** entries
   are listed newest-first, each showing a relative timestamp and the original question.
2. **Given** a Journal with no entries, **When** the person opens it, **Then** a gentle empty
   state invites them to ask something, with no error or blank screen.
3. **Given** a Journal entry, **When** the person expands it, **Then** the four saved states
   (label, heading, body, verse reference) and the practice line are shown exactly as they were
   when first generated.
4. **Given** a saved journal, **When** the person closes and reopens the app, **Then** their past
   entries are still present.
5. **Given** the Journal, **When** the person views it, **Then** there is no way to re-ask or edit
   from within it (it is read-only).

---

### User Story 3 - Trust that the experience is safe, private, and never harmful (Priority: P1)

Because people bring their rawest problems here, the app must be dependable in tone and safety at
every moment. If a person expresses crisis or self-harm, the app steps out of the reflective flow
and surfaces real support resources instead. No reflection ever gives a verdict, tells the person
what to do, endorses anything that could hurt them or others, or sounds negative or discouraging.
If a reflection cannot be prepared (e.g., no connection), the person still receives a steadying
built-in reflection rather than an error dead-end. The person's problems and history never leave
their device in v1.

**Why this priority**: These are constitutional guarantees (reflective-not-prescriptive,
do-no-harm/non-negativity, safety-as-a-hard-gate, privacy-by-default). They are not optional
polish; a single harmful or negative line, or a leak of private data, breaks the product's core
promise. They apply to every reflection produced by User Story 1 and every entry stored by User
Story 2.

**Independent Test**: Submit inputs that signal crisis and confirm the app shows support
resources instead of a reflection; submit inputs inviting a verdict or harmful action and confirm
the response stays reflective, warm, and non-harmful; disable connectivity and confirm a steadying
built-in reflection is still shown; confirm no reflection or journal content is transmitted off
the device.

**Acceptance Scenarios**:

1. **Given** input that signals crisis or self-harm, **When** the person submits, **Then** the app
   skips the reflective flow and presents real crisis-support resources.
2. **Given** any reflection shown to the person, **When** it is read, **Then** it contains no
   verdict, diagnosis, or prescriptive advice, and nothing that endorses or implies harm to the
   person or others.
3. **Given** any surface in the app (reflection, practice, fallback, empty and error states, static
   copy), **When** the person reads it, **Then** the tone is warm and steadying and never negative,
   harsh, shaming, or discouraging.
4. **Given** a reflection cannot be generated (failure or no connectivity), **When** the person
   submits, **Then** a built-in steadying four-state reflection and practice are shown instead of
   an error.
5. **Given** the person uses the app, **When** they ask and journal, **Then** their problem text
   and journal entries remain on their device and are not sent to any server except, transiently,
   the problem text required to prepare a single reflection.

---

### Edge Cases

- **Empty / whitespace-only submission**: no reflection is generated; the person stays on the Ask
  screen.
- **Very long problem text**: the input accepts a reasonably long passage and the reflection is
  still produced; overly long input is handled gracefully without truncation errors visible to the
  person.
- **Malformed or incomplete reflection content**: if the prepared reflection is missing states or
  is otherwise invalid, the built-in steadying reflection is shown instead.
- **No connectivity / service unavailable**: the built-in steadying reflection is shown; the
  person is never blocked.
- **Returning to the Ask tab with a prior reflection present**: the person lands back on their most
  recent reflection rather than a blank screen; switching away and back does not lose it.
- **Reduced-motion preference enabled**: all ambient animations are minimized or disabled.
- **Local storage unavailable or cleared**: the app still functions for asking; the Journal simply
  shows the empty state, with no crash.
- **Rapid repeated submissions**: only one reflection is prepared at a time; the in-progress state
  prevents duplicate concurrent requests.

## Requirements *(mandatory)*

### Functional Requirements

**Asking & reflection**

- **FR-001**: The app MUST provide an Ask screen where a person can type a free-text everyday
  problem and submit it.
- **FR-002**: The app MUST offer a small set of suggestion chips that, when tapped, pre-fill the
  input with an editable first-person starting sentence.
- **FR-003**: The app MUST ignore empty or whitespace-only submissions (no reflection generated).
- **FR-004**: On submission, the app MUST show a calming in-progress state while the reflection is
  prepared.
- **FR-005**: The app MUST return a reflection organized into exactly four states, always presented
  in this fixed order: Waking (Jāgrat), The story (Svapna), Underneath (Suṣupti), The witness
  (Turīya).
- **FR-006**: Each state MUST display a short plain-language heading and a body written about the
  person's specific situation (not generic), together with that state's fixed, pre-defined Kārikā
  verse (Devanagari, English translation, and reference).
- **FR-007**: The verse text and citation for each state MUST be fixed/pre-defined content and MUST
  NOT be generated at request time, so citations cannot be fabricated.
- **FR-008**: Each reflection MUST include exactly one "one thing to try" practice line, phrased as
  a small reflective action for today, not a directive to resolve the person's problem.
- **FR-009**: The reflection MUST present the first state expanded by default and allow each state
  to be independently expanded and collapsed.
- **FR-010**: After a reflection is shown, the person MUST be able to return to a cleared Ask screen
  to ask again.

**Journal**

- **FR-011**: The app MUST automatically save every reflection (the question, the four states as
  shown, and the practice line, with a timestamp) to the Journal without any explicit save action.
- **FR-012**: The app MUST provide a Journal view listing saved entries newest-first, each showing
  a human-friendly relative timestamp and the original question.
- **FR-013**: The Journal MUST allow each entry to be expanded to re-read the full saved four-state
  reflection and practice exactly as originally generated.
- **FR-014**: The Journal MUST be read-only — it MUST NOT allow re-asking from, or editing of,
  saved entries.
- **FR-015**: The app MUST show a gentle empty state in the Journal when there are no entries.
- **FR-016**: Saved Journal entries MUST persist across app restarts.

**Navigation & experience**

- **FR-017**: The app MUST provide two primary destinations — Ask and Journal — reachable from a
  persistent navigation bar.
- **FR-018**: Returning to Ask MUST land on the most recent reflection if one exists in the current
  session, otherwise on the empty Ask form.
- **FR-019**: The app MUST faithfully honor the approved visual design (typography, color, spacing,
  copy, and interaction flow) and MUST meet a minimum interactive target size of at least 44px.
- **FR-020**: Ambient motion MUST be subtle and MUST be minimized or disabled when the person's
  device indicates a reduced-motion preference.
- **FR-021**: Relative timestamps MUST read as "Today · h:mm", "Yesterday · h:mm", "N days ago"
  for the last week, and an abbreviated month-and-day date beyond that.

**Safety, tone & privacy (constitutional)**

- **FR-022**: The app MUST detect inputs signalling crisis or self-harm before preparing a
  reflection and, in that case, MUST skip the reflective flow and present real crisis-support
  resources instead.
- **FR-023**: No reflection or any other surface MUST ever present a verdict, diagnosis, or
  prescriptive advice, or endorse or imply any action that could harm the person or others.
- **FR-024**: Every surface (reflection, practice, fallback, empty and error states, and all static
  copy) MUST maintain a warm, steadying tone and MUST NOT sound negative, harsh, shaming, or
  discouraging.
- **FR-025**: If a reflection cannot be prepared or is invalid, the app MUST show a built-in
  steadying four-state reflection and practice so the person never reaches a dead-end or error.
- **FR-026**: In v1, the person's problem text and Journal entries MUST remain private to their
  device; the only transmission permitted is the problem text required to prepare a single
  reflection, and no credentials or secrets for the reflection service MUST be stored in the app.
- **FR-027**: The app MUST clearly convey that content is private to the device (e.g., a
  "Saved only on this device" assurance on the Ask input) and MUST NOT present itself as therapy,
  counseling, or treatment.

### Key Entities *(include if feature involves data)*

- **Problem (Question)**: the free-text everyday problem a person submits. Transient input;
  echoed back on the reflection and stored as part of a Journal Entry.
- **State**: one of the four fixed states of consciousness (Waking, The story, Underneath, The
  witness), each with a fixed Sanskrit name, English label, accent identity, and a fixed Kārikā
  verse (Devanagari, English translation, reference). The set and order are constant and defined
  by the app, not generated.
- **Reflection**: the generated content for a single question — a heading and situation-specific
  body for each of the four states, plus one practice line. Produced per submission.
- **Journal Entry**: a persisted record combining the question, the timestamp, the four states as
  shown (label, heading, body, verse reference), and the practice line. Stored privately on the
  device, listed newest-first, read-only.
- **Crisis-support resources**: the fixed set of real support resources shown in place of a
  reflection when crisis/self-harm is detected.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A person can go from opening the app to reading a complete four-state reflection in
  under 30 seconds under normal conditions.
- **SC-002**: 100% of reflections display exactly four states in the correct fixed order, each with
  its correct, unaltered verse and reference.
- **SC-003**: In a review of reflections across the supported problem types, at least 95% are rated
  as warm and reflective, and 0% contain a verdict, prescriptive advice, or harmful/negative
  content (zero tolerance on harm and negativity).
- **SC-004**: 100% of inputs that signal crisis or self-harm result in support resources being
  shown instead of a reflection.
- **SC-005**: 100% of reflection failures (service error, invalid content, or no connectivity)
  result in a steadying built-in reflection rather than a visible error or empty state.
- **SC-006**: 100% of generated reflections are saved to the Journal and remain available after the
  app is closed and reopened.
- **SC-007**: In v1, no reflection or Journal content is transmitted off the device except the
  problem text sent to prepare a single reflection (verifiable by inspecting outbound data).
- **SC-008**: In first-use testing, at least 90% of people successfully ask a problem and read a
  reflection on their first attempt without guidance.
- **SC-009**: In first-use testing, a majority of people describe the app as calming/quieting when
  asked for a one-word impression.

## Assumptions

- **Scope is v1 Ask + Journal only.** Śaṅkara's commentary, a browsable verse library, daily
  practice, streaks, community, and accounts are deferred and will not be built here.
- **Platform.** v1 is a single cross-platform mobile app; a person uses it on one device, and the
  Journal is that device's local history (no cross-device sync in v1).
- **Reflection generation uses an external language model reached only through the project's own
  backend**, per the constitution — so no model credentials live in the app. This is reconciled
  with device-only privacy: the problem text is sent transiently to prepare one reflection, while
  the Journal itself is never uploaded or stored server-side in v1.
- **The four states, their order, their glosses, and their verses are fixed app content**, defined
  once and proof-read against a Sanskrit source before public release; they are never generated.
- **Suggestion chips** cover the common everyday problem types (e.g., a relationship, stress,
  anger, a decision, loneliness) and pre-fill editable starting sentences.
- **Crisis detection in v1** is a dependable, rule-based gate sufficient for an early release,
  enforced before any reflection is prepared; it favors showing support resources when uncertain.
- **Reasonable-length input** is expected (a sentence to a short paragraph); extremely long input is
  handled gracefully.
- **Standard mobile performance and error-handling expectations** apply; the in-progress state and
  built-in fallback keep the person from ever hitting a dead-end.
