# Implementation Brief: AI Web Search Grounding (Claude + Gemini)

## What this is
An opt-in toggle that lets AI-drafted content be based on a **live web search** the provider runs server-side, rather than the model's trained knowledge alone. Same API key as normal calls — this only adds one field to the request body.

**Default state: OFF.** Ungrounded calls (trained-knowledge-only) remain the default; grounding is something the user explicitly turns on.

---

## Claude API — "web search tool"

Add a `tools` array to the `/v1/messages` request:

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1000,
  "messages": [{"role": "user", "content": "your prompt"}],
  "tools": [{ "type": "web_search_20250305", "name": "web_search" }]
}
```

**Response shape changes:** with search active, `data.content` can contain multiple blocks — `text`, `tool_use`, `tool_result` — not just one text block. Parse by filtering `type === "text"` and joining, not by grabbing `content[0]`:

```javascript
const textBlocks = (data.content || [])
  .filter(b => b.type === "text")
  .map(b => b.text);
const answer = textBlocks.join("\n");
```

If calling from a browser (not a backend), you also need this header, or the request will be blocked:
```
"anthropic-dangerous-direct-browser-access": "true"
```

---

## Gemini API — "Grounding with Google Search"

Add a `tools` array to the `generateContent` request:

```json
{
  "contents": [{"parts": [{"text": "your prompt"}]}],
  "tools": [{ "google_search": {} }]
}
```

**Response parsing:** join all parts defensively, same reasoning as Claude:
```javascript
const parts = data.candidates?.[0]?.content?.parts || [];
const answer = parts.filter(p => p.text).map(p => p.text).join("\n");
```

---

## Non-negotiables when you wire this into any app

1. **Same key, no new credential.** Don't ask the user for a separate key — it's one extra parameter on the existing call.
2. **Off by default, explicit toggle.** Label it plainly — "Ground with web search" — not buried in an "advanced" menu.
3. **State the cost/behavior difference next to the toggle**, not in a help doc three clicks away:
   - Makes a real network call every time (not a one-time library load)
   - Typically costs extra on top of normal token usage — point to `docs.claude.com` / `ai.google.dev` for current pricing rather than hardcoding a number, since it changes
   - Has no effect if you also have an offline/sandbox fallback mode — say so explicitly
4. **Fix your response parser before you ship this.** If your existing code does `content[0].text` or `parts[0].text`, it will silently truncate or break once search is on and the response has multiple blocks. Audit this first.
5. **Don't claim grounding = correctness.** It reduces hallucination risk, doesn't eliminate it. Keep whatever "review before use" framing you already apply to AI drafts.

---

## Reference: what "Sandbox mode" (if your app has one) should NOT claim
If your app has an offline/local fallback mode (templates, heuristics, no network), don't call it a provider sandbox — Anthropic and Google don't offer a Stripe-style test environment with fake keys. Your offline mode is your own simulation; the real API (grounded or not) is the only way to reach the actual model.

## Source of truth
Verify current tool names, request shape, and pricing before shipping — this brief reflects the shape as of this conversation; both providers version and rename these periodically.
- Claude API docs: https://docs.claude.com/en/api/overview
- Gemini API docs: https://ai.google.dev
