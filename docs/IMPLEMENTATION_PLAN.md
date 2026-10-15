# Implementation plan — v1

A milestone-based plan. Each milestone is independently demoable. References:
[REQUIREMENTS.md](./REQUIREMENTS.md), [ARCHITECTURE.md](./ARCHITECTURE.md),
[DECISIONS.md](./DECISIONS.md).

Order is chosen so the client is usable end-to-end **early** via the built-in
fallback (FR-13), before the backend/RAG exist — de-risking the product feel
first, the AI quality second.

---

## M0 — Project setup
- Monorepo scaffold: `apps/mobile`, `apps/api`, `packages/shared`, `db/`.
- TypeScript, lint/format, and a SessionStart-friendly test setup.
- `packages/shared`: the state→verse constants (§2.3) and the Zod **answer
  schema** (`{ levels[4]{heading,body}, practice }`), imported by both app and api.
- **Exit:** `pnpm install` + typecheck + lint pass; shared schema importable.

## M1 — Client shell & design system (NFR-6)
- Expo app boots to a calm **Ask** screen.
- Encode design tokens (colors, Newsreader/Karla/Noto Serif Devanagari, radii,
  type scale, input-card shadow) and base components.
- Bottom tab nav (Ask / Journal); reduce-motion plumbing (NFR-5).
- **Exit:** both tabs navigable; Ask matches the handoff visually on a device.

## M2 — Ask + Answer + Journal with the built-in fallback (FR-1–FR-20)
- Build Ask (greeting, input, "Look at it", chips, four-states primer),
  Thinking (breathing halo + cycling status), Answer (four accordion cards with
  hard-coded verses + practice card), Journal (list, empty state, accordions,
  timestamps).
- Wire the full flow using **only** the ported `fallback()` — no network yet.
- On-device persistence via MMKV (DEC-3); frozen entry shape; try/catch around IO.
- **Exit:** a user can ask, get a (fallback) four-state answer, see it saved, and
  reopen it from Journal across restarts. Acceptance criteria 1, 3, 4, 6, 7 pass.

## M3 — Backend skeleton + `/v1/answer` (ungrounded) (FR-21–FR-24, NFR-3/7/9)
- Thin TS service (Vercel AI SDK + `@ai-sdk/anthropic`), key in env only.
- `POST /v1/answer` → `generateObject` with the system prompt and the shared Zod
  schema (no RAG yet). `GET /v1/health`.
- Point the client at it; keep fallback on any failure.
- **Exit:** real four-state answers for typed problems; invalid/failed responses
  still fall back cleanly. Acceptance criterion 2 passes.

## M4 — Postgres + pgvector + corpus seeding (DEC-5, CR-1/CR-2)
- Provision Postgres with pgvector; migrations for `karika_passage`,
  `state_verse`, `daily_counter`.
- **Content work (can run in parallel from M0):** source + license the
  translation/commentary (CR-2); proofread Devanagari + refs (CR-1).
- Seed script: load corpus → chunk → embed → insert; populate `state_verse` from
  the proofread map.
- **Exit:** corpus queryable; nearest-neighbour retrieval returns sensible
  passages for sample questions.

## M5 — RAG grounding in `/v1/answer` (DEC-7, FR-22)
- Pipeline: embed question → retrieve top-k → pass passages as grounding context
  → `generateObject`.
- Light eval pass: run ~10–15 representative problems, read answers for fidelity
  to the commentary and tone; tune prompt/k.
- **Exit:** answers visibly reflect Gauḍapāda's commentary; schema always honored;
  acceptance criteria 2 and 7 hold with real content.

## M6 — Privacy hardening & polish (NFR-1/2/4/5)
- Verify no question/answer text in logs or DB (acceptance criterion 5).
- Privacy copy in-app: device-only storage + no-training provider statement
  (DEC-6).
- Offline behavior: Journal fully readable offline; submit without network →
  fallback.
- Accessibility pass: reduce-motion, 44px targets, contrast.
- **Exit:** all acceptance criteria (1–7) pass.

## M7 — Release prep
- App icons/splash, store metadata, privacy nutrition labels reflecting DEC-1/6.
- Backend deploy + health checks; secrets configured; basic non-PII request
  logging/metrics.
- Build and submit iOS + Android.
- **Exit:** builds submitted; backend live.

---

## Critical path & parallelism
- **Content (CR-1/CR-2)** is the long-lead, non-code item — start it at M0; it
  gates M4/M5, not the client.
- M1–M2 (client + fallback) can proceed fully in parallel with backend work.
- M3 unblocks "real answers"; M4→M5 add fidelity.

## Risks & mitigations
- **Translation licensing** (CR-2) → start early; have a public-domain fallback
  text identified.
- **Sanskrit accuracy** (CR-1) → proofread with a qualified reader before launch;
  verses are hard-coded so errors are correctable in one place.
- **Model output drift** → schema validation + client fallback make bad output
  non-fatal; the light eval in M5 catches tone/fidelity regressions.
- **Framework churn** → staying on the AI SDK (not Mastra) in v1 keeps the surface
  small; Mastra remains an additive v2 option (DEC-4).

## Explicitly out of scope for this plan
Everything under Non-goals in REQUIREMENTS.md (Śaṅkara, verse library, practice,
streaks, community, accounts, sync, push, analytics dashboards).
