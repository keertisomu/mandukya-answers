# Feature Specification: Turiya — Learn, Ask & Journal (v1)

**Feature Branch**: `001-turiya-ask-journal`

**Created**: 2026-10-17

**Status**: Draft

**Updated**: 2026-10-24 — v1 launch region restricted to Australia

**Updated**: 2026-11-06 — v1 refocused on learning the Upaniṣad (a daily lesson path), with Ask
kept for applying it to everyday problems

**Input**: User description: "based on the design handoff README and Turiya.dc.html prototype, write the specifications for this product idea"

**Update input**: "can we restrict the spec to just australia for v1 and then for v2 we can think about India followed by other countries"

**Update input**: "rather than using this as a platform to answer questions about anything using the mandukya upanishad, what if we re-purpose this platform allowing them to learn the upanishad?" — chosen direction: learning at the centre, Ask kept for applying it.

## Overview

Turiya is a calm mobile app for **learning the Māṇḍūkya Upaniṣad through Gauḍapāda's Māṇḍūkya
Kārikā**, and for living what it teaches. Its centre is **the Path**: a short lesson each day, one
verse at a time, explained in plain words and ending with a small reflection that applies the verse
to the person's own life. Alongside the Path, **Ask** lets a person bring an everyday problem — a
relationship strain, stress, anger, a decision — and receive a reflection through the four states of
consciousness (waking, dream, deep sleep, and the witness), each paired with a fixed Kārikā verse.
Lesson notes and Ask reflections are kept together in a private, device-only **Journal**.

The experience goal is unchanged: the app feels quieting the moment it opens, and it never advises,
judges, or sounds negative. Learning gives the person a reason to return each day; Ask is there for
the days something is weighing on them.

This specification covers the complete v1 scope: **Learn (the Path), Ask, and Journal**. Śaṅkara's
commentary, chapters 2–4 of the Kārikā as lessons, a browsable library of verses not yet taught,
streaks, community, and accounts are explicitly out of scope for v1.

**Launch region**: v1 is offered **only in Australia**. Its crisis-support resources, privacy
notices, language conventions and store availability are all Australian. India is planned as the
next region (v2), followed by other countries; each new region will be specified separately,
because it brings its own crisis services, privacy law and language needs.

**Requirement IDs**: IDs from earlier versions of this spec are kept stable so other documents can
keep referring to them. Requirements added for learning are numbered from FR-033 and SC-013.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Learn the Upaniṣad one lesson a day (Priority: P1)

A person opens the app and lands on today's lesson in the Path. The lesson shows one verse — the
Sanskrit in Devanagari, a transliteration, an English translation and its reference — followed by a
short explanation in plain words of what the verse says as Gauḍapāda reads it, and which of the four
states it concerns. It ends with a "live it today" prompt: a small reflection that applies the verse
to the person's own day. The person can write a few lines in response, which are kept in their
Journal. When they finish, the next lesson becomes available the following day. They can see how far
along the Path they are and re-read any lesson they have completed.

**Why this priority**: Learning is now the centre of the product and the main reason to return each
day. It is also what sets Turiya apart from a general chatbot: an authentic, structured way into a
specific text. On its own — with no Ask at all — it is a complete, demonstrable product.

**Independent Test**: Install the app, complete the first lesson (read it, write a response), confirm
the response appears in the Journal, confirm the second lesson is not yet available until the next
day, then (after the day changes) open the second lesson and confirm progress shows 2 of the total.

**Acceptance Scenarios**:

1. **Given** a first launch, **When** the app opens, **Then** the person sees the first lesson of the
   Path (after any one-time notices required by FR-031 and FR-041), not an empty screen.
2. **Given** an open lesson, **When** the person reads it, **Then** it shows the verse (Devanagari,
   transliteration, English translation, reference and translation source), a plain-language
   explanation, the related state where there is one, and a "live it today" prompt.
