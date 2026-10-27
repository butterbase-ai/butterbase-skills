---
name: ai
description: Use when calling the app's AI gateway from agent tools — chat completions, embeddings, listing models, configuring defaults or BYOK, reading token/cost usage, polling video jobs or preparing declared fictional image references
---

# Butterbase AI Gateway

The AI gateway provides chat, embeddings, model listing, configuration, usage reporting, and asynchronous video jobs. One umbrella tool: **`manage_ai`**.

| Action | What it does | Returns |
|---|---|---|
| `chat` | Synchronous chat completion (no streaming) | OpenAI-shaped `{ choices: [...] }` |
| `embed` | Vector embeddings for string or string[] | OpenAI-shaped `{ data: [{ embedding: [...] }] }` |
| `list_models` | Available models with capabilities | `{ models: AiModel[] }` |
| `get_config` | Current AI config (default model, BYOK key flag, etc.) | `AiConfig` |
| `update_config` | Set defaults, allowed models, max tokens, BYOK | `AiConfig` |
| `get_usage` | Token + cost aggregate over a window | usage record |
| `submit_video` | Submit an ordinary async video job | `{ job_id, status, polling_url }` |
| `poll_video` | Read an existing video job without submitting again | status, optional stage and authenticated content URLs |

---

## 1. Chat

```
manage_ai({
  action: "chat",
  app_id,
  messages: [
    { role: "system", content: "You are a helpful assistant." },
    { role: "user",   content: "What's RAG?" }
  ],
  model: "openai/gpt-4o-mini",     // optional — falls back to app's default
  temperature: 0.2,                // optional
  max_tokens: 500                  // optional
})
```

This action sets `stream: false` deliberately — agent tools don't stream. If you need partial-token deltas, drive the SDK's `ai.chatStream(…)` from inside a function or DO instead.

`messages[].content` can be a string or an array of content parts (`{ type: "text", text }`, `{ type: "image_url", image_url: {...} }`, `{ type: "video_url", video_url: {...} }`).

---

## 2. Embed

```
manage_ai({
  action: "embed",
  app_id,
  input: "hello world",            // or ["a", "b", "c"]
  model: "openai/text-embedding-3-small",   // optional
  encoding_format: "float"          // or "base64"
})
```

---

## 3. List models

```
manage_ai({ action: "list_models", app_id })
// → { models: [{ id, provider, capabilities: ["chat", "embed", ...], context_window, pricing }, ...] }
```

Use this to discover what the app can call — capabilities + context window matter when picking a model.

---

## 4. Configure

```
manage_ai({
  action: "update_config",
  app_id,
  config: {
    defaultModel: "openai/gpt-4o-mini",
    allowedModels: ["openai/gpt-4o-mini", "anthropic/claude-haiku-4-5"],
    maxTokensPerRequest: 4000,
    byokKey: "..." // optional — rotates the customer-supplied OpenRouter / Anthropic key
  }
})
```

- `maxTokensPerRequest` is server-clamped to 1–100000.
- `allowedModels` is a whitelist — empty means all models the provider exposes.
- Setting `byokKey` switches the app to route through that customer key. Clear it by passing `byokKey: ""` (returns to platform pool).

---

## 5. Usage

```
manage_ai({
  action: "get_usage",
  app_id,
  startDate: "2026-05-01",
  endDate:   "2026-05-31"
})
```

Returns aggregate token counts + cost. Useful for billing reconciliation, spending-cap diagnostics, and showing dashboards.

---

## 6. Common pitfalls

- **Trying to stream from a tool** — `manage_ai` is synchronous. Use the SDK inside a function for streamed deltas.
- **Sending `stream: true` in the body** — the tool ignores it; always wired to `false`.
- **Hardcoding `model`** — better to omit, let the app's `defaultModel` win, and surface that knob via `update_config`.
- **Skipping `list_models` before suggesting one** — model availability shifts; verify before recommending.

---

## 7. What this skill does NOT cover

- Streaming chat — use the SDK (`ai.chatStream`) inside a function or DO.
- Vector storage / retrieval — see `butterbase-skills:rag-dev` (RAG collections wrap embeddings + search together).
- AI in deployed functions — they import `@butterbase/sdk` and call `client.ai.*` directly; no MCP needed at runtime.

