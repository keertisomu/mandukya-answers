# Feature Specification: Turiya v1 — Learn the Māṇḍūkya

**Feature Branch**: `001-turiya-learn` · **Created**: 2026-10-17 · **Status**: Draft

**Input**:
- The original design handoff (`design/`).
- Scope changes since:
  - Australia only (2026-10-24, DEC-10);
  - learning first (2026-11-06, DEC-11);
  - **v1 is learning only, and Ask moves to v2** (2026-11-20, DEC-12);
  - **the Path teaches the Upaniṣad's 12 mantras, with Gauḍapāda's kārikās inside each lesson to
    explain them** (2026-11-23).
- The lesson format is modelled on a phrase-by-phrase study of Māṇḍūkya mantra 7 shared by the
  product owner.

## Overview

Turiya is a calm mobile app for **learning the Māṇḍūkya Upaniṣad**, mantra by mantra, with
**Gauḍapāda's Māṇḍūkya Kārikā** — the oldest commentary on it — to help understand each one.

A person reads one short lesson at a time, at their own pace. Each lesson takes a mantra **phrase by
phrase**, then turns to Gauḍapāda:
- what each phrase says, how to pronounce it, and what it means;
- **What Gauḍapāda says**: his kārikās on that mantra, with a plain note on how each one makes it
  clearer;
- the whole mantra, and a small reflection for the day.

People can save the lessons that matter most to them — mantra 7, for example, which clears away
wrong ideas about consciousness — and return to them every morning.

- **Learn** (the home screen): the Path of lessons, plus the person's saved lessons.
- **Journal**: what the person writes in reply to each lesson, kept only on their phone.

The app should feel quieting the moment it opens. It teaches; it never advises, judges or sounds
negative.

**v1 needs no server, no AI and no network.** Everything ships inside the app, and nothing a person
writes ever leaves their phone.

**In v1**: Learn and Journal, offered in **Australia only**.

**Not in v1**:
- **Ask** — reflections on everyday problems, planned for v2 (`docs/ASK_V2_DESIGN.md`);
- Kārikā chapters 2–4, which are independent treatises rather than commentary on the mantras;
- Śaṅkara's commentary;
- audio;
- streaks, community, accounts, sync.

Later plans are in `docs/ROADMAP.md`.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Learn a mantra phrase by phrase, with Gauḍapāda's help (Priority: P1)

The person opens the app to their next lesson. The mantra is broken into short phrases. For each
phrase they see:
- the Sanskrit;
- an easy pronunciation guide;
- its meaning;
- a short plain-words note on what it points to.

Phrases are grouped under simple headings where a mantra has natural parts.

Then comes **What Gauḍapāda says**: the one or two of his kārikās that most help with this mantra,
each with a plain note on what it adds. The rest of his kārikās on the mantra are a tap away, under
"More from Gauḍapāda".

The lesson ends with the whole mantra — in Devanagari, transliteration, pronunciation and
translation — then a short summary in plain words, and a "live it today" prompt. When they finish,
they can go straight on to the next lesson or stop for the day.

**Why this priority**: this is the product. On its own it delivers the core value: understanding
the Upaniṣad, line by line, through its oldest commentary.

**Independent Test**: Open lesson 1 and read every phrase. Confirm each shows Sanskrit,
pronunciation, meaning and note. Then confirm the Gauḍapāda section (where the mantra has kārikās),
the full mantra, the summary and the prompt. Finish the lesson, continue straight to lesson 2, and
confirm progress shows "2 of N".

**Acceptance Scenarios**:

1. **Given** a first launch, **When** the app opens, **Then** lesson 1 (mantra 1) is shown.
2. **Given** a lesson, **When** the person reads it, **Then** each phrase shows its Devanagari,
   transliteration, pronunciation guide, meaning and note, in the mantra's order.
3. **Given** a mantra that Gauḍapāda comments on, **When** the person reaches "What Gauḍapāda says",
   **Then** they see the most helpful kārikā(s). Each shows Devanagari, transliteration,
   pronunciation, translation, source and a plain note on how it clarifies the mantra.
4. **Given** that section, **When** the person taps "More from Gauḍapāda", **Then** every other kārikā
   on that mantra is shown the same way. It's collapsed by default and not needed to finish the
   lesson.