3. **Given** an open lesson, **When** the person writes a response to the prompt and saves it,
   **Then** the response is kept in the Journal with the lesson's title and verse reference.
4. **Given** a completed lesson, **When** the person returns on the same day, **Then** the next lesson
   is shown as available tomorrow, and the completed lesson can still be re-read.
5. **Given** the next calendar day, **When** the person opens the app, **Then** the next lesson is
   available.
6. **Given** a person who has not opened the app for several days, **When** they return, **Then** they
   continue from the next lesson in order, with no message about missed days.
7. **Given** a person anywhere in the Path, **When** they look at the Learn screen, **Then** they see
   their progress (e.g., "Lesson 7 of 30") and can open any lesson already completed.
8. **Given** a person completes the final lesson, **When** they finish it, **Then** they see a gentle
   completion moment, can re-read any lesson or begin the Path again, and are told that further
   chapters will come later.
9. **Given** no network connection, **When** the person opens and completes a lesson, **Then** it works
   fully.

---

### User Story 2 - Ask about a problem and receive a four-state reflection (Priority: P2)

A person brings something weighing on them — typed directly, or by tapping a suggestion to pre-fill a
starting sentence — and submits it. After a brief, calming wait, they receive a reflection organized
into the four fixed states — each with a short plain-language heading, a few sentences written about
their situation, and the hard-coded Kārikā verse for that state — plus a single small "one thing to
try" today. The reflection is warm, never prescriptive, and never negative. Where one of its verses is
also a lesson the person has reached in the Path, they can open that lesson from the reflection.

**Why this priority**: Ask is how the teaching meets a person's real day, and it remains a strong
reason to open the app when something is wrong. It is now secondary to the Path but independently
valuable and testable.

**Independent Test**: Open Ask, type a problem, submit, and confirm a four-state reflection plus a
practice line is shown in the fixed order with the correct verse per state, reads as warm and
non-advisory, and is saved to the Journal.

**Acceptance Scenarios**:

1. **Given** an empty Ask screen, **When** the person types a problem and submits, **Then** a calming
   in-progress state is shown while the reflection is prepared, followed by a reflection with exactly
   four states in the fixed order (Waking, The story, Underneath, The witness).
2. **Given** a returned reflection, **When** the person views it, **Then** each state shows an eyebrow
   label (Sanskrit · English), a short heading, a body written about their specific situation, and the
   correct fixed verse (Devanagari, English translation, reference) for that state, with the first
   state expanded by default.
3. **Given** a returned reflection, **When** the person reads it, **Then** a single "one thing to try"
   practice line is shown, phrased as a small reflective action, not a directive to resolve the
   problem.
4. **Given** the Ask screen, **When** the person taps a suggestion chip (e.g., "A relationship"),
   **Then** the input is pre-filled with a first-person starting sentence they can edit before
   submitting.
5. **Given** an empty or whitespace-only input, **When** the person submits, **Then** nothing happens
   (no reflection is generated).
6. **Given** a returned reflection, **When** the person chooses to ask again, **Then** they return to a
   cleared Ask screen ready for a new problem.
7. **Given** a reflection whose verse belongs to a lesson the person has already reached, **When** they
   choose to learn more about that verse, **Then** the lesson opens.
8. **Given** an open lesson, **When** the person chooses to bring something they are facing to it,
   **Then** Ask opens, ready for them to type.

---

### User Story 3 - Return to a private journal of lessons and reflections (Priority: P1)

Everything the person writes or receives is kept in one Journal: their responses to lesson prompts
and their Ask reflections, newest first. They can expand any entry to re-read it. The Journal is a
place to return to, not to edit or re-ask from, and it stays on the person's device.

**Why this priority**: The Journal is what turns daily lessons and one-off questions into a practice
the person can look back on ("kept here for you to return to"). It is independently testable with a
single entry of either kind.

**Independent Test**: Write a response to a lesson prompt and complete one Ask, open the Journal, and
confirm both entries appear newest-first with their timestamps, expand to their full content, and
persist after the app is closed and reopened.

