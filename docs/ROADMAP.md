# Roadmap

What comes after v1, in order. v1 itself is defined in the
[spec](../specs/001-turiya-ask-journal/spec.md).

| Version | Theme | Status |
|---|---|---|
| **v1** | Learn (chapter 1 Path), Ask, Journal — Australia only | Specified |
| **v1.1** | Deeper Ask: follow-ups and patterns | Planned |
| **v2** | India, and more of the Kārikā | Planned |
| **Later** | Other countries and ideas below | Not committed |

## v1.1 — deeper Ask (soon after launch)

**Goal**: make Ask a thread, not a one-off. Everything runs on the phone, so the server learns
nothing new (DEC-9).

**Follow-up check-ins**
- 3 days after an Ask, one local notification: *"Something you were sitting with a few days ago —
  how does it look now?"* It never contains the question.
- Tapping it opens that entry with a short reply box. The reply is saved under the original answer.
- Permission is asked for after the first answer, or reused from the daily reminder. There is an
  off switch.
- Never sent for crisis questions. At most one per entry.

**Pattern mirror**
- Each reflection also names the "story" behind the problem:
  - a key from a fixed list (e.g. "not respected", "must fix it alone", "not good enough");
  - plus a short phrase in the person's own words.
- Once there are 5 or more labelled entries, the Journal shows recurring stories by count, e.g.
  "This story has come up 4 times." It is worded as noticing, never as diagnosis.
- The list of stories is reviewed for tone before release.

**How we'll know it worked**: week-4 retention for v1.1 installs vs v1 installs, from App Store
Connect and Play Console. No in-app analytics.

## v2 — India, and more of the Kārikā

- **India launch** (DEC-10):
  - Indian support services;
  - a review under the Digital Personal Data Protection Act 2023;
  - possibly Indian languages.
  - It gets its own spec.
- **Paths for Kārikā chapters 2–4** (about 186 more kārikās). These are the natural paid offering.
  Each Path is a new `content/` folder, so no new screens are needed.
- **Daily four-states rhythm**, which the text itself suggests:
  - morning (jāgrat: the facts);
  - evening (svapna: the story I ran);
  - bedtime (suṣupti: letting go).
- **Bedtime wind-down.**

## Later — ideas, not commitments

- Other countries, each with its own spec.
- Śaṅkara's commentary.
- A library of verses not yet taught.
- **Audio of each Sanskrit verse**, so beginners can learn to recite it:
  - recorded by a trained chanter (not text-to-speech, which gets Sanskrit stress and long vowels
    wrong);
  - at a slow learning pace and at natural pace;
  - ask the lesson reviewer to recommend a chanter early, even though audio comes later.
- Guided reflection audio.
- An opt-in encrypted backup to the person's own iCloud or Google Drive, so a lost phone doesn't
  lose the Journal. It needs no account with us.

## Not planned

- **Streaks**: they punish missed days, which clashes with the teaching.
- **Community**: moderation cost and privacy risk.
- **Open-ended chat**: it drifts toward an "AI guru" persona.
- **Accounts**: they break the device-only promise. Any change needs an explicit decision
  (constitution, Principle VI).