5. **Given** the end of a mantra's last lesson, **When** the person reaches it, **Then** they see the
   full mantra (Devanagari, transliteration, pronunciation, translation, reference, translation
   source), a short plain-words summary, the related state of consciousness where there is one,
   and a "live it today" prompt.
6. **Given** a long mantra such as mantra 7, **When** the person studies it, **Then** it is spread
   over several lessons, one for each natural part. The full mantra comes at the end of the last
   part.
7. **Given** a finished lesson, **When** the person completes it, **Then** they can go straight on or
   stop, with a gentle line such as "The next lesson is ready whenever you are."
8. **Given** time away, **When** the person returns, **Then** they continue from their next lesson,
   with no message about missed days.
9. **Given** the last lesson is finished, **When** the person completes it, **Then** they see a gentle
   completion moment and can re-read any lesson or start the Path again.
10. **Given** no network connection, **When** the person uses the app, **Then** everything works.

---

### User Story 2 - Return to the mantras that matter (Priority: P1)

The person saves lessons they want to keep coming back to, such as mantra 7. Their saved lessons
sit at the top of the Learn screen, so each morning they can open one straight away and read it
again.

**Why this priority**: re-reading a key mantra daily is how this text is actually studied. It is a
main reason to open the app every day, even after the Path is finished.

**Independent Test**: Save a mantra 7 lesson (or any completed lesson), close the app, reopen it,
and open the saved lesson from the Learn screen in no more than two taps.

**Acceptance Scenarios**:

1. **Given** a completed lesson, **When** the person taps "Save", **Then** it appears in Saved on the
   Learn screen.
2. **Given** saved lessons, **When** the person opens the app, **Then** they can open any saved lesson
   in no more than two taps.
3. **Given** a saved lesson, **When** the person taps "Saved" again, **Then** it is removed from Saved.
4. **Given** saved lessons, **When** the app restarts, **Then** they are still saved.

---

### User Story 3 - Keep a private journal of reflections (Priority: P1)

Each lesson ends with a small prompt. The person can write a few lines in reply. Their replies are
kept in a Journal, newest first, so they can look back on what each mantra stirred in them. The
Journal never leaves their phone.

**Why this priority**: writing turns reading into practice. The Journal is where a person sees
their own understanding grow.

**Independent Test**: Write a reply to a lesson prompt. Open the Journal and confirm it appears with
the lesson title and time, opens in full, links back to the lesson, and is still there after a
restart.

**Acceptance Scenarios**:

1. **Given** a lesson prompt, **When** the person writes a reply and saves it, **Then** it appears in
   the Journal with the lesson title, mantra reference and time.
2. **Given** no entries, **When** the person opens the Journal, **Then** a gentle empty state invites
   them to begin a lesson.
3. **Given** an entry, **When** the person opens it, **Then** they see their reply and the prompt, with
   a link to the lesson.
4. **Given** a reply, **When** the person changes it from the lesson, **Then** the Journal shows the new
   version.

---

### User Story 4 - Feel safe, private and never harmed (Priority: P1)

Even a learning app receives people's private thoughts in their journal replies. If someone writes
something that signals crisis or self-harm, the app gently shows real support services. Help is
always one tap away. No text in the app gives advice, judges or sounds negative. Nothing the person
writes ever leaves their phone.

**Why this priority**: these are constitutional guarantees. One harmful line, a misquoted verse or a
leak of private writing would break the product's promise.

**Independent Test**:
- Write crisis wording as a journal reply while offline, and confirm support services appear and
  nothing is sent.
- Open help from each tab.
- Inspect outbound traffic during normal use and confirm the app sends nothing.

**Acceptance Scenarios**:

1. **Given** a reply that signals crisis or self-harm, **When** saved, **Then** it is kept and
   Australian support services are shown, including 000 and Lifeline, with nothing sent anywhere.
2. **Given** any screen, **When** the person wants help, **Then** support services are one tap away,
   offline too.
3. **Given** any text in the app, **When** read, **Then** it teaches and reflects, never advises,
   judges or implies harm, and is never negative.
4. **Given** normal use, **When** traffic is inspected, **Then** the app sends nothing about the
   person.

