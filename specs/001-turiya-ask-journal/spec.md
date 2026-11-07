# Feature Specification: Turiya v1 — Learn, Ask & Journal

**Feature Branch**: `001-turiya-ask-journal` · **Created**: 2026-10-17 · **Status**: Draft

**Input**: The design handoff (`design/README.md`, `design/Turiya.dc.html`), then two scope
changes: launch in Australia only (2026-10-24, DEC-10) and centre v1 on learning the Upaniṣad
(2026-11-06, DEC-11).

## Overview

Turiya is a calm mobile app for **learning the Māṇḍūkya Upaniṣad through Gauḍapāda's Māṇḍūkya
Kārikā**, one short lesson a day, and for applying it to everyday life.

- **Learn** (the home screen): a Path of daily lessons. Each lesson teaches one verse in plain words
  and ends with a small reflection for the person's own day.
- **Ask**: the person describes something weighing on them and gets a reflection through the four
  states of consciousness — waking, dream, deep sleep and the witness — with one fixed verse per
  state.
- **Journal**: lesson responses and Ask reflections, kept privately on the device.

The app should feel quieting the moment it opens. It never advises, judges or sounds negative.

**In v1**: Learn, Ask and Journal, offered in **Australia only**.

**Not in v1**: Kārikā chapters 2–4 as lessons, Śaṅkara's commentary, a library of verses not yet
taught, streaks, community, accounts, cross-device sync. Later plans are in `docs/ROADMAP.md`.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Learn the Upaniṣad one lesson a day (Priority: P1)

The person opens the app to today's lesson. It shows one verse — in Devanagari, with a
transliteration, an English translation and its reference — then a plain-language explanation of
what Gauḍapāda says, the state of consciousness it concerns, and a "live it today" prompt. They can
write a few lines in reply, which go to their Journal. The next lesson opens the following day.

**Why this priority**: Learning is the centre of the product and the main reason to come back each
day. It works on its own, even without Ask.

**Independent Test**: Complete lesson 1 and write a response. Confirm the response is in the Journal
and lesson 2 is locked until tomorrow. The next day, open lesson 2 and confirm progress shows
"2 of 30".

**Acceptance Scenarios**:

1. **Given** a first launch, **When** the app opens, **Then** the first lesson is shown.
2. **Given** an open lesson, **When** the person reads it, **Then** they see the verse (Devanagari,
   transliteration, translation, reference, translation source), the explanation, the related
   state where there is one, and the prompt.
3. **Given** a lesson, **When** the person writes and saves a response, **Then** it appears in the
   Journal with the lesson title and verse reference.
4. **Given** a lesson completed today, **When** the person looks for the next one, **Then** it shows
   as available tomorrow, and the completed lesson can still be re-read.
5. **Given** several days away, **When** the person returns, **Then** they continue from the next
   lesson with no message about missed days.
6. **Given** the final lesson is completed, **When** the person finishes it, **Then** they see a gentle
   completion moment and can re-read lessons or start the Path again.
7. **Given** no network connection, **When** the person uses Learn, **Then** everything works.

---

### User Story 2 - Ask about a problem (Priority: P2)

The person types what is weighing on them, or taps a suggestion to start a sentence, and submits.
After a short, calming wait they see a reflection in four fixed states. Each state has a short
heading, a few sentences about their situation and a fixed verse. One small "one thing to try"
line closes the reflection. Where a verse is also a lesson they have reached, they can open it.

**Why this priority**: Ask is how the teaching meets a real day, and a reason to open the app when
something is wrong. It is secondary to Learn but works on its own.

**Independent Test**: Ask a question and confirm a four-state reflection, in order, with the correct
verse per state and one practice line. It reads as warm, not advice, and is saved to the Journal.

**Acceptance Scenarios**:

1. **Given** the Ask screen, **When** the person submits a problem, **Then** a calming wait is shown,
   followed by four states in this order: Waking, The story, Underneath, The witness.
2. **Given** a reflection, **When** the person reads it, **Then** each state shows its label, heading,
   body and fixed verse, and the first state is open.
3. **Given** a reflection, **When** the person reads the practice line, **Then** it is a small
   reflective action, not an instruction to fix the problem.
4. **Given** a suggestion chip, **When** the person taps it, **Then** the input is pre-filled with an
   editable first-person sentence.
5. **Given** an empty input, **When** the person submits, **Then** nothing happens.
6. **Given** a reflection whose verse is a lesson the person has reached, **When** they choose to
   learn more, **Then** that lesson opens.

---

### User Story 3 - Look back in the Journal (Priority: P1)

Everything the person writes or receives is kept in one Journal, newest first: lesson responses
and Ask reflections. They can open any entry to re-read it. Nothing in the Journal can be edited
or re-asked, and it stays on the device.

