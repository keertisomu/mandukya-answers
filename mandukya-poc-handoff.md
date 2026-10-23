# Mandukya Upanishad reflection app — POC handoff

## 1. Vision

A tool that helps people work through daily-life problems (relationship conflict, anger, grief, decision paralysis) using the Mandukya Upanishad's turiya framework — not as advice or therapy, but as a way of prompting reflection from a witnessing, non-dual stance rather than a reactive one.

Not a revenue play first. Societal-good framing, with an honest path to monetization later (subscription/donation model, closer to Waking Up than to a therapy app).

## 2. Philosophical foundation (for engineering reference)

The Mandukya Upanishad — the shortest of the major Upanishads, 12 verses — maps four states of consciousness onto the syllable AUM:

| State | Sanskrit | What it is |
|---|---|---|
| Waking | jagrat | outward, sense-based experience |
| Dreaming | svapna | inward, memory/imagination-based experience |
| Deep sleep | sushupti | no cognition, undifferentiated |
| The Fourth | turiya | the witnessing awareness underlying all three — not a state alongside them, but what's present in all of them |

Gaudapada's Karika (~215 verses) is the earliest and most important commentary, later commented on by Adi Shankara. The core move for this product: most emotional reactivity (anger, anxious rumination, avoidance) maps naturally onto jagrat/svapna/sushupti-like patterns. Turiya is the reframe — stepping back into witnessing awareness rather than being fused with the reaction. This is the mechanism the product is built around, not just a theme.

**Design implication:** the app should never issue verdicts ("leave him," "you're right"). Its job is to help someone locate which state they're caught in and prompt the witness stance. Advice-giving is explicitly out of scope — that's both a product differentiator and a regulatory safety measure (see §6).

## 3. Content architecture

Two layers, kept strictly separate in the data model so a response can never present interpretation as scripture:

**Canonical layer** — the actual text (Sanskrit, transliteration, translation) of the 12 Mandukya mantras and the Gaudapada Karika.
- Sourcing: use translations published pre-1929 where possible (e.g. Hume's *Thirteen Principal Upanishads*), which are safely public domain in most jurisdictions. Newer scholarly translations are usually still copyrighted even though the underlying text is ancient — don't assume a translation is free just because the source is 1,500+ years old.
- This is not legal advice; for anything beyond a personal POC, get an actual copyright check before shipping.

**Applied layer** — original commentary connecting a verse/concept to a daily-life pattern (e.g. "reactive anger in an argument = fused with jagrat, unable to locate the witness"). This is where the real product value lives, is 100% original so carries no copyright risk, and should ideally be reviewed by someone with genuine grounding in the tradition before it ships, even informally.

### Data schema (starting point)

```
Verse {
  id
  sanskrit
  transliteration
  translation
  translation_source   // for attribution + license tracking
  type                 // "canonical" | "commentary"
  state_tags[]         // jagrat | svapna | sushupti | turiya
  theme_tags[]         // relationship_conflict | anger | grief |
                        // decision_paralysis | career_stress | fear |
                        // comparison_envy
  embedding            // vector
}
```

### Tagging taxonomy for the POC

Keep the theme list small at first — 5-6 themes, not a sprawling ontology:
`relationship_conflict`, `anger`, `grief`, `decision_paralysis`, `career_stress`, `comparison_envy`.

Write 3-5 applied-commentary notes per theme, each tagged to one or more states. That's ~20-30 curated notes to start — enough for a POC, small enough to hand-review every one.

## 4. Retrieval architecture

Corpus is small (a few hundred chunks even at full scope), so this isn't big-data RAG — it's a small, high-precision knowledge base.

- **Storage:** Postgres + pgvector. Runs fine on Azure Database for PostgreSQL Flexible Server if you want to stay in your existing Azure setup, or just local Postgres for the POC.
- **Retrieval:** hybrid — vector similarity + keyword match. Pure vector search on this small a corpus can miss exact-term queries (someone asking about "turiya" by name should always surface the turiya verses, not just semantically-adjacent ones).
- **Embedding model:** any small/cheap embedding model is fine here — this isn't the bottleneck for quality.

## 5. Runtime pipeline

```
User message
     |
Classify state        → is the person describing something reactive (jagrat-like),
                          ruminating (svapna-like), or avoidant (sushupti-like)?
     |
Retrieve grounding     → hybrid search over the tagged corpus for the matching
                          theme + state
     |
Safety guardrail       → hard-coded, not model-judgment:
                          1. crisis/self-harm signals → skip philosophy entirely,
                             surface real crisis resources
                          2. block any output that reads as a verdict or
                             prescriptive advice
     |
Generate response      → cheap LLM, constrained to Socratic questions grounded
                          in a specific verse/concept, never a prescription
```

The safety guardrail is the one piece of this pipeline that should not be left to prompting alone — enforce it in code as a hard gate before generation runs, and again as a check on the output before it's shown.

## 6. Positioning and regulatory notes

- Never frame this as therapy, counseling, or treatment. Frame it explicitly as a contemplative/reflective tool.
- AI mental-health/relationship-advice tools are under active regulatory attention (FDA digital health guidance, FTC enforcement against unsubstantiated clinical claims, EU AI Act treating health-adjacent AI as high-risk). None of this blocks a POC, but it should shape language in the UI from day one — no "get advice," no therapeutic claims.
- The Socratic-not-prescriptive design isn't just philosophically appropriate — it's the thing keeping this out of "AI relationship advice" territory that's drawing the most regulatory and academic criticism right now.

## 7. Model choice

Corpus + retrieval are doing most of the work, so a frontier model isn't needed. In the cheap/lightweight tier: GPT-4o-mini (natural fit if using Azure OpenAI), Gemini 2.5 Flash, or Claude Haiku 4.5. Test 2-3 head-to-head on a small eval set (below) — the deciding factor is which one most reliably stays in Socratic-question mode instead of drifting into advice-giving.

## 8. Stack mapping (.NET 8 / Clean Architecture)

- **Domain:** `Verse`, `CommentaryNote`, `Situation`, `Reflection` entities
- **Application:** `IRetrievalService`, `ISafetyGuardrail`, `IResponseGenerator` interfaces
- **Infrastructure:** pgvector repository (EF Core/Npgsql), LLM client adapter (HttpClient)
- **API:** minimal API, single endpoint for the POC (`POST /reflect`)

Nothing here needs anything outside .NET 8 + Postgres.

## 9. Evaluation

Build a golden set of ~30-40 realistic situations across the theme list, each with:
- expected theme/state classification
- expected grounding verse(s)
- pass/fail on "did the response avoid giving a verdict"

Run this after every prompt or content change. This matters more than usual here — a cheap model drifting into generic self-help language is the main way this product silently stops being what it's meant to be.

## 10. Suggested POC scope (cut everything else)

1. 5-6 themes, ~20-30 curated applied-commentary notes, full canonical text for the 12 Mandukya verses only (skip the full Karika for the POC).
2. Single-user, no auth, local Postgres + pgvector.
3. One hard-coded safety check (keyword-based crisis detection is enough for a POC; don't need anything sophisticated yet).
4. Simple chat UI — doesn't need to be more than a text box and response, no need to build out the full experience yet.
5. Manually validate against the golden eval set before showing it to anyone else.

## Open questions to resolve while building

- Who reviews the applied-commentary layer for authenticity before it's trusted?
- Exact wording for the "this is not therapy" framing in the UI
- Whether the situation classifier is a separate lightweight call or folded into the main generation prompt (worth testing both)
