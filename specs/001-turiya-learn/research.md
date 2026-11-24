# Research: Turiya v1 — Learn

Phase 0 output of `/speckit-plan`. Each item: **Decision**, **Why**, **Alternatives considered**.

v1 is an app with no server (DEC-12). Research for Ask's server, AI, search and hosting has moved
to [docs/ASK_V2_DESIGN.md](../../docs/ASK_V2_DESIGN.md). These are already decided in
[docs/DECISIONS.md](../../docs/DECISIONS.md):
- React Native + Expo (DEC-2);
- phone-only storage (DEC-1, DEC-3);
- Australia and India for v1 (DEC-10);
- learning first (DEC-11, DEC-12).

---

## R1 — App runtime and storage

- **Decision**: Expo SDK 58 (React Native 0.88, New Architecture), or the latest stable SDK at M0.
  `react-native-mmkv` for storage, using an **Expo development build** (EAS) from the start.
- **Why**: SDK 58 is current (October 2026), and since SDK 55, Expo supports only the New
  Architecture. MMKV is fast and simple for a few small JSON documents, but it's a native module, so
  it doesn't run in Expo Go.
- **Alternatives**: `AsyncStorage`, which runs in Expo Go but is slower and asynchronous. It's kept
  as the fallback.

## R2 — Navigation

- **Decision**: Expo Router with two tabs (Learn, Journal), plus stack screens for a lesson, the
  Support sheet (mental health support, feedback) and settings.
- **Why**: it's Expo's default. File-based routes keep the screen list obvious, and deep links
  (e.g. from the reminder) work out of the box.
- **Alternatives**: React Navigation set up by hand (more boilerplate for the same result).

## R3 — Lesson content format

- **Decision**: one Markdown file per lesson, with YAML front matter holding the phrases, verse and
  prompt (see [contracts/lesson-file.md](./contracts/lesson-file.md)). A build step:
  - validates each file with Zod;
  - turns notes and summaries into a few simple text blocks (paragraphs, emphasis);
  - bundles everything as one JSON module.
- **Why**:
  - The authors and reviewers aren't developers. Markdown reads well in a GitHub pull request and
    is easy to comment on.
  - Pre-built text blocks render with the app's own components, with no WebView or runtime
    Markdown parser.
  - Mistakes fail the build, not the app.
- **Alternatives**:
  - JSON files: hard for non-developers to edit;
  - a headless CMS: another service and login, which is overkill for about 15 lessons;
  - rendering Markdown at runtime: a heavier dependency, harder to style.

## R4 — Phrase-by-phrase lessons on the mantras, with Gauḍapāda inside

- **Decision**: lessons follow the **12 mantras**. Each lesson holds an ordered list of
  **phrases**, optionally under a part heading. Long mantras span several lessons, linked by
  `mantra` and `part`. The mantra's last part also carries:
  - **Gauḍapāda's kārikās** on it, exactly one `featured`, the rest under "More from Gauḍapāda",
    shown one at a time;
  - the full mantra, summary and prompt.

  The build checks that all 29 kārikās of chapter 1 appear exactly once.
- **Why**:
  - The Upaniṣad is the subject and the Kārikā is the guide, so kārikās sit with the mantra they
    explain rather than as lessons of their own.
  - This follows the format the product owner reads every morning: mantra 7 in 19 phrases, under
    three headings.
  - Featuring one kārikā, and showing the rest one at a time, keeps lessons to about 3–5 minutes
    without leaving any out. In the prototype, two kārikās side by side felt overwhelming.
- **Alternatives**:
  - a lesson per kārikā, which makes the Kārikā a second subject;
  - only selected kārikās, which is shorter but incomplete;
  - whole-mantra lessons, which are too dense for mantra 7.

## R5 — Pronunciation guide scheme

- **Decision**: one simple English respelling, documented in `content/README.md`:
  - syllables split by hyphens;
  - long vowels doubled (*aa*, *ee*, *oo*);
  - aspirated consonants written with *h* (*bh*, *dh*);
  - *jña* written as commonly spoken (*gya*).

  It's checked by the Sanskrit proofreader.
- **Why**: beginners can say the words without learning IAST. One scheme keeps every lesson
  consistent.