---

### User Story 5 - Get a gentle daily reminder (Priority: P3)

A person who wants help with the habit can choose a time for a quiet daily reminder to read. It is
off unless they turn it on.

**Why this priority**: reminders support a daily habit, but the app works fully without them.

**Independent Test**: Turn the reminder on, confirm it arrives at the chosen time on a day with no
reading done and shows no journal content, then turn it off and confirm it stops.

**Acceptance Scenarios**:

1. **Given** reminders were never turned on, **When** the app is used, **Then** no reminder is sent and
   notification permission is never asked for.
2. **Given** the person turns reminders on, **When** they pick a time, **Then** permission is asked for
   at that moment.
3. **Given** the person has already read a lesson today, **When** the reminder time comes, **Then** no
   reminder is sent.

---

### Edge Cases

- **A mantra with no kārikā of its own** (e.g. mantras 1–5, which Gauḍapāda discusses together
  later): the Gauḍapāda section is left out, or points ahead to where he discusses it.
- **Empty reply**: the lesson can be completed without writing; nothing goes to the Journal.
- **Long reply**: accepted and handled without visible errors.
- **Skipping "More from Gauḍapāda"**: allowed; the lesson still counts as finished.
- **Several lessons in one sitting**: allowed; nothing encourages or discourages it.
- **Saving a lesson not yet reached**: not possible. Only completed lessons, or the one being read,
  can be saved.
- **Re-reading the Path after finishing**: progress can restart; saved lessons and the Journal are
  kept.
- **App reinstalled or phone replaced**: progress, saved lessons and the Journal are lost, because
  they live only on the phone. The app's privacy information says so.
- **Storage unavailable**: the app still works; progress restarts at lesson 1, and Saved and the
  Journal show their empty states.
- **Travelling overseas**: the app keeps working. The support screen notes that 000 and Australian
  lines only work in Australia, and links to an international helpline directory.
- **Phone set to another language or region**: the app stays in Australian English with Australian
  services.

## Requirements *(mandatory)*

### Functional Requirements

**Lessons**

- **FR-001**: The Path MUST teach the **12 mantras of the Māṇḍūkya Upaniṣad**, in order.
- **FR-002**: Each lesson MUST present its mantra **phrase by phrase**. Each phrase shows:
  - Devanagari;
  - a standard transliteration;
  - an easy pronunciation guide (syllables split, long sounds shown plainly, e.g. "MAAN-doo-kya");
  - its meaning;
  - a short plain-words note.

  Where a mantra has natural parts, its phrases MUST be grouped under short headings (e.g. mantra
  7: "Not the knower", "Not the known", "What it is").
- **FR-003**: A mantra too long for one sitting MUST be spread over several lessons, one per part.
  A lesson should take about 3–5 minutes to read, not counting "More from Gauḍapāda".
- **FR-004**: **All 29 kārikās of Gauḍapāda's chapter 1 (Āgama Prakaraṇa) MUST appear in the Path**,
  each with the mantra it explains — normally the one it follows in the text (e.g. kārikās
  1.10–1.18 with mantra 7). Each kārikā shows:
  - Devanagari;
  - a transliteration;
  - a pronunciation guide;
  - an English translation and its source;
  - a plain note on how it helps understand the mantra.

  The one or two most helpful kārikās appear under **What Gauḍapāda says**; the rest are under an
  expandable **More from Gauḍapāda**, collapsed by default.