**Acceptance Scenarios**:

1. **Given** past lesson responses and Ask reflections, **When** the person opens the Journal,
   **Then** entries of both kinds are listed newest-first, each showing a relative timestamp and
   either the lesson title or the original question, with the kind of entry clearly distinguishable.
2. **Given** a Journal with no entries, **When** the person opens it, **Then** a gentle empty state
   invites them to begin the Path or ask something, with no error or blank screen.
3. **Given** an Ask entry, **When** the person expands it, **Then** the four saved states (label,
   heading, body, verse reference) and the practice line are shown exactly as first generated.
4. **Given** a lesson entry, **When** the person expands it, **Then** their response, the lesson's
   prompt and the verse reference are shown, with a way to open the lesson.
5. **Given** a saved journal, **When** the person closes and reopens the app, **Then** all entries are
   still present.
6. **Given** the Journal, **When** the person views it, **Then** there is no way to re-ask or edit from
   within it.

---

### User Story 4 - Trust that the experience is safe, private, and never harmful (Priority: P1)

Because people bring their rawest problems here, the app must be dependable in tone and safety at
every moment. If a person expresses crisis or self-harm — in Ask or in a lesson response — the app
surfaces real support resources. No reflection ever gives a verdict, tells the person what to do,
endorses anything that could hurt them or others, or sounds negative or discouraging. If a
reflection cannot be prepared (e.g., no connection), the person still receives a steadying built-in
reflection rather than an error dead-end. Lessons and lesson responses never leave the device; the
only thing ever sent is an Ask question, and only after the person has agreed to it.

**Why this priority**: These are constitutional guarantees (reflective-not-prescriptive,
do-no-harm/non-negativity, textual integrity, safety-as-a-hard-gate, privacy-by-default). A single
harmful or negative line, a misquoted verse, or a leak of private data breaks the product's core
promise.

**Independent Test**: Submit Ask inputs that signal crisis and confirm support resources are shown
instead of a reflection; write a crisis-signalling lesson response and confirm support resources are
shown; submit inputs inviting a verdict or harmful action and confirm the response stays reflective,
warm and non-harmful; disable connectivity and confirm a steadying built-in reflection is still shown;
confirm no lesson response is transmitted off the device.

**Acceptance Scenarios**:

1. **Given** Ask input that signals crisis or self-harm, **When** the person submits, **Then** the app
   skips the reflective flow and presents real Australian crisis-support resources, including
   emergency services (000) and Lifeline.
2. **Given** a lesson response that signals crisis or self-harm, **When** the person saves it, **Then**
   the response is kept and the same support resources are shown, without anything being sent off
   the device.
3. **Given** any screen, **When** the person wants help, **Then** a way to reach the support resources
   is available within one tap from Learn, Ask and Journal.
4. **Given** any reflection shown to the person, **When** it is read, **Then** it contains no verdict,
   diagnosis, or prescriptive advice, and nothing that endorses or implies harm to the person or
   others.
5. **Given** any surface in the app (lessons, reflections, practice, fallback, empty and error states,
   static copy), **When** the person reads it, **Then** the tone is warm and steadying and never
   negative, harsh, shaming, or discouraging.
6. **Given** a reflection cannot be generated (failure or no connectivity), **When** the person
   submits, **Then** a built-in steadying four-state reflection and practice are shown instead of an
   error.
7. **Given** the person uses the app, **When** they learn, ask and journal, **Then** lesson responses
   and journal entries remain on their device, and only Ask question text is sent — transiently, to
   prepare a single reflection, and only after they have agreed to it.

---

### User Story 5 - A gentle daily reminder (Priority: P3)

A person who wants help keeping the habit can choose a time for a quiet daily reminder that their next
lesson is ready. It is off unless they turn it on, never mentions anything they have written, and can
be changed or turned off at any time.

**Why this priority**: Gentle reminders were the strongest retention signal in the market research,
but the Path works without them. It is a small, independent addition.