- **Alternatives**: IAST only (unreadable to most beginners); audio (planned for later, see
  ROADMAP).

## R6 — Daily reminder

- **Decision**: use `expo-notifications` local notifications:
  - schedule one-off notifications for the next 7 days at the chosen time;
  - when the person finishes or reads a lesson, cancel today's;
  - each time the app opens, top the schedule back up to 7 days.
- **Why**: a repeating daily trigger can't skip days, and local notifications can't run logic when
  they fire. A week ahead covers people who don't open the app for a while. No push server is
  needed, so nothing leaves the phone.
- **Alternatives**: a repeating trigger (can't skip a day); server push (needs a server and push
  tokens).

## R7 — Crisis check on journal replies

- **Decision**:
  - **Word and phrase lists** in `packages/shared`, matched on the phone, with negation handling and
    an idiom allow-list (e.g. "this job is killing me").
  - The lists are drafted, then reviewed with someone experienced in crisis support, and tested
    against a crisis test set.
  - The matcher leans toward showing support.
- **Why**: replies never leave the phone, so the check must run there. Word lists are transparent,
  easy to review and work offline. In v2, the same lists become Ask's first gate.
- **Alternatives**: an on-device ML classifier (adds app size, and no reliable small model was
  found for this); no check at all (unsafe, and against the constitution).

## R8 — Truly offline, nothing sent

- **Decision**:
  - no analytics, no crash-reporting SDK and no tracking in v1;
  - **no `expo-updates`** (over-the-air updates) — updates ship through the app stores;
  - crash data comes only from App Store Connect and Play Console, which users opt into at the OS
    level.
- **Why**: the spec promises the app sends nothing about the person (FR-027, SC-010). Each of these
  SDKs makes network calls. Leaving them out also lets the App Store privacy label say **"Data Not
  Collected"**.
- **Alternatives**:
  - Sentry or a similar tool (useful, but sends data and needs consent);
  - OTA updates (faster fixes, but checks a server on launch).

  Both can be reconsidered in v2, when Ask already needs a consent screen.

## R9 — Fonts offline

- **Decision**: bundle Newsreader, Karla and Noto Serif Devanagari with `@expo-google-fonts/*`.
- **Why**: fonts load from the app, so Devanagari and lessons render correctly offline.
- **Alternatives**: loading fonts from Google at runtime, which fails offline.

## R10 — Testing

- **Decision**:
  - **Vitest** for `packages/shared` and the content build (schemas, unlock rule, crisis matcher);
  - **Jest** with React Native Testing Library for screens;
  - **Maestro** for end-to-end flows on simulators;
  - the **crisis test set** runs in CI.
- **Why**: each is the standard choice for its layer, and Maestro flows are plain, readable YAML.
  The constitution's golden eval set (Principle VII) tests generated responses, so it starts with
  Ask in v2. In v1, content quality is gated by review (SC-003).
- **Alternatives**: Detox, which is heavier to set up than Maestro.

## R11 — Feedback without sending anything ourselves

- **Decision**: "Give feedback" opens the person's own email app with `Linking.openURL("mailto:…")`.
  It's pre-addressed, with the subject "Turiya feedback" and a body holding the app version, device
  type and current lesson ID. If no email app can open it, the address is shown as selectable text
  with a copy button.
- **Why**: Turiya stays free of network code (R8), and the person sees and controls exactly what's
  sent. Being optional and initiated by the person, it should keep the App Store label "Data Not
  Collected" under Apple's optional-disclosure rule (confirm at submission).
- **Alternatives**:
  - an in-app form posting to a server: needs a server and network code, which breaks the v1
    promise;
  - a third-party feedback SDK: sends data, and it's the same problem.

## R12 — Support list by country

- **Decision**: show the services for the phone's region, from `expo-localization` (no network). If
  the region isn't Australia or India, ask once and save `supportRegion`. A switch on the screen
  moves between the two lists.
- **Why**: the right numbers must appear without a network or an account. The device region is the
  best offline signal, and travellers can switch.
- **Alternatives**: the store country (not readable offline in a reliable way); asking everyone at
  first launch (adds friction for most people).
