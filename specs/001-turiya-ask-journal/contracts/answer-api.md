# Contract: Answer API (v1)

The only interface between the app and the server. It serves Ask; Learn and Journal never call the
server.

- **Base path**: `/v1`, over HTTPS only.
- **Auth**: none. There are no accounts, and the app holds no secrets (FR-035).
- **Rate limiting**: per IP address, held in memory only and never stored or logged with
  question text.

## `POST /v1/answer`

The app calls this only after the person has given consent (FR-036).

### Request

```json
{ "question": "My partner and I keep having the same fight and I can't let it go." }
```

| Field | Rules |
|---|---|
| `question` | String, 1–2000 characters after trimming. Anything else → `400`. |

### Response `200` — one of two shapes, told apart by `kind`

**Reflection**
```json
{
  "kind": "reflection",
  "levels": [
    { "heading": "What actually happened", "body": "…" },
    { "heading": "The story on top", "body": "…" },
    { "heading": "What was never touched", "body": "…" },
    { "heading": "Who notices all of it", "body": "…" }
  ],
  "practice": "Once today, say the bare fact out loud, then the story, and notice the gap."
}
```

| Field | Rules (enforced by the shared Zod schema) |
|---|---|
| `levels` | Exactly 4, in the order jāgrat, svapna, suṣupti, turīya (FR-013) |
| `levels[].heading` | 1–8 words |
| `levels[].body` | 45–70 words, about the person's own situation (FR-014) |
| `practice` | 1–25 words; a small reflective action, not a directive (FR-015) |

**Crisis** — no reflection was written (FR-028)
```json
{ "kind": "crisis" }
```

- The reply carries **only generated text**. Verses, labels, order and colours come from the app's
  bundled content, so the model can't change them (FR-014).
- Clients **must ignore unknown fields**, so later additions (e.g. v1.1's `story`) don't break
  older apps.

### Errors

| Status | When | What the app does |
|---|---|---|
| `400` | Invalid request | Shows the built-in reflection (FR-033) |
| `429` | Rate limited | Shows the built-in reflection |
| `500` / `502` / `503` | Model failure, invalid output or a failed safety check | Shows the built-in reflection |
| Timeout (app side, 25 s) | No reply in time | Shows the built-in reflection |

The error body is `{ "error": "<short code>" }` and never echoes the question.

### What happens inside (for reference; see ARCHITECTURE)
1. Word-list check, then the model crisis check. If either flags → `crisis`.
2. Hybrid search over the Kārikā passages.
3. Generate the reflection with the schema.
4. Output safety check. If it fails → `502`.

### Privacy guarantees
- The question is used for this one request only. It is never written to a database, file or log.
- Logs record status, timing and model ID only.
- Only Anthropic receives the question. Search embeddings are computed on our own server
  (research R2).

## `GET /v1/health`

`200 { "status": "ok" }` when the server can reach the database. Used by the host for liveness
checks.