**Independent Test**: Turn on the reminder for a chosen time, confirm a reminder arrives at that time
on a day with a lesson available, confirm it contains no journal content, then turn it off and
confirm no further reminders arrive.

**Acceptance Scenarios**:

1. **Given** a person who has not turned on reminders, **When** they use the app, **Then** no reminder
   is ever sent and they are not asked for notification permission at first launch.
2. **Given** a person turning on reminders, **When** they choose a time, **Then** the app asks for the
   device's notification permission at that moment, with a one-line explanation.
3. **Given** reminders are on and a new lesson is available, **When** the chosen time arrives, **Then**
   a reminder appears whose text contains no question, response or other journal content.
4. **Given** reminders are on, **When** the person has already opened that day's lesson, **Then** no
   reminder is sent that day.
5. **Given** the person declines notification permission, **When** they continue, **Then** everything
   else works normally.

---

### Edge Cases

- **Empty / whitespace-only Ask submission**: no reflection is generated; the person stays on the Ask
  screen.
- **Empty lesson response**: the lesson can still be completed without writing anything; nothing is
  added to the Journal.
- **Very long problem text or lesson response**: accepted up to a reasonable length and handled
  gracefully without visible truncation errors.
- **Malformed or incomplete reflection content**: if the prepared reflection is missing states or is
  otherwise invalid, the built-in steadying reflection is shown instead.
- **No connectivity / service unavailable**: Learn and Journal work fully; Ask shows the built-in
  steadying reflection; the person is never blocked.
- **Missed days**: the Path simply waits; the person continues from the next lesson, with no streak
  lost, no catch-up, and no guilt-inducing copy.
- **Device date or time changed**: lessons unlock by the device's calendar day; moving the clock
  forward unlocks lessons early, which is acceptable and not prevented.
- **Finishing the Path**: a completion moment, then free re-reading or starting again; no dead end.
- **App reinstalled or device changed**: Path progress and the Journal are device-only in v1 and are
  lost; the app says so plainly in its privacy information.
- **Returning to the Ask tab with a prior reflection present**: the person lands back on their most
  recent reflection rather than a blank screen; switching away and back does not lose it.
- **Reduced-motion preference enabled**: all ambient animations are minimized or disabled.
- **Local storage unavailable or cleared**: the app still works for learning and asking; progress
  restarts at the first lesson and the Journal shows the empty state, with no crash.
- **Rapid repeated Ask submissions**: only one reflection is prepared at a time; the in-progress state
  prevents duplicate concurrent requests.
- **Person declines the AI data-sharing consent (FR-041)**: Learn and Journal work fully; Ask explains
  that it needs this consent and offers to ask again; support resources stay available.
- **Person outside Australia**: the app is not offered in other countries' app stores. A person who
  installed it in Australia and then travels keeps full use of it; the crisis-support screen still
  lists the Australian resources, notes that 000 and the Australian phone lines work only within
  Australia, and offers an international helpline directory.
- **Device set to another language or region**: the app still uses Australian English and Australian
  resources in v1; it does not switch regions based on device settings.

## Requirements *(mandatory)*

### Functional Requirements

**Learning — the Path (new in this update)**

- **FR-033**: The app MUST provide a Path of lessons covering the whole of the first chapter of
  Gauḍapāda's Māṇḍūkya Kārikā (Āgama Prakaraṇa) — the twelve mantras of the Māṇḍūkya Upaniṣad
  together with Gauḍapāda's twenty-nine kārikās on them — in their textual order.
- **FR-034**: Each lesson MUST present one verse, or a mantra together with the kārikās that comment
  on it, showing: Devanagari, transliteration, English translation, the verse reference, and the
  source of the translation.
- **FR-035**: Each lesson MUST include a plain-language explanation of what the verse says as
  Gauḍapāda reads it (readable in about 3–5 minutes together with the verse), the state of
  consciousness it concerns where there is one, and a "live it today" prompt that invites a small
  reflection on the person's own day.
