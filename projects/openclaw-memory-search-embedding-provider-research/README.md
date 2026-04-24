# OpenClaw `memory_search` Embedding Provider Research

**Date:** 2026-04-24  
**Context:** Jones asked why OpenClaw `memory_search` used Gemini embeddings while the main runtime/model was OpenAI Codex, and whether requiring an OpenAI API key for OpenAI embeddings is normal.

## Executive summary

OpenClaw's `memory_search` is a separate retrieval/embedding subsystem from the chat/coding model runtime. Using `openai-codex/gpt-5.5` for agent replies does **not** automatically mean OpenAI embeddings are available.

The current behavior is consistent with OpenClaw's documented design:

- `memory_search` uses hybrid retrieval: vector embeddings + BM25 keyword search.
- Embedding provider selection is auto-detected when not explicitly configured.
- Auto-detection chooses the first available provider in a documented order.
- OpenAI embeddings require an OpenAI API key (`OPENAI_API_KEY` or `models.providers.openai.apiKey`).
- OpenAI Codex OAuth is explicitly documented as covering chat/completions only, not embedding requests.
- If Google/Gemini credentials are available and OpenAI API-key auth is not, OpenClaw may legitimately select Gemini for memory embeddings.

For this machine, that means the observed setup is not obviously broken: Codex handles the main agent model, while Gemini handles memory embeddings because it is the first available remote embedding provider after unavailable higher-priority options.

The downside is operational: Gemini free-tier embedding quota can temporarily break semantic memory search with `429` quota errors. When that happens, memory files are still intact, and OpenClaw can degrade to lexical/file reads, but semantic recall quality and convenience drop.

## Why this surprised us

The confusing part is the word "OpenAI" being used for two different auth surfaces:

1. **OpenAI Codex OAuth**
   - Used by OpenClaw's Codex runtime / coding agent path.
   - Current main model observed locally: `openai-codex/gpt-5.5`.
   - Does not imply access to the OpenAI embeddings API.

2. **OpenAI API key**
   - Used by OpenAI's platform API endpoints such as `/v1/embeddings`.
   - Required by OpenClaw's `openai` memory embedding provider.
   - Typical env/config: `OPENAI_API_KEY` or `models.providers.openai.apiKey`.

So the phrase "we use OpenAI now" is true for chat/coding, but not sufficient for memory embeddings.

## Official documentation findings

### Memory overview

OpenClaw memory is file-first:

- `MEMORY.md` for long-term memory.
- `memory/YYYY-MM-DD.md` for daily notes.
- Optional `DREAMS.md` for dreaming/consolidation review.

The docs describe `memory_search` as a semantic search tool over those memory files and note that it works automatically once an embedding API key for a supported provider exists.

Source: <https://docs.openclaw.ai/concepts/memory>

### Memory search docs

The memory search page states:

- `memory_search` indexes notes into chunks.
- Retrieval uses embeddings, keywords, or both.
- If a GitHub Copilot subscription, OpenAI, Gemini, Voyage, or Mistral API key is configured, memory search works automatically.
- A provider can be set explicitly under `agents.defaults.memorySearch.provider`.
- Local embeddings are supported with `provider: "local"` and no API key, but require `node-llama-cpp`.

Source: <https://docs.openclaw.ai/concepts/memory-search>

### Memory config reference

The config reference is the clearest source. It says all memory search settings live under:

```json5
agents.defaults.memorySearch
```