**Why this priority**: The Journal turns daily lessons and one-off questions into a practice the
person can look back on.

**Independent Test**: Save one lesson response and complete one Ask. Confirm both appear newest
first, open in full, and are still there after restarting the app.

**Acceptance Scenarios**:

1. **Given** entries of both kinds, **When** the person opens the Journal, **Then** they are listed
   newest first with a relative time and the lesson title or question, and the two kinds are easy
   to tell apart.
2. **Given** no entries, **When** the person opens the Journal, **Then** a gentle empty state invites
   them to start the Path or ask something.
3. **Given** an entry, **When** the person opens it, **Then** it shows exactly what was saved; a lesson
   entry also links to its lesson.
4. **Given** a restart, **When** the person reopens the app, **Then** all entries are still there.

---

### User Story 4 - Feel safe, private and never harmed (Priority: P1)

People bring their rawest problems here. If they express crisis or self-harm, in Ask or in a
lesson response, the app shows real support services. Nothing in the app gives a verdict or
advice, endorses harm, or sounds negative. If a reflection can't be prepared, a steadying built-in
one is shown instead. Lessons and lesson responses never leave the device; only an Ask question is
sent, and only after the person agrees.

**Why this priority**: These are constitutional guarantees. One harmful line, misquoted verse or
leak of private data breaks the product's promise.

**Independent Test**: Enter crisis wording in Ask and in a lesson response, and confirm support
services appear both times. Invite a verdict and confirm the reply stays reflective. Go offline
and confirm Ask shows the built-in reflection. Confirm no lesson response leaves the device.

**Acceptance Scenarios**:

1. **Given** crisis wording in Ask, **When** submitted, **Then** no reflection is prepared and
   Australian support services are shown, including 000 and Lifeline.
2. **Given** crisis wording in a lesson response, **When** saved, **Then** the response is kept and the
   same services are shown, with nothing sent off the device.
3. **Given** any screen, **When** the person wants help, **Then** support services are one tap away.
4. **Given** any text in the app, **When** read, **Then** it is warm, gives no verdict or advice, and
   implies no harm.
5. **Given** a failed or invalid reflection, **When** the person submits, **Then** a built-in steadying
   reflection is shown, not an error.
6. **Given** a first Ask, **When** the person submits, **Then** they are asked to agree to the
   question being sent to the AI service before anything is sent.

---

### User Story 5 - Get a gentle daily reminder (Priority: P3)

A person who wants help with the habit can pick a time for a quiet daily reminder that their next
lesson is ready. It is off unless they turn it on.

**Why this priority**: Reminders help retention, but the Path works without them.

**Independent Test**: Turn the reminder on, confirm it arrives at the chosen time on a day with a
new lesson and shows no journal content, then turn it off and confirm it stops.

**Acceptance Scenarios**:

1. **Given** reminders were never turned on, **When** the app is used, **Then** no reminder is sent and
   no notification permission is asked for.
2. **Given** the person turns reminders on, **When** they pick a time, **Then** permission is asked for
   at that moment.
3. **Given** today's lesson was already opened, **When** the reminder time comes, **Then** no reminder is
   sent.

---

### Edge Cases

- **Empty lesson response**: the lesson can be completed without writing; nothing goes to the Journal.
- **Long text**: long questions and responses are accepted and handled without visible errors.
- **Crisis question in Ask**: no reflection is prepared, so nothing is saved to the Journal.
- **Invalid or incomplete reflection**: the built-in reflection is shown instead.
- **No connection**: Learn and Journal work fully; Ask shows the built-in reflection.
- **Device clock moved forward**: lessons unlock early. This is accepted, not prevented.
- **App reinstalled or phone replaced**: progress and the Journal are lost, because they live only on
  the device. The privacy information says so.
- **Storage unavailable**: the app still works; progress restarts at lesson 1 and the Journal shows
  its empty state.
- **Repeated taps on submit**: only one reflection is prepared at a time.
- **Consent declined**: Learn and Journal work; Ask explains it needs consent and can ask again.
- **Travelling overseas**: the app keeps working. The support screen notes that 000 and Australian
  lines only work in Australia and links to an international helpline directory.
- **Phone set to another language or region**: the app stays in Australian English with Australian
  services.

## Requirements *(mandatory)*

### Functional Requirements

**Learn**

- **FR-001**: The Path MUST cover chapter 1 of the Kārikā (Āgama Prakaraṇa) — the 12 mantras of the
  Upaniṣad with Gauḍapāda's 29 kārikās — in textual order, in about 30 lessons.
- **FR-002**: Each lesson MUST show its verse(s) in Devanagari, a transliteration, an English
  translation, the reference and the translation's source.
- **FR-003**: Each lesson MUST include a plain-language explanation (verse and explanation readable
  in 3–5 minutes), the related state where there is one, and a "live it today" prompt.