- **FR-036**: All lesson content — verse text, translation, explanation and prompt — MUST be fixed,
  pre-written content that has been proof-read and reviewed before release. It MUST NOT be generated
  at the time the person reads it.
- **FR-037**: The person MUST be able to write a response to a lesson's prompt. A saved response MUST
  be kept in the Journal with the lesson's title and verse reference, and MUST stay on the device.
  Writing a response MUST be optional for completing a lesson.
- **FR-038**: Lessons MUST become available in order, one new lesson per calendar day: completing a
  lesson makes the next one available from the following day. Missed days MUST NOT reset progress,
  break a streak, or produce any message about missing days.
- **FR-039**: The Learn screen MUST show the person's progress through the Path (e.g., "Lesson 7 of
  30") and let them re-read any lesson they have completed. Completing the final lesson MUST lead to
  a completion moment with the option to re-read lessons or begin the Path again.
- **FR-040**: The Path MUST work fully without a network connection.

**Asking & reflection**

- **FR-001**: The app MUST provide an Ask screen where a person can type a free-text everyday problem
  and submit it.
- **FR-002**: The app MUST offer a small set of suggestion chips that, when tapped, pre-fill the input
  with an editable first-person starting sentence.
- **FR-003**: The app MUST ignore empty or whitespace-only submissions (no reflection generated).
- **FR-004**: On submission, the app MUST show a calming in-progress state while the reflection is
  prepared.
- **FR-005**: The app MUST return a reflection organized into exactly four states, always presented in
  this fixed order: Waking (Jāgrat), The story (Svapna), Underneath (Suṣupti), The witness (Turīya).
- **FR-006**: Each state MUST display a short plain-language heading and a body written about the
  person's specific situation (not generic), together with that state's fixed, pre-defined Kārikā
  verse (Devanagari, English translation, and reference).
- **FR-007**: The verse text and citation for each state MUST be fixed/pre-defined content and MUST
  NOT be generated at request time, so citations cannot be fabricated.
- **FR-008**: Each reflection MUST include exactly one "one thing to try" practice line, phrased as a
  small reflective action for today, not a directive to resolve the person's problem.
- **FR-009**: The reflection MUST present the first state expanded by default and allow each state to
  be independently expanded and collapsed.
- **FR-010**: After a reflection is shown, the person MUST be able to return to a cleared Ask screen to
  ask again.
- **FR-042**: Where a reflection's verse is part of a lesson the person has already reached, the
  reflection MUST offer a way to open that lesson. Each lesson MUST offer a way to open Ask.

**Journal**

- **FR-011**: The app MUST automatically save every Ask reflection (the question, the four states as
  shown, and the practice line, with a timestamp) and every saved lesson response (with the lesson
  title, verse reference and a timestamp) to the Journal, without any separate save action for Ask.
- **FR-012**: The app MUST provide a Journal view listing all entries newest-first, each showing a
  human-friendly relative timestamp and the lesson title or original question, with lesson entries
  and Ask entries clearly distinguishable.
- **FR-013**: The Journal MUST allow each entry to be expanded to re-read it in full exactly as saved;
  a lesson entry MUST also offer a way to open its lesson.
- **FR-014**: The Journal MUST be read-only — it MUST NOT allow re-asking from, or editing of, saved
  entries. A person MAY change their response to a lesson by revisiting the lesson itself.
- **FR-015**: The app MUST show a gentle empty state in the Journal when there are no entries.
- **FR-016**: Journal entries and Path progress MUST persist across app restarts.

**Navigation & experience**

- **FR-017**: The app MUST provide three primary destinations — Learn, Ask and Journal — reachable
  from a persistent navigation bar. Learn MUST be the screen the app opens to.
- **FR-018**: Returning to Ask MUST land on the most recent reflection if one exists in the current
  session, otherwise on the empty Ask form.
- **FR-019**: The app MUST faithfully honor the approved visual design (typography, color, spacing,
  copy, and interaction flow) and MUST meet a minimum interactive target size of at least 44px.
