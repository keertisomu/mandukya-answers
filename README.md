# Turiya — learn the Māṇḍūkya Upaniṣad

A calm mobile app for learning the **Māṇḍūkya Upaniṣad**, mantra by mantra, with **Gauḍapāda's
Kārikā** — its oldest commentary — to help understand each one.

- **Learn**: about 15 lessons on the Upaniṣad's 12 mantras, phrase by phrase. Each phrase has its
  Sanskrit, an easy pronunciation guide, its meaning and a plain-words note. Then "What Gauḍapāda
  says": his kārikās on that mantra, with notes. Each mantra ends with the full text, a short
  summary and a small reflection for your day.
- **Saved**: keep the verses that matter to you — mantra 7, say — at the top, and re-read them each
  morning.
- **Journal**: your replies to each lesson, kept only on your phone.

v1 runs **entirely on your phone** — no account, no server, no AI, no network. It launches in
**Australia and India**. **Ask** (reflections on everyday problems) comes in v2.

**Status**: planning — no application code yet.

## Principles

- **Private**: nothing you write ever leaves your phone.
- **Faithful**: every verse and note is proofread and reviewed, never written by AI.
- **Safe**: if a journal reply signals crisis, real mental health services for your country appear.
  A small **Support** item in every header offers mental health support and a way to send feedback.
- **Calm**: no advice, no verdicts, no streaks.

## Docs — where to look

| Question | Read |
|---|---|
| What does v1 do? | [Spec](specs/001-turiya-learn/spec.md) — the source of truth |
| How is it built? | [Architecture](docs/ARCHITECTURE.md), then the detailed [plan](specs/001-turiya-learn/plan.md), [data model](specs/001-turiya-learn/data-model.md), [research](specs/001-turiya-learn/research.md) |
| How are lessons written? | [Lesson file format](specs/001-turiya-learn/contracts/lesson-file.md) |
| How do I run and test it? | [Quickstart](specs/001-turiya-learn/quickstart.md) |
| In what order do we build it? | [Implementation plan](docs/IMPLEMENTATION_PLAN.md), then the [task list](specs/001-turiya-learn/tasks.md) (80 tasks) |
| Why was it decided this way? | [Decisions](docs/DECISIONS.md) |
| What comes after v1? | [Roadmap](docs/ROADMAP.md) and the [Ask v2 design](docs/ASK_V2_DESIGN.md) |
| Is there a market? | [Market validation](docs/MARKET_VALIDATION.md) |
| What are the project's rules? | [Constitution](.specify/memory/constitution.md) |
| What does the visual style look like? | [Design handoff](design/README.md) |

## Needs a person before launch

- Translations of the 12 mantras and of Kārikā chapter 1, and their licences.
- Authors for the phrase meanings and the kārikā notes (in our own words), and someone grounded
  in the tradition to review them.
- Proofreading of all Sanskrit and pronunciation guides.
- Designs for Learn, the lesson layout, Saved and settings.
- Verifying the Australian and Indian helpline numbers.
- An email address for feedback.