- **FR-004**: All lesson content MUST be written, proofread and reviewed before release. It MUST NOT
  be generated when the person reads it.
- **FR-005**: The person MUST be able to write an optional response to the prompt. It is saved to
  the Journal and never leaves the device.
- **FR-006**: One new lesson MUST become available per calendar day, after the previous one is
  completed. Missed days MUST NOT reset progress or produce any message. There is no skipping
  ahead.
- **FR-007**: Learn MUST show progress (e.g. "Lesson 7 of 30") and let the person re-read completed
  lessons. Finishing the last lesson MUST lead to a completion moment with the option to re-read or
  start again.
- **FR-008**: Learn MUST work fully without a network connection.
- **FR-009**: The person MUST be able to turn on an optional daily reminder at a time they choose. It
  is off by default; permission is asked for only when they turn it on; no reminder is sent if
  today's lesson was opened or no new lesson is available; reminder text MUST NOT include journal
  content; it can be changed or turned off at any time.

**Ask**

- **FR-010**: The person MUST be able to type a problem and submit it. Empty input MUST be ignored.
- **FR-011**: Suggestion chips MUST pre-fill the input with an editable first-person sentence.
- **FR-012**: A calming in-progress state MUST show while the reflection is prepared.
- **FR-013**: A reflection MUST have exactly four states in this order: Waking (Jāgrat), The story
  (Svapna), Underneath (Suṣupti), The witness (Turīya). The first is open; each can be opened and
  closed.
- **FR-014**: Each state MUST show a short heading and a body about the person's own situation,
  with that state's fixed verse (Devanagari, translation, reference). Verses MUST NOT be generated.
- **FR-015**: Each reflection MUST end with exactly one "one thing to try" line: a small reflective
  action, not an instruction to fix the problem.
- **FR-016**: After a reflection, the person MUST be able to return to a cleared Ask screen.
- **FR-017**: A reflection MUST link each verse to its lesson when the person has reached that
  lesson, and each lesson MUST link to Ask.

**Journal**

- **FR-018**: Every Ask reflection and every saved lesson response MUST be saved to the Journal with
  a timestamp, without a separate save step for Ask.
- **FR-019**: The Journal MUST list entries newest first, each with a relative time and the lesson
  title or question, the two kinds easy to tell apart, and a gentle empty state.
- **FR-020**: Opening an entry MUST show it exactly as saved; a lesson entry MUST link to its lesson.
- **FR-021**: The Journal MUST be read-only. A lesson response can be changed only from its lesson.
- **FR-022**: Journal entries and Path progress MUST survive app restarts.

**Navigation & feel**

- **FR-023**: The app MUST have three tabs — Learn, Ask, Journal — and open to Learn. Returning to Ask
  MUST show the latest reflection from the current session, if there is one.
- **FR-024**: The app MUST follow the approved design, with touch targets of at least 44px.
- **FR-025**: Motion MUST be subtle and reduced or off when the device asks for reduced motion.
- **FR-026**: Times MUST read "Today · h:mm", "Yesterday · h:mm", "N days ago" within a week, then
  "24 Oct" style.
- **FR-027**: All copy MUST use Australian English.

**Safety**

- **FR-028**: Ask MUST detect crisis or self-harm before preparing a reflection and, if found, show
  support services instead.
- **FR-029**: Saved lesson responses MUST be checked for crisis or self-harm on the device, without
  sending them anywhere; if found, the support services MUST be shown.
- **FR-030**: Support services MUST be one tap away from Learn, Ask and Journal, and work offline.
- **FR-031**: No text in the app MUST give a verdict, diagnosis or prescriptive advice, or endorse or
  imply harm to anyone.
- **FR-032**: All text MUST be warm and steadying, never negative, harsh, shaming or discouraging.
- **FR-033**: If a reflection fails or is invalid, a built-in steadying reflection and practice MUST
  be shown.
- **FR-034**: The app MUST NOT present itself as therapy, counselling, treatment or religious
  authority. Every reflection MUST say it is "a reflection drawing on Gauḍapāda's Kārikā — not
  religious authority, therapy, or medical advice."

**Privacy**

- **FR-035**: Lessons, lesson responses, progress and the Journal MUST stay on the device. Only Ask
  question text may be sent, once, to prepare a reflection. The app MUST NOT contain secrets for
  any service.
- **FR-036**: Before the first Ask question is sent, the app MUST explain in plain words that the
  question goes to a named AI service that may be outside Australia and does not store it, and
  that everything else stays on the device. The person MUST explicitly agree first. Until they do,
  nothing is sent and Learn and Journal still work. They can review the choice later.
- **FR-037**: The app MUST show that content is saved only on the device.

**Australia**

