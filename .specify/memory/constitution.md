<!--
Sync Impact Report
- Version change: 1.0.0 → 1.1.0
- Rationale: Added a new non-negotiable principle (Do No Harm, Carry No Negativity). MINOR bump:
  new principle added, no existing principle removed or redefined. Subsequent principles
  renumbered III–VII; internal cross-references and governance reviewer list updated to match.
- History:
  - 1.0.0 (2026-10-17): Initial ratification from the unfilled template scaffold.
  - 1.1.0 (2026-10-17): Added Principle II (Do No Harm, Carry No Negativity).
- Principles (current):
  1. Reflection, Not Prescription
  2. Do No Harm, Carry No Negativity
  3. Textual Integrity & Source Separation
  4. Safety Is a Hard Gate
  5. Calm, Faithful Experience
  6. Privacy by Default
  7. Evidence-Based Quality Gates
- Added sections: Technology & Architecture Constraints; Development Workflow & Quality Gates
- Removed sections: none
- Deferred items / TODOs: none
- Templates reviewed for alignment:
  - .specify/templates/spec-template.md ✅ (no changes needed)
  - .specify/templates/plan-template.md ✅ (no changes needed)
  - .specify/templates/tasks-template.md ✅ (no changes needed)
-->

# Turiya Constitution

Turiya is a mobile application that helps people reflect on everyday problems and questions
through the Māṇḍūkya Upanishad's four states of consciousness, primarily grounded in
Gauḍapāda's Māṇḍūkya Kārikā. It is a contemplative, reflective tool — not therapy, counseling,
or advice. This constitution defines the non-negotiable principles that govern how the product
is designed, built, and evolved.

## Core Principles

### I. Reflection, Not Prescription

The product MUST help a person locate which state of consciousness they are caught in and prompt
a witnessing (turīya) stance; it MUST NOT issue verdicts, diagnoses, or prescriptive advice.
Output MUST NOT tell a person what to do about a relationship, decision, or feeling
(e.g., "leave them", "you are right"), and MUST NOT frame itself as therapy, counseling, or
treatment. The single daily "one thing to try" practice line is a small concrete reflective
exercise, never a directive to resolve the person's problem.

**Rationale**: The witness stance is the core mechanism of the product, not a theme. It is also
the boundary that keeps Turiya out of regulated "AI health/relationship advice" territory. A
model drifting into generic self-help is the primary way this product silently stops being what
it is meant to be.

### II. Do No Harm, Carry No Negativity

At every point in the experience, the product MUST NOT sound negative, harsh, shaming, bleak, or
discouraging, and MUST NOT suggest, endorse, or imply any action that could harm the person or
others — physically, emotionally, relationally, financially, or otherwise. This applies to the
tone and content of every surface: generated responses, practice lines, fallback text, error and
empty states, and all static copy. When a situation is painful, the product MUST meet it with
warmth and steadiness rather than reinforcing distress, catastrophizing, or validating a harmful
impulse. If the only faithful response to an input would risk harm or read as negative, the
product MUST decline gracefully or redirect to a steadying reflection (and, where relevant, the
crisis path in Principle IV) rather than produce it.

**Rationale**: People bring their rawest problems here, often while already hurting. A single
negative or harm-suggesting line can do real damage and breaks the trust the whole product
depends on. Non-harm and a warm, non-negative tone are therefore absolute, not aspirational —
they outrank fidelity to any other goal when the two conflict.

### III. Textual Integrity & Source Separation

Canonical material (Sanskrit, transliteration, translation of Māṇḍūkya mantras and Gauḍapāda's
Kārikā) and applied commentary MUST be kept in separate layers in the data model so a response
can never present interpretation as scripture. Verse text and citations shown to the user MUST
be hard-coded from a verified source and MUST NEVER be model-generated. Every canonical entry
MUST carry its translation source for attribution and license tracking; translations MUST be
public-domain or properly licensed before shipping. Devanagari, transliteration, and references
MUST be proof-read against a Sanskrit source before any public release.

**Rationale**: Citations that hallucinate destroy trust in a product whose authority rests on a
1,500-year-old text. Separating original commentary (which carries the product value and no
copyright risk) from canonical text protects both integrity and legal standing.

### IV. Safety Is a Hard Gate

Crisis and self-harm handling and the no-verdict/no-advice rule MUST be enforced in code as hard
gates, not left to prompting alone. Before generation, inputs signalling crisis or self-harm MUST
skip the reflective flow entirely and surface real crisis resources. After generation, output
MUST be checked and blocked if it reads as a verdict or prescriptive advice. If the model call
fails or returns invalid output, the app MUST fall back to a built-in reflective response so the
user never hits a dead end.