- **FR-005**: Each mantra MUST end with:
  - the full mantra (Devanagari, transliteration, pronunciation, English translation, reference and
    the translation's source);
  - a short plain-words summary;
  - the related state of consciousness, where there is one;
  - a "live it today" prompt.
- **FR-006**: All lesson content MUST be written, proofread and reviewed before release, and MUST
  NOT be generated when the person reads it.
- **FR-007**: Lessons MUST open in order. After a lesson, the person MUST be able to go straight on or
  stop; there is no daily limit and no skipping ahead. When they stop, a gentle line MUST say the
  next lesson is ready whenever they are. Time away MUST NOT reset progress or bring any message
  about missed days.
- **FR-008**: Learn MUST show progress (e.g. "Lesson 7 of 15") and let the person re-read any
  completed lesson. Finishing the last lesson MUST lead to a completion moment with the option to
  re-read or start again.

**Saved lessons**

- **FR-009**: The person MUST be able to save and unsave any completed lesson, or the one they are
  reading.
- **FR-010**: Saved lessons MUST appear at the top of the Learn screen, each openable in no more than
  two taps from opening the app.

**Journal**

- **FR-011**: The person MUST be able to write an optional reply to a lesson's prompt. Replies are
  saved to the Journal with the lesson title, mantra reference and time.
- **FR-012**: The Journal MUST list replies newest first, with a relative time and the lesson title,
  and show a gentle empty state when there are none.
- **FR-013**: Opening an entry MUST show the reply and the prompt, with a link to the lesson.
- **FR-014**: The Journal MUST be read-only. A reply can be changed only from its lesson.

**App and feel**

- **FR-015**: The app MUST have two tabs — Learn and Journal — and open to Learn, showing saved
  lessons and the next lesson.
- **FR-016**: Progress, saved lessons, Journal replies and settings MUST survive app restarts.
- **FR-017**: The whole app MUST work without a network connection.
- **FR-018**: The app MUST follow the approved visual style, with touch targets of at least 44px.
- **FR-019**: Motion MUST be subtle, and reduced or off when the device asks for reduced motion.
- **FR-020**: Times MUST read "Today · h:mm", "Yesterday · h:mm", "N days ago" within a week, then
  "24 Oct" style.
- **FR-021**: All copy MUST use Australian English.
- **FR-022**: The person MUST be able to turn on an optional daily reminder at a time they choose. It
  is off by default; permission is asked for only when they turn it on; no reminder is sent on a day
  they have already read a lesson; reminder text MUST NOT include journal content; it can be changed
  or turned off at any time.

**Safety**

- **FR-023**: Journal replies MUST be checked for signs of crisis or self-harm **on the phone**. If
  found, the reply is kept and support services MUST be shown.
- **FR-024**: Support services MUST be one tap away from Learn and Journal, and work offline.
- **FR-025**: No text in the app MUST give a verdict, diagnosis or advice about the person's life, or
  imply harm to anyone. All text MUST be warm and steadying, never negative.
- **FR-026**: The app MUST present itself as teaching the Upaniṣad with the help of Gauḍapāda's
  Kārikā — not as therapy, counselling, treatment or religious authority.

**Privacy**

- **FR-027**: Nothing a person writes or does MUST leave the phone: no accounts, no server, no
  analytics, no tracking.
- **FR-028**: The app MUST say that everything is saved only on this phone, and that deleting the app
  deletes it.

**Australia**

- **FR-029**: v1 MUST be offered only in the Australian App Store and Google Play.
- **FR-030**: Support services MUST be Australian and include at least 000, Lifeline, a service for
  young people and one for anxiety and depression, each with how to contact it and when.
- **FR-031**: Every support service MUST be verified before release and every 90 days after.

### Key Entities *(include if feature involves data)*

- **Path**: the ordered lessons for the 12 mantras. The same for everyone.
- **Lesson**: one mantra, or one part of a long mantra, with its phrases. The lesson that ends a
  mantra also carries its kārikās, the full mantra, summary, related state and prompt. Fixed,
  reviewed content.
- **Phrase**: a piece of a mantra — Devanagari, transliteration, pronunciation, meaning, note.
- **Mantra**: one of the Upaniṣad's 12 verses — Devanagari, transliteration, pronunciation,
  translation, reference and translation source.
- **Kārikā**: one of Gauḍapāda's 29 verses on the mantras — Devanagari, transliteration,
  pronunciation, translation, source and a note — attached to the mantra it explains, and marked as
  shown in the lesson or under "More from Gauḍapāda".
- **Progress**: which lessons are done, and when. Phone only.
- **Saved lesson**: a lesson the person keeps handy. Phone only.
- **Journal reply**: what the person wrote for a lesson, with the time. Phone only.
- **Reminder setting**: on or off, and a time. Phone only.
- **Support service**: name, contact methods, hours and the date it was last verified.

## Success Criteria *(mandatory)*

### Measurable Outcomes

**Learning**

- **SC-001**: In first-use testing, at least 90% of people finish lesson 1 without help, and the
  median lesson takes 3–5 minutes.
- **SC-002**: In first-use testing, most people can explain in their own words what at least one
  phrase of the lesson means, and how Gauḍapāda's kārikā helped.
- **SC-003**: Before release, all of the following hold:
  - all 12 mantras and all 29 kārikās are in the Path;
  - 100% of Sanskrit, transliterations and pronunciation guides are proofread against a Sanskrit
    source;
  - 100% of meanings, notes and summaries are reviewed by someone grounded in the tradition;
  - every mantra and kārikā names its translation source.

**Coming back**

- **SC-004**: People open a saved lesson within two taps of opening the app, every time.
- **SC-005**: Day-30 retention reported by the app stores is at least 12% in the first 60 days
  (meditation apps typically see 6–8%).
- **SC-006**: In first-use testing, most people say they would come back to read again tomorrow.

**Safety, privacy and reliability**

- **SC-007**: 100% of crisis wording in journal replies leads to support services.
- **SC-008**: 100% of replies, saved lessons and progress survive a restart.
- **SC-009**: 100% of the app works with no network connection.
- **SC-010**: Inspection of outbound traffic shows the app sends nothing about the person.

**Feel and region**

- **SC-011**: In first-use testing, most people describe the app as calming.
- **SC-012**: 100% of support screens show Australian services verified in the last 90 days.
- **SC-013**: v1 can't be found or installed outside Australia.

## Assumptions

- **The Upaniṣad is the subject; the Kārikā is the guide.**
  - Chapter 1 of Gauḍapāda's Kārikā (Āgama Prakaraṇa) comments directly on the mantras, and its
    kārikās follow the mantras they explain: 1.1–1.9 after mantra 6, 1.10–1.18 after mantra 7,
    1.19–1.23 after mantra 11, and 1.24–1.29 after mantra 12.
  - Authors may attach a kārikā to an earlier mantra it illuminates better, as long as all 29
    appear exactly once.
  - Chapters 2–4 are independent treatises and come later (ROADMAP).
- **About 15 lessons**: the 12 mantras, with long ones (like mantra 7, about 19 phrases) spread over
  two or three lessons. That's roughly two to three weeks at a lesson a day.
- **Pace**: people go at their own pace. A lesson a day is suggested, helped by the reminder, but not
  enforced. Lessons stay in order so the text is read as written.
- **Meanings and notes are written for this app.** Modern translations and teachers' commentaries
  (e.g. Gambhirananda, Nikhilananda, Chinmayananda, Sarvapriyananda) are mostly still in copyright.
  They can guide the authors, but they can't be quoted without permission.
- **Translations**: full mantras and kārikās use a public-domain translation, or one used with
  permission. The mantras and the Kārikā may need different sources (see the open questions in
  `docs/DECISIONS.md`).
- **Lesson content is the main dependency**:
  - phrase breakdowns, meanings, notes, summaries and prompts for the 12 mantras;
  - notes for all 29 kārikās;
  - review by someone grounded in the tradition.
- **Pronunciation guide**: one simple, consistent respelling scheme, checked by the Sanskrit
  proofreader. Audio comes later.
- **Design**: the approved visual style (colours, type, calm feel) from `design/` applies. The Learn
  screens, lesson layout (including the Gauḍapāda section), Saved list and reminder setting need
  new designs in that style.
- **v1 is free.**
- **One phone**: no accounts and no sync.
- **No server in v1**: crisis checking of replies uses word and phrase lists on the phone, and leans
  toward showing support when unsure.
- **Australia first** (DEC-10). Ask (v2) and India are the next steps; their order is still open.
- **Australian support services** (verify before release): 000; Lifeline 13 11 14 (also text and
  chat); Kids Helpline 1800 55 1800; Beyond Blue 1300 22 4636.
- **Privacy law**: with nothing leaving the phone, the Privacy Act's overseas-transfer rules don't
  apply to v1. The privacy policy still explains device-only storage.
- **Audience**: adults (18+). A youth service is still listed because younger people may arrive.
- **No therapy claims** (FR-026) keep the app clearly outside medical-device rules.