- **FR-020**: Ambient motion MUST be subtle and MUST be minimized or disabled when the person's device
  indicates a reduced-motion preference.
- **FR-021**: Relative timestamps MUST read as "Today · h:mm", "Yesterday · h:mm", "N days ago" for
  the last week, and an abbreviated date in Australian day-month order beyond that (e.g., "24 Oct").
- **FR-043**: The person MUST be able to turn on an optional daily reminder at a time they choose.
  Reminders MUST be off by default; notification permission MUST be requested only when the person
  turns reminders on; a reminder MUST NOT be sent on a day the person has already opened that day's
  lesson, or when no new lesson is available; reminder text MUST NOT contain any journal content; and
  the person MUST be able to change the time or turn reminders off at any time.

**Safety, tone & privacy (constitutional)**

- **FR-022**: The app MUST detect Ask inputs signalling crisis or self-harm before preparing a
  reflection and, in that case, MUST skip the reflective flow and present real crisis-support
  resources instead.
- **FR-044**: The app MUST check saved lesson responses for signals of crisis or self-harm on the
  device, without sending them anywhere, and in that case MUST present the same crisis-support
  resources.
- **FR-045**: A way to reach the crisis-support resources MUST be available within one tap from
  Learn, Ask and Journal, and MUST work without a network connection.
- **FR-023**: No reflection, lesson or any other surface MUST ever present a verdict, diagnosis, or
  prescriptive advice, or endorse or imply any action that could harm the person or others.
- **FR-024**: Every surface (lessons, reflections, practice, fallback, empty and error states, and all
  static copy) MUST maintain a warm, steadying tone and MUST NOT sound negative, harsh, shaming, or
  discouraging.
- **FR-025**: If a reflection cannot be prepared or is invalid, the app MUST show a built-in steadying
  four-state reflection and practice so the person never reaches a dead-end or error.
- **FR-026**: In v1, lesson responses, Path progress and Journal entries MUST remain private to the
  person's device. The only transmission permitted is the Ask question text required to prepare a
  single reflection, and no credentials or secrets for the reflection service MUST be stored in the
  app.