- **FR-038**: v1 MUST be offered only in the Australian App Store and Google Play.
- **FR-039**: Support services MUST be Australian and include at least 000, Lifeline, a service for
  young people and one for anxiety and depression, each with how to contact it and when.
- **FR-040**: Every support service MUST be verified before release and every 90 days after.

### Key Entities *(include if feature involves data)*

- **Lesson**: a verse (or a mantra with its kārikās) — Devanagari, transliteration, translation,
  reference, source — plus an explanation, the related state and a prompt. Fixed, reviewed content.
- **Path**: the ordered lessons for chapter 1. The same for everyone.
- **Path progress**: which lessons are done and when the next one opens. Device only.
- **Lesson response**: what the person wrote for a lesson, with a timestamp. Device only.
- **Question**: what the person asked in Ask. Sent once to prepare a reflection; saved on the device.
- **State**: one of the four fixed states, with its Sanskrit name, English label, colour and verse.
- **Reflection**: a heading and body per state, plus one practice line.
- **Journal entry**: a lesson response or a reflection, with a timestamp. Device only, read-only.
- **Reminder setting**: on or off, and a time. Device only.
- **Support service**: name, contact methods, hours and the date it was last verified.

## Success Criteria *(mandatory)*

### Measurable Outcomes

**Learn**

- **SC-001**: In first-use testing, at least 90% of people finish lesson 1 without help, and the median
  time for a lesson is 3–5 minutes.
- **SC-002**: Before release, 100% of verses are proofread against a Sanskrit source, 100% of
  explanations are reviewed by someone grounded in the tradition, and every lesson names its
  translation source.
- **SC-003**: 100% of Learn and Journal works without a connection.
- **SC-004**: Day-30 retention reported by the app stores is at least 12% in the first 60 days
  (meditation apps typically see 6–8%).
- **SC-005**: In first-use testing, most people say they would come back for tomorrow's lesson.

**Ask**

- **SC-006**: A person can go from opening Ask to reading a full reflection in under 30 seconds.
- **SC-007**: 100% of reflections show four states in the right order with the right verses.
- **SC-008**: In review, at least 95% of reflections read as warm and reflective, and 0% contain a
  verdict, advice, or harmful or negative content.
- **SC-009**: In first-use testing, at least 90% of people complete an Ask on their first try.

**Safety and privacy**

- **SC-010**: 100% of crisis wording, in Ask or lesson responses, leads to support services.
- **SC-011**: 100% of failed reflections lead to the built-in reflection, never an error.
- **SC-012**: 100% of reflections and lesson responses are saved and survive a restart.
- **SC-013**: Nothing leaves the device except Ask question text sent after consent (checked by
  inspecting outbound traffic).
- **SC-014**: 100% of people see the notice and give or refuse consent before any question is sent.

**Feel and region**

- **SC-015**: In first-use testing, most people describe the app as calming.
- **SC-016**: 100% of support screens show Australian services verified in the last 90 days.
- **SC-017**: v1 can't be found or installed outside Australia.

## Assumptions

- **Why chapter 1 first**: it holds all 12 mantras of the Upaniṣad with Gauḍapāda's commentary, so
  finishing it means having studied the whole Upaniṣad through his eyes. About 41 verses fit about
  30 lessons — roughly a month.
- **Pace**: one lesson a day keeps the app calm and builds a habit. Completed lessons can be re-read.
- **Lesson content is the main dependency**: a licensed or public-domain translation, proofread
  Sanskrit, explanations written for the app, and review by someone grounded in the tradition.
- **Design**: the approved design covers Ask and Journal. Learn, the reminder setting and the consent
  screen need designs in the same style.
- **The four Ask verses** from the design handoff must be proofread; some references may be wrong.
- **v1 is free.**
- **One device**: no accounts and no sync, so progress and the Journal live on one phone.
- **Reflections come from an AI model reached only through the project's own server**, so the app
  holds no keys. The question is sent once; nothing else is.
- **Crisis detection** leans toward showing support when unsure. For lesson responses it runs
  entirely on the device.
- **Australia first, India next** (DEC-10). Each new region gets its own spec.
- **Australian support services** (verify before release): 000; Lifeline 13 11 14 (also text and
  chat); Kids Helpline 1800 55 1800; Beyond Blue 1300 22 4636.
- **Australian privacy law**: questions about feelings and relationships are treated as sensitive
  information under the Privacy Act 1988, including its rules on sending data overseas. The notice
  and consent (FR-036) support this; a legal review is planned before release.
- **Apple's rules**: App Review Guideline 5.1.2(i) (November 2025) requires explicit permission
  before sharing personal data with a third-party AI service. FR-036 meets it.
- **Audience**: adults (18+). A youth service is still listed because younger people may arrive.
- **No therapy claims** (FR-034) also keep the app outside Australia's medical-device rules.