---

## 8. Decisions (typed classification)

For routing, classification, moderation or scoring, where code needs a choice / yes-no / score with probabilities, use `manage_ai` `action: "decide"` (SDK `ai.decide`) with a decision model (default `typesafe/jev-1.13`) instead of asking a chat model for JSON. It is cheaper, faster and returns typed probabilities. Full reference: `butterbase_docs` topic `ai`, "Decision models".

---

## 9. Video jobs and declared fictional references

Before video work, call `butterbase_docs` with `topic: "ai"` and discover supported models with `manage_ai({ action: "list_models", app_id, modality: "video" })`.

Preserve the returned job ID, show the reported stage, poll without issuing another submission, and display authenticated content URLs after completion. Use the caller's explicit fictional-character declaration only. Never silently add declarations or automatically label real people as fictional.

Inspect the connected `manage_ai` tool schema before choosing a client surface. When it includes both `reference_declarations` and `prepare_references`, add the declaration example below to `manage_ai` with `action: "submit_video"`. For older MCP servers that lack these fields, send the fields in the REST body for `POST /v1/:appId/videos/completions`; do not assume an older wrapper forwards unknown arguments. Poll the returned job ID with `poll_video` or `GET /v1/:appId/videos/completions/:jobId`. Do not invent an SDK video helper.

### Optional fictional character references

Eligible apps and models can declare specific image slots as AI-generated fictional characters:

```json
{
  "input_images": ["https://images.example/fictional-character.png"],
  "reference_declarations": [
    {"path": "/input_images/0", "kind": "ai_generated_fictional_character"}
  ],
  "prepare_references": "on_rejection"
}
```

Add these fields to an ordinary video request. Declare only images the caller knows are AI-generated fictional characters. A declaration does not establish consent for a real person's likeness or guarantee that an image will pass provider validation. Do not infer a declaration from appearance or a prompt.

`on_rejection` is the default: prepare only after a verified eligible input rejection and retry once after settlement. `upfront` prepares all declared slots before submission. Canonical paths address image entries in `input_images`, `frame_images`, or `input_references`; use one canonical collection and avoid conflicting aliases. Up to eight distinct slots can be declared. Undeclared requests use ordinary video generation.

Submission returns the same stable `job_id` while the worker advances through `preparing_references`, `submitting`, `polling`, and `settling`. Polling reads status; it does not trigger generation. Completion includes authenticated content URLs and acknowledged charges. Unsupported app/model combinations return `UNSUPPORTED_REFERENCE_PREPARATION`. Changed image bytes, unavailable or rejected assets, and capacity exhaustion may fail preparation. A timeout with unknown provider acceptance remains pending for reconciliation; do not submit another job merely because polling is slow.

Preparation sends declared images to an external generation provider for storage and validation. Registration is not separately charged in this release; video generation uses the app's normal credits and settled usage. Source URLs and prompts are removed from preparation records after 24 hours. Provider copies are queued for deletion when the last retained job reference is released, subject to active jobs and unresolved operations; provider deletion can therefore take longer. App deletion preserves cleanup obligations and unsettled billing evidence. URL inputs have no storage-object identity: deleting the original file does not immediately delete its provider copy. Use stable storage `objectId` values in your own app and resolve fresh download URLs when loading UI.

When retained-output delivery is enabled, prepared videos complete only after every output is copied into private app storage and settlement is acknowledged. Generated outputs count toward app and organization storage limits. Authenticated content endpoints support one byte range and return 410 after the stored output is deleted. Keep the stable job ID for future content loads; older jobs can still depend on provider retention. App copies exclude prepared-video history.

There is no character-library dashboard. Preparation eligibility and resource limits are deployment-controlled. For troubleshooting, inspect the returned stage and sanitized error, verify the declaration and unchanged image bytes, and contact the operator for unresolved acceptance or cleanup.

---

If a `docs/butterbase/00-state.md` exists in the working directory, prefer invoking via `/butterbase-skills:journey-ai` so the journey orchestrator stays in sync.
