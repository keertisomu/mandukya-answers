# Roadmap

What comes after v1, in order. v1 itself is defined in the [spec](../specs/001-turiya-learn/spec.md).

| Version | Theme | Status |
|---|---|---|
| **v1** | Learn (12 mantras, phrase by phrase, with Gauḍapāda), Saved, Journal — Australia and India, fully offline | Specified |
| **v2** | **Ask**: reflections on everyday problems through the four states | Designed ([ASK_V2_DESIGN.md](./ASK_V2_DESIGN.md)) |
| **Later** | More of the Kārikā, audio, and the ideas below | Not committed |

## v2 — Ask

The full design is in [ASK_V2_DESIGN.md](./ASK_V2_DESIGN.md). In short:

- **What**: a person describes a problem and gets a reflection through the four states of
  consciousness, with fixed verses, linked to the lessons they've learned.
- **What it adds**: a server, an AI provider (Anthropic), search over the Kārikā, crisis and output
  safety checks, a consent screen and a golden eval set.
- **Privacy**: v1's "nothing leaves the phone" changes to "only your Ask question leaves the phone,
  with your consent".
- **Afterwards**: follow-up check-ins and a pattern mirror deepen Ask (design §6).

## Later — ideas, not commitments

- **Paths for Kārikā chapters 2–4** (about 186 more kārikās). These are the natural paid offering.
  Each is a new `content/` folder, so no new screens are needed.
- **Audio of each Sanskrit phrase and verse**, so beginners can learn to recite:
  - recorded by a trained chanter (not text-to-speech, which gets Sanskrit stress and long vowels
    wrong);
  - at a slow learning pace and at natural pace;
  - ask the lesson reviewer to recommend a chanter early.
- **Other teachers' renderings side by side** for each phrase, as in the mantra 7 study, if their
  publishers give permission.
- **Daily four-states rhythm**: morning (jāgrat), evening (svapna), bedtime (suṣupti).
- Śaṅkara's commentary.
- **Hindi and other Indian languages**, for lessons and the app itself.
- Other countries, each with its own spec.
- An opt-in encrypted backup to the person's own iCloud or Google Drive, so a lost phone doesn't
  lose the Journal. It needs no account with us.

## Not planned

- **Streaks**: they punish missed days, which clashes with the teaching.
- **Community**: moderation cost and privacy risk.
- **Open-ended chat**: it drifts toward an "AI guru" persona.
- **Accounts**: they break the device-only promise. Any change needs an explicit decision
  (constitution, Principle VI).