**Rationale**: Prompts drift and models fail; a person in distress cannot depend on probabilistic
behavior. Enforcing safety as code (input gate + output gate + fallback) is the one place where
model judgment alone is unacceptable.

### V. Calm, Faithful Experience

The experience MUST feel quieting the moment it opens. The UI MUST faithfully honor the approved
design (typography, color tokens, spacing, copy, and interaction flow) rather than approximate
it. Interactive targets MUST be at least 44px. Motion MUST be subtle and MUST respect the
platform's reduced-motion preference. Scope is governed by the stated version boundary: features
explicitly deferred to a later version MUST NOT be built early.

**Rationale**: The calm is the product. Visual and interaction fidelity, accessible touch
targets, and respect for reduced-motion are what make the app feel like a refuge rather than
another anxious app. Scope discipline keeps v1 shippable and the experience focused.

### VI. Privacy by Default

A user's problems and reflections are private, sensitive data and MUST be treated as such. In the
current version, journal history is device-only — no accounts and no server-side storage of user
entries — persisted through the platform's secure/local store. LLM credentials and backend secrets
MUST NEVER be shipped in the client; all model calls MUST go through the project's own backend.
Any future move to server-side storage or accounts MUST be an explicit, documented decision.

**Rationale**: People bring their rawest problems here. Default privacy and server-held secrets
are both a trust commitment and a basic security requirement; silently centralizing private data
would betray the premise of the app.

### VII. Evidence-Based Quality Gates

A golden evaluation set of realistic situations across the supported themes MUST exist, each with
an expected state/theme classification, expected grounding verse(s), and a pass/fail check that
the response avoided giving a verdict. This eval set MUST be run after every prompt change or
commentary-content change, and regressions MUST block release.

**Rationale**: The qualities that matter most here — staying Socratic, grounding in the right
verse, avoiding advice — are easy to erode invisibly. A repeatable eval is the only reliable
guard against slow quality drift.

## Technology & Architecture Constraints

- **Client**: A cross-platform mobile app built with React Native, following the project's
  established component and navigation patterns.
- **Backend**: A server-side API backed by PostgreSQL. The small, high-precision knowledge base
  (canonical text + applied commentary) MAY use vector search (e.g., pgvector) combined with
  keyword matching; pure vector search MUST NOT be the sole retrieval method, since exact-term
  queries must reliably surface the correct verses.
- **LLM**: A capable chat model invoked only through the backend. Model choice favors the
  cost-effective tier; the deciding factor is reliable adherence to the reflective (non-advice)
  stance, validated against the eval set.
- **Data model**: Canonical and applied-commentary layers remain distinct, with source/license
  attribution on canonical entries, per Principle III.

## Development Workflow & Quality Gates

- Every feature MUST trace to an approved specification before implementation begins.
- Changes to prompts, safety gates, or commentary content MUST run the golden eval set
  (Principle VII) and MUST NOT be merged on a regression.
- Safety-gate logic (crisis handling, verdict/advice blocking, fallback) MUST have explicit tests
  and MUST be reviewed on every change that touches the generation pipeline.
- The applied-commentary layer SHOULD be reviewed by someone grounded in the tradition before it
  is trusted in a release.
- UI changes MUST be checked against the approved design tokens and accessibility rules in
  Principle V.
- Any deviation from these principles MUST be justified in the relevant spec/plan and recorded;
  unjustified complexity or scope expansion MUST be rejected in review.

## Governance

This constitution supersedes other development practices for the Turiya project. Amendments MUST
be proposed in writing with a rationale, reviewed, and recorded in the Sync Impact Report at the
top of this file.

Versioning follows semantic versioning:
- **MAJOR**: Backward-incompatible governance changes — removing or redefining a principle.
- **MINOR**: Adding a new principle or section, or materially expanding guidance.
- **PATCH**: Clarifications, wording, and non-semantic refinements.

All specs, plans, and pull requests MUST be reviewed for compliance with these principles.
Reviewers MUST verify, at minimum, the reflective-stance rule (I), the do-no-harm and
non-negativity rule (II), source separation and citation integrity (III), safety gates (IV), and
privacy defaults (VI). Complexity or scope beyond the current version boundary MUST be explicitly
justified or rejected.

**Version**: 1.1.0 | **Ratified**: 2026-10-17 | **Last Amended**: 2026-10-17