- **FR-027**: The app MUST clearly convey that content is private to the device (e.g., a "Saved only
  on this device" assurance) and MUST NOT present itself as therapy, counseling, or treatment.
  Lessons and reflections MUST be presented as teaching and reflection drawing on Gauḍapāda's Kārikā,
  not as religious authority.

**Launch region (v1: Australia)**

- **FR-028**: v1 MUST be made available only through the Australian app storefronts. It MUST NOT be
  offered in any other country.
- **FR-029**: The crisis-support resources MUST be Australian and MUST include, at minimum, emergency
  services (000) and Lifeline, plus a resource suited to young people and one for ongoing anxiety and
  depression support. Each entry MUST show the service name, how to contact it (call and, where
  offered, text or chat), and when it is available.
- **FR-030**: Every crisis-support resource MUST be verified as current before release and
  re-verified at least every 90 days while v1 is live.
- **FR-031**: Before the person's first Ask, the app MUST tell them, in plain language, that their
  question is sent to a service that may be located outside Australia to prepare the reflection,
  that it is not stored there, and that their lessons, responses and Journal never leave the device.
  The same information MUST be available at any time from within the app.
- **FR-041**: Before the first Ask question is sent, the app MUST obtain the person's explicit
  agreement to sending it to the third-party AI service that prepares reflections, naming that
  service and what is sent. Until they agree, no question is sent; Learn and Journal remain fully
  usable. The person MUST be able to review this choice later.
- **FR-032**: All app copy MUST use Australian English spelling and conventions.

### Key Entities *(include if feature involves data)*

- **Path**: the ordered sequence of lessons covering the first chapter of the Kārikā. Fixed content,
  the same for everyone.
- **Lesson**: one step of the Path — a verse (or a mantra with its kārikās) with Devanagari,
  transliteration, translation, reference and translation source; a plain-language explanation; the
  related state, if any; and a "live it today" prompt. Fixed, reviewed content.
- **Path progress**: which lessons the person has completed and when the next one becomes available.
  Stored only on the device.
- **Lesson response**: what the person writes in reply to a lesson prompt, with the lesson it belongs
  to and a timestamp. Stored only on the device; never sent anywhere.
- **Problem (Question)**: the free-text everyday problem a person submits in Ask. Sent transiently to
  prepare one reflection; stored on the device as part of a Journal entry.
- **State**: one of the four fixed states of consciousness (Waking, The story, Underneath, The
  witness), each with a fixed Sanskrit name, English label, accent identity, and a fixed Kārikā verse
  (Devanagari, English translation, reference). The set and order are constant.
- **Reflection**: the generated content for a single Ask question — a heading and situation-specific
  body for each of the four states, plus one practice line.
- **Journal entry**: either a lesson response or an Ask reflection, with a timestamp. Stored privately
  on the device, listed newest-first, read-only in the Journal.
- **Reminder setting**: whether the daily reminder is on and at what time. Stored on the device.
- **Crisis-support resources**: the fixed set of real Australian support resources shown when crisis
  or self-harm is detected, or when the person asks for help. Each has a name, contact methods,
  availability hours, and the date it was last verified.

## Success Criteria *(mandatory)*

### Measurable Outcomes

**Learning**

- **SC-013**: In first-use testing, at least 90% of people complete the first lesson without guidance,
  and the median time to read a lesson and its prompt is between 3 and 5 minutes.
- **SC-014**: 100% of lesson verses, transliterations, translations and references have been
  proof-read against a Sanskrit source, and 100% of explanations reviewed by someone grounded in the
  tradition, before release; every lesson names its translation source.
- **SC-015**: 100% of lessons, Path progress and the Journal work with no network connection.
- **SC-016**: In the first 60 days after launch, day-30 retention reported by the app stores is at
  least 12% — above the roughly 6–8% typical of meditation apps.
- **SC-017**: In first-use testing, a majority of people say they would come back for tomorrow's
  lesson.

**Asking & reflection**

- **SC-001**: A person can go from opening Ask to reading a complete four-state reflection in under 30
  seconds under normal conditions.
- **SC-002**: 100% of reflections display exactly four states in the correct fixed order, each with
  its correct, unaltered verse and reference.
- **SC-003**: In a review of reflections across the supported problem types, at least 95% are rated as
  warm and reflective, and 0% contain a verdict, prescriptive advice, or harmful/negative content
  (zero tolerance on harm and negativity).

**Safety, privacy & reliability**

- **SC-004**: 100% of Ask inputs and lesson responses that signal crisis or self-harm result in
  support resources being shown.
- **SC-005**: 100% of reflection failures (service error, invalid content, or no connectivity) result
  in a steadying built-in reflection rather than a visible error or empty state.
- **SC-006**: 100% of reflections and saved lesson responses appear in the Journal and remain
  available after the app is closed and reopened.
- **SC-007**: In v1, nothing is transmitted off the device except Ask question text sent to prepare a
  single reflection after the person has agreed (verifiable by inspecting outbound data).

**Experience**

- **SC-008**: In first-use testing, at least 90% of people successfully ask a problem and read a
  reflection on their first attempt without guidance.
- **SC-009**: In first-use testing, a majority of people describe the app as calming/quieting when
  asked for a one-word impression.

**Launch region**

- **SC-010**: 100% of crisis-support screens show Australian resources, including 000 and Lifeline,
  and every listed resource has been verified within the last 90 days.
- **SC-011**: v1 cannot be found or installed from any app storefront outside Australia.
- **SC-012**: 100% of people see the data-handling notice (FR-031) and give or decline consent
  (FR-041) before any Ask question is sent.

## Assumptions

- **Scope is v1 Learn + Ask + Journal.** Śaṅkara's commentary, chapters 2–4 of the Kārikā as
  lessons, a browsable library of verses not yet taught, streaks, community, and accounts are
  deferred and will not be built here.
- **Why chapter 1 first.** The first chapter (Āgama Prakaraṇa) contains all twelve mantras of the
  Upaniṣad with Gauḍapāda's commentary on them, so finishing it means having studied the whole
  Upaniṣad through Gauḍapāda's eyes. It is about 41 verses, which fits roughly 30 lessons — about a
  month at one a day. Chapters 2–4 (about 186 further kārikās) are natural follow-on paths in later
  versions and a candidate paid offering.
- **One lesson a day** keeps the experience calm and builds a daily habit. People who want more can
  re-read completed lessons; there is no way to skip ahead in v1.
- **Lesson content is the main v1 dependency.** It needs a licensed (or public-domain) translation of
  the chapter, transliteration, plain-language explanations written for this app, and review by
  someone grounded in the tradition before release, per the constitution's textual-integrity
  principle. The explanations are applied commentary and are kept separate from the canonical text.
- **Design.** The approved design covers Ask and Journal. Learn (today's lesson, progress, completed
  lessons, completion moment) and the reminder setting need designs in the same visual language
  before they can be built.
- **The Ask verses need checking.** The four hard-coded verse references from the design handoff must
  be proof-read before release; they may need correcting.
- **Monetization.** v1 is free. Charging, if any, is a later decision (likely for further paths).
- **Platform.** v1 is a single cross-platform mobile app; a person uses it on one device, and progress
  and the Journal are that device's local data (no cross-device sync in v1).
- **Reflection generation uses an external language model reached only through the project's own
  backend**, per the constitution — so no model credentials live in the app. This is reconciled with
  device-only privacy: the Ask question is sent transiently to prepare one reflection, while lessons,
  responses and the Journal are never uploaded or stored server-side in v1.
- **Suggestion chips** cover the common everyday problem types (e.g., a relationship, stress, anger, a
  decision, loneliness) and pre-fill editable starting sentences.
- **Crisis detection in v1** is a dependable, rule-based gate sufficient for an early release,
  enforced before any reflection is prepared and on saved lesson responses; it favours showing
  support resources when uncertain. For lesson responses it runs entirely on the device.
- **Australia only for v1; India next.** A single English-speaking market with one set of crisis
  services and one privacy law keeps the first release's safety and compliance work manageable.
  Australia's Indian community is one of its largest overseas-born communities, so the core audience
  is present. India (v2) will need its own crisis resources, its own privacy-law review, and possibly
  Indian languages; other countries follow after that. Each region will get its own specification.
- **Australian crisis resources (to be verified before release, FR-030)**: emergency services 000;
  Lifeline 13 11 14 (also text and online chat); Kids Helpline 1800 55 1800 (for young people);
  Beyond Blue 1300 22 4636 (anxiety and depression support).
- **Australian privacy law.** Problem text about feelings and relationships is treated as sensitive,
  potentially health-related information under the Privacy Act 1988 and the Australian Privacy
  Principles, including the rules on sending personal information overseas. The data-handling notice
  (FR-031) and consent (FR-041) support this; the privacy approach will get a legal review before
  release.
- **App store rules.** Apple's App Review Guideline 5.1.2(i) (updated November 2025) requires
  disclosing and getting explicit permission before sharing personal data with a third-party AI
  service; FR-041 meets it.
- **Intended audience** is adults (18+). The app is not marketed to minors; a youth resource is still
  listed because younger people may reach it.
- **No therapeutic claims** (FR-027) also keep the app outside medical-device regulation in
  Australia; store listings and copy must avoid treatment or diagnosis language.
- **Reasonable-length input** is expected (a sentence to a short paragraph); extremely long input is
  handled gracefully.
- **Standard mobile performance and error-handling expectations** apply; the in-progress state and
  built-in fallback keep the person from ever hitting a dead-end.