Provider selection fields:

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "openai", // or gemini, local, github-copilot, etc.
        model: "text-embedding-3-small",
        fallback: "none"
      }
    }
  }
}
```

Documented auto-detection order:

1. `local` — only if `memorySearch.local.modelPath` is configured and the file exists.
2. `github-copilot` — if a GitHub Copilot token can be resolved.
3. `openai` — if an OpenAI key can be resolved.
4. `gemini` — if a Gemini key can be resolved.
5. `voyage` — if a Voyage key can be resolved.
6. `mistral` — if a Mistral key can be resolved.
7. `bedrock` — if AWS SDK credentials resolve.

The same page explicitly says:

> Codex OAuth covers chat/completions only and does not satisfy embedding requests.

It also lists OpenAI API-key resolution as:

- Env var: `OPENAI_API_KEY`
- Config key: `models.providers.openai.apiKey`

Source: <https://docs.openclaw.ai/reference/memory-config>

## Local implementation findings

Checked installed OpenClaw 2026.4.22 files under `/opt/homebrew/lib/node_modules/openclaw/dist`.

Relevant observations:

- The bundled OpenAI plugin registers `openAiMemoryEmbeddingProviderAdapter`.
- OpenAI memory embedding adapter:
  - id: `openai`
  - auth provider: `openai`
  - default model: `text-embedding-3-small`
  - auto-select priority: 20
- Gemini memory embedding adapter:
  - id: `gemini`
  - auth provider: `google`
  - default model: `gemini-embedding-001`
  - auto-select priority: 30
- OpenAI embedding provider calls an OpenAI-compatible `/embeddings` endpoint and resolves bearer auth through provider API key/profile/env resolution.
- Local config schema confirms `agents.defaults.memorySearch.provider` supports `openai`, `gemini`, `voyage`, `mistral`, `bedrock`, `lmstudio`, `ollama`, and `local`.

Current local status before any config change:

```json
{
  "provider": "gemini",
  "model": "gemini-embedding-001",
  "requestedProvider": "auto",
  "sources": ["memory"],
  "fts": { "enabled": true, "available": true },
  "vector": { "enabled": true, "available": true }
}
```

Current auth/config situation observed locally:

- OpenAI Codex OAuth profile exists.
- Google/Gemini auth profile exists.
- No `OPENAI_API_KEY` env var was present in the agent shell.
- No `models.providers.openai.apiKey` was configured.
- Therefore auto-detection selecting Gemini is expected.

## What the community / issue tracker shows

There does not appear to be a large public discussion specifically saying "OpenClaw should let Codex OAuth pay for OpenAI embeddings." Instead, public issues and docs mostly treat memory embeddings as their own provider layer.

A few relevant patterns from GitHub issue searches:

### 1. Users often use Gemini as the practical workaround when local embeddings fail

Example issue: node-llama-cpp missing on Apple Silicon / Node.js 22. The workaround suggested in the issue was to patch config to use Gemini:

```bash
clawdbot gateway config.patch '.agents.defaults.memorySearch.provider = "gemini"'
```

Source: <https://github.com/openclaw/openclaw/issues/29548>

Interpretation: Gemini memory embeddings are not unusual; they have been a normal workaround/provider path for users whose local embeddings were unavailable.

### 2. Local embeddings have historically had install/runtime sharp edges

Examples:

- `node-llama-cpp` missing after upgrade: <https://github.com/openclaw/openclaw/issues/46569>
- local GGUF embeddings deadlocking under concurrent calls: <https://github.com/openclaw/openclaw/issues/7548>

Interpretation: The local/no-API-key path is attractive, but historically not as frictionless as remote providers. It may be better now, but should be tested deliberately before switching a working setup.

### 3. Gemini embedding path has had its own operational issues

Examples:

- Gemini behind HTTP(S) proxy failed in one version: <https://github.com/openclaw/openclaw/issues/54279>
- Older Gemini model/batch config confusion: <https://github.com/openclaw/openclaw/issues/7319>

Interpretation: Gemini is a valid provider, but it can hit provider-specific issues, including quota, proxy, model naming, and batch behavior.

### 4. QMD backend has had integration complexity

Examples:

- QMD backend CLI hangs on Raspberry Pi / arm64: <https://github.com/openclaw/openclaw/issues/65553>
- QMD returned empty results despite direct `qmd search` working: <https://github.com/openclaw/openclaw/issues/58055>
- QMD collection name mismatch issues: <https://github.com/openclaw/openclaw/issues/21349>, <https://github.com/openclaw/openclaw/issues/9854>, <https://github.com/openclaw/openclaw/issues/23228>
- QMD MCP daemon proposal to avoid cold-start timeout/fallback problems: <https://github.com/openclaw/openclaw/issues/15562>

Interpretation: Advanced local-first memory stacks exist, but are not the simplest thing to change casually. For Jones/Jarvis continuity, a stable provider matters more than architectural purity.

### 5. Public discussion volume is limited

Searches across OpenClaw repo issues and GitHub discussion search found no strong public consensus thread about "Codex OAuth should provide embedding access." The clearest guidance is currently the official memory config reference.

## Is requiring an API key normal?

For OpenAI embeddings specifically: yes, in OpenClaw's documented design.

This is not because OpenClaw wants a duplicate credential for no reason; it is because the `openai` embedding provider uses OpenAI's platform embedding endpoint. That endpoint is normally billed and authenticated via OpenAI API keys.

However, it is fair to say the UX is surprising for users who came in through Codex OAuth. The ideal product behavior might be clearer onboarding/status messaging, such as:

- "Main model auth: OpenAI Codex OAuth"
- "Memory embedding auth: Gemini API key"
- "Codex OAuth does not cover embeddings"
- "To use OpenAI embeddings, configure OPENAI_API_KEY or choose local/Copilot/Gemini"

## Practical options for Jones/Jarvis

### Option A — Keep Gemini for now

Recommended if quota errors are rare.

Pros:

- Minimal change.
- Current architecture has worked since early Jarvis history.
- No new OpenAI API billing/config needed.
- Avoids destabilizing memory search while other work is ongoing.

Cons:

- Free-tier quota can occasionally 429.
- Provider mismatch remains conceptually confusing.
- If quota becomes frequent, continuity suffers.

Operational rule:

- If `memory_search` fails with Gemini quota, fall back to direct file reads (`memory_get`, `read MEMORY.md`, today's/yesterday's notes) and retry later.

### Option B — Configure OpenAI API key for memory embeddings

Recommended if Gemini quota becomes frequent and Jones is okay with API billing.

Pros:

- Simple and well-documented.
- OpenAI `text-embedding-3-small` is cheap and reliable for text memory.
- Avoids Google/Gemini free-tier quota.

Cons:

- Requires OpenAI platform API key separate from Codex OAuth.
- Adds another paid credential to manage.
- Changing provider/model may trigger reindexing.

Potential config:

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "openai",
        model: "text-embedding-3-small",
        fallback: "none"
      }
    }
  }
}
```

