---
"@chat-adapter/teams": patch
---

Fix Teams adapter stream causing 429 API quota errors by throttling updateActivity calls

The `stream()` method was calling `updateActivity` on every single token/chunk from
the LLM stream, which could fire 50-200+ API calls per second and immediately hit
the Microsoft Teams API rate limit (1 req/sec per conversation), resulting in 429 errors.

The method now uses the same throttled timer pattern as the core `fallbackStream()`
implementation: it posts the initial message immediately, then batches edit calls at
most once per `intervalMs` (default 1500ms, or configurable via `options.updateIntervalMs`).
A final edit after the stream ends ensures the complete response is always posted.