Auth options:

- `OPENAI_API_KEY` environment variable, or
- `models.providers.openai.apiKey`, preferably via a secret reference if supported in the active config path.

### Option C — Test local embeddings

Recommended if Jones wants no external embedding API dependency.

Pros:

- No external quota.
- Better privacy; memory chunks do not leave the machine for embedding.
- Fits self-hosted/local-agent philosophy.

Cons:

- Requires `node-llama-cpp`/GGUF path to work reliably.
- May need model download/build fixes.
- Historically had sharp edges in OpenClaw issue tracker.

Potential config:

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "local"
      }
    }
  }
}
```

Validation commands:

```bash
openclaw memory status --deep --agent main
openclaw memory index --force --agent main --verbose
openclaw memory search "test recall" --agent main
```

### Option D — GitHub Copilot embeddings

Docs say GitHub Copilot can be auto-detected and does not need a separate API key if a Copilot token can be resolved.

Pros:

- May avoid OpenAI platform key.
- Comes before OpenAI/Gemini in auto-detection.

Cons:

- Unknown fit for current Jarvis machine without testing.
- Ties memory recall to GitHub/Copilot token availability.

## Recommendation

For now: **keep Gemini**, because it has been working and the observed quota issue was a transient free-tier limit, not data corruption or architectural failure.

Do not rush to add an OpenAI API key just to make provider names line up.

Suggested follow-up plan:

1. Leave current config unchanged.
2. Add an operational note: if Gemini quota causes `memory_search` failure, use direct memory file reads as fallback.
3. Track whether quota failures recur over the next several days.
4. If recurrence is annoying, run a controlled local embedding test first.
5. If local embedding is unreliable or too slow, then configure OpenAI API key for `text-embedding-3-small`.

## Open questions for later

- Does the current OpenClaw install have a working `node-llama-cpp` local embedding path on this Mac Mini?
- Would GitHub Copilot embedding auth be available via existing `gh`/Copilot credentials?
- How often does Gemini embedding free-tier quota actually reset/hit under Jarvis's normal memory usage?
- Should OpenClaw status/control UI surface "chat provider" and "memory embedding provider" side by side to avoid this confusion?
- Would `cache.enabled` reduce repeated embedding work enough to avoid quota spikes?

## Sources

Official docs:

- OpenClaw Memory Overview: <https://docs.openclaw.ai/concepts/memory>
- OpenClaw Memory Search: <https://docs.openclaw.ai/concepts/memory-search>
- OpenClaw Memory Config Reference: <https://docs.openclaw.ai/reference/memory-config>
- OpenClaw Memory CLI: <https://docs.openclaw.ai/cli/memory>

Issue tracker examples:

- Local/node-llama install workaround using Gemini: <https://github.com/openclaw/openclaw/issues/29548>
- node-llama-cpp missing after upgrade: <https://github.com/openclaw/openclaw/issues/46569>
- Local GGUF embedding deadlock: <https://github.com/openclaw/openclaw/issues/7548>
- Gemini proxy issue: <https://github.com/openclaw/openclaw/issues/54279>
- Gemini model/batch issue: <https://github.com/openclaw/openclaw/issues/7319>
- QMD backend hang: <https://github.com/openclaw/openclaw/issues/65553>
- QMD empty results: <https://github.com/openclaw/openclaw/issues/58055>
- QMD collection mismatch: <https://github.com/openclaw/openclaw/issues/21349>
- QMD collection mismatch / empty results: <https://github.com/openclaw/openclaw/issues/9854>
- QMD suffix bug: <https://github.com/openclaw/openclaw/issues/23228>
- QMD MCP daemon proposal: <https://github.com/openclaw/openclaw/issues/15562>

Local evidence:

- `openclaw memory status --json` on Jarvis Mac Mini, 2026-04-24.
- OpenClaw installed dist inspection under `/opt/homebrew/lib/node_modules/openclaw/dist`.
- Config schema lookup for `agents.defaults.memorySearch`.
