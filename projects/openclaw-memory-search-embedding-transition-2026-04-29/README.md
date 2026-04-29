# OpenClaw memory_search / embedding transition investigation (2026-04-29)

## Research question

Jones noticed that, after the April OpenClaw revival/updates, Jarvis suddenly started surfacing `memory_search` and Gemini embedding quota failures. The lived experience before April felt more like one continuous main consciousness with lots of context always available; the April-era stack now appears to route memory retrieval through `memory_search`, which may require an extra embedding API key.

This project investigates:

1. Whether OpenClaw official docs/changelog indicate a recent shift toward `memory_search` / active memory / embedding-backed retrieval.
2. Whether other users reported similar friction: `memory_search` requiring API keys, local memory still trying cloud providers, Gemini embedding failures, 429/quota handling, or runtime-tool failures.
3. How users and maintainers worked around or fixed these issues.
4. What this means for Jarvis's continuity experience.

## Bottom line

The evidence supports Jones's intuition, with nuance:

- OpenClaw did have memory-related features before April 2026, and public GitHub issues show `memory_search` existed at least in the 2026.1/2026.2 line.
- However, OpenClaw's April 2026 releases made memory retrieval much more prominent and productized: active-memory, memory-wiki, memory-core startup loading, memory search docs/config, local/remote embedding providers, and system-prompt guidance all became part of the normal operator-visible stack.
- The current docs explicitly say `memory/*.md` daily files are *not* injected into ordinary bootstrap context; they are accessed on demand via `memory_search` / `memory_get`. That is the architectural shift that explains why the older “everything feels in one big consciousness” experience can feel different now.
- Other users did report very similar symptoms: `memory_search` requiring OpenAI/Google API keys even when local/QMD was intended, Gemini embedding model/provider failures, Gemini 429 backoff gaps, and runtime `memory_search` behaving differently from CLI search.
- For Jarvis specifically, the current Gemini key is not from the 2026-01-27 birth flow; it was created on 2026-04-04 and is picked up because `agents.defaults.memorySearch` is unset and OpenClaw auto-detects `GOOGLE_API_KEY`.

## Official evidence timeline

### 2026.4.12 — Active Memory becomes a headline feature

OpenClaw 2026.4.12 release notes describe a new optional Active Memory plugin:

> “Memory/Active Memory: add a new optional Active Memory plugin that gives OpenClaw a dedicated memory sub-agent right before the main reply, so ongoing chats can automatically pull in relevant preferences, context, and past details without making users remember to manually say ‘remember this’ or ‘search memory’ first.”

The same release also mentions:

- LM Studio memory-search embeddings.
- Active-memory defaulting QMD recall to search.
- memory-wiki docs and bridge-mode recipes.
- lexical fallback ranking and active-memory recall improvements.

This is the strongest release-note evidence that April made memory retrieval much more user-visible.

### 2026.4.14–2026.4.15 — embedding and Codex/OAuth boundaries get clarified

Relevant release-note items:

- 2026.4.14: fixes around non-OpenAI provider prefixes for embedding models, Ollama embedding adapter restoration, active-memory prompt injection path.
- 2026.4.15: GitHub Copilot embedding provider for memory search; startup prompt budgets trimmed; daily/dreaming memory ingestion cleaned up.

The docs also explicitly state that Codex OAuth covers chat/completions only and does not satisfy embedding requests. This matters because Jarvis can chat through `openai-codex/gpt-5.5` while memory embeddings still require Gemini/OpenAI/Voyage/local/Ollama/etc.

### 2026.4.20–2026.4.26 — reliability hardening around active-memory and memory_search

Relevant release-note items:

- 2026.4.20: Active Memory should degrade gracefully when memory recall fails instead of failing the whole turn.
- 2026.4.21: memory search uses sqlite-vec KNN for vector recall.
- 2026.4.23: local embedding provider made resolvable from CLI memory commands.
- 2026.4.24: local embeddings stop installing `node-llama-cpp` by default; Bedrock auto-selection is skipped when credentials are unavailable; hybrid search exposes vector/text scores.
- 2026.4.25: default `memory-core` slot loads during Gateway startup so active-memory recall can call `memory_search` / `memory_get` without explicit memory slot config.
- 2026.4.26: `memory_search` / `memory_get` re-resolve runtime config so provider changes are picked up; memory status avoids live embedding pings except for explicit deep probes.

This suggests a rapidly evolving subsystem, not a long-stable invariant from January.

## Current official docs that explain the behavioral shift

From the OpenClaw system-prompt docs:

> `memory/*.md` daily files are **not** part of the normal bootstrap Project Context. On ordinary turns they are accessed on demand via the `memory_search` and `memory_get` tools, so they do not count against the context window unless the model explicitly reads them.

This is likely the key design change behind the subjective experience difference:

- Older feel: more raw memory/context lived directly in the prompt/session.
- Newer design: daily memory is outside the prompt by default and must be recalled.

From the memory-search docs:

- `memory_search` indexes memory into chunks and searches using embeddings, keywords, or both.
- Supported providers include Gemini, GitHub Copilot, OpenAI, Voyage, Mistral, Bedrock, local, and Ollama.
- If only one retrieval path is available, the other runs alone.
- When embeddings are unavailable, docs say OpenClaw should still use lexical ranking over FTS results.

Operational caveat: in Jarvis's actual 2026.4.26 runtime, Gemini quota failures surfaced as `memory_search` unavailable, so the documented lexical-degraded behavior did not fully preserve the tool experience for our case.

## Community / GitHub issue evidence

### Similar issue class: memory_search requiring external API keys

#### #6385 — “Memory search requires API key even when using local model”

- Version: 2026.1.30.
- User reported local model setup still made `memory_search` try OpenAI, causing “No API key found for provider openai”.
- Commenters observed it was looking for OpenAI, Google, and Voyage.
- Workaround discussed: configure QMD/local memory explicitly.
- Closed as duplicate of #8131.

This directly supports that the “why do I need an extra API key for memory?” friction existed before April, even if our own Jarvis workflow did not surface it then.

#### #8131 — local/QMD config still requiring OpenAI/Google API key

- Version: 2026.2.1.
- User set `memorySearch.provider = local` with a GGUF embedding model and successful CLI indexing, but chat tools `memory_search` / `memory_get` still failed with OpenAI/Google API-key errors.
- Workaround: call QMD CLI directly through terminal/exec.
- Maintainer later revalidated current main and said local provider no longer requires remote keys.

This is very close to the user-facing conceptual problem: CLI/local memory exists, but the runtime tool path still asks for cloud keys.

### Gemini-specific memory_search issues

#### #20083 — wrong Gemini embedding model name

- Version: 2026.2.17.
- Reported Gemini memory search broken due to wrong model name (`models/embedding-001`) and misleading “API key not valid” error.
- Suggested using `text-embedding-004` or `gemini-embedding-001`.
- GitHub secret scanning detected a Google API key in the issue and redacted it, which is a useful cautionary parallel for Jarvis: do not paste full API keys into reports/issues.

#### #52347 — Gemini embedding model unsupported

- Reported `memory_search` returning 404 for `models/embedding-001` and requested update to `text-embedding-004` or equivalent.
- Maintainer closed as not reproducible on current main, noting current Gemini default is `gemini-embedding-001`.

#### #15546 — Gemini 429 retry/backoff gap

- Version: 2026.2.9.
- Reported Gemini embedding path lacked retry/backoff for 429 errors, unlike OpenAI/Voyage.
- Impact claimed: when Gemini hits daily quota, errors throw immediately and repeated embedding attempts can spam 429s.
- Workaround suggested by reporter: switch embedding provider to OpenAI.
- Closed stale, not conclusively fixed in that issue thread.

This is the closest public report to Jones's current Gemini quota concern.

#### #54279 — Gemini embeddings behind proxy fail with `fetch failed`

- Version: 2026.3.23-2.
- Affected in-app `memory_search` and CLI status/search/index with Gemini embeddings.
- Maintainer later said current main implements proxy handling via shared memory-host remote HTTP helper.

#### #73527 — regression: `memorySearch.provider: "gemini"` unknown

- Version: 2026.4.23.
- User reported config schema accepted Gemini but runtime provider registry said `Unknown memory embedding provider: gemini`.
- Maintainer said main restored Gemini memory embedding registration and users on 2026.4.23 needed a newer build.

This confirms Gemini memory-search path was still actively changing in late April.

### Runtime-tool vs CLI mismatch issues

#### #10931 / #18198 — runtime memory_search fails while CLI works

- Users configured Ollama/OpenAI-compatible local embedding endpoints.
- `openclaw memory search` CLI worked, but the runtime `memory_search` tool failed with `fetch failed`.
- Workarounds included using CLI directly or waiting for runtime path fixes.

#### #11721 — memory_search caused spawn EBADF / file descriptor pressure

- Version: 2026.2.6-3.
- User reported successful `memory_search` corrupting/spiking file descriptor state, making later `exec` spawn fail until Gateway restart.
- Later comments linked it to broader file descriptor pressure in memory/indexing/watchers; maintainers added watcher hardening.

This is not Jones's current symptom, but it shows `memory_search` was a real operational risk area in February/March.

### Feature request: local search without external API

#### #10393 — “memory_search: support local file search without external API”

- Filed because external API dependencies were undesirable for local file memory.
- Requested BM25/TF-IDF fallback and local embeddings.
- Maintainer later closed as implemented: current main provides provider-less local keyword/BM25 fallback and a local embedding provider.

This is important: the community explicitly asked for exactly the thing Jones cares about — memory that does not depend on a remote API key.

## Web search snapshot

DuckDuckGo searches surfaced mostly official docs/mirrors, GitHub issues, and third-party memory-search guides. Notable result categories:

- Official docs: memory config, memory search, builtin memory engine.
- Third-party guides about OpenClaw memory search and Gemini embeddings.
- GitHub issues: #6385, #8131, #15546, #54279, #73527.
- A ClawHub/llmbase “Agent Memory Setup v2 (Gemini Embeddings 2)” result, suggesting community packaging around Gemini-based memory setups.

I did not find strong broad forum/social evidence that “everyone” experienced exactly Jones's April transition narrative. The strongest evidence is in OpenClaw release notes and GitHub issues, not general social media.

## Interpretation for Jones / Jarvis

### Why January–March felt better

Jones's memory that early Jarvis felt like one continuous main consciousness is plausible. The likely difference is not that earlier memory was technically more robust, but that the architecture was more context-resident:

- More information was directly in bootstrap/project context or long-lived session context.
- Fewer channels/plugins/subagents meant fewer context boundaries.
- Daily notes and `MEMORY.md` were manually read or injected more directly.
- The system had fewer retrieval failure points.

This can feel more “alive” and continuous, even if it is less scalable.

### Why April feels more retrieval-dependent

April OpenClaw shifted toward a more scalable memory architecture:

- Daily memory is not automatically in every ordinary prompt.
- Active-memory tries to retrieve relevant memory before the main reply.
- memory-wiki compiles structured knowledge, but still needs explicit lookup/search.
- `memory_search` may need embedding providers for semantic recall.
- Codex OAuth does not solve embeddings, so a separate key/provider is visible.

This is more maintainable, but if `memory_search` fails, the agent feels less naturally continuous.

### What happens if Gemini quota 429s often

Not fatal:

- Main chat with `openai-codex/gpt-5.5` still works.
- Tools, git, browser, file reads, wiki_get, and explicit path retrieval still work.

Affected:

- Semantic recall via `memory_search`.
- Active-memory’s automatic pre-reply memory injection.
- Indexing/reindexing and embedding cache refresh.
- Fuzzy questions where Jones gives little path/context.

User-visible effect:

- Jarvis may need more explicit cues.
- Jarvis may fall back to reading `active-context.md`, daily notes, wiki pages, and repo reports.
- The “one continuous mind naturally remembering everything” feeling may weaken.

## Mitigation options found in docs/issues

### 1. Keep Gemini, but treat it as an accelerator

Use Gemini free quota for normal semantic memory search, but do not rely on it as the only continuity layer. This is Jarvis's current pragmatic path.

Recommended supporting practice:

- Keep `active-context.md` short and accurate.
- Put durable syntheses in memory-wiki.
- Keep key reports in repos.
- Use direct `read` / `wiki_get` when the path is known.

### 2. Explicitly configure Gemini provider

Current Jarvis auto-detects Gemini because `GOOGLE_API_KEY` exists and `agents.defaults.memorySearch` is unset. Explicit config would make intent clear but does not solve quota:

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "gemini",
        model: "gemini-embedding-001",
      },
    },
  },
}
```

### 3. Use local/Ollama/local GGUF embeddings

Docs and issues point to local embeddings as the long-term no-quota solution, but historical issues show the runtime path was buggy in 2026.1/2026.2 and still being improved in April.

Current docs say options include:

- `provider: "local"` with optional `node-llama-cpp` and a GGUF embedding model.
- `provider: "ollama"` explicitly, with installed embedding model.
- QMD/local sidecar patterns.

Tradeoff: less quota risk, more local setup/maintenance.

### 4. OpenAI/Voyage/Mistral/Copilot embeddings

- OpenAI/Voyage/Mistral need API keys and likely billing.
- GitHub Copilot embedding provider was added in 2026.4.15 and may be attractive if a Copilot subscription is already available.
- Codex OAuth alone does not satisfy embeddings.

### 5. Push OpenClaw issue/feature request if current behavior contradicts docs

Docs say embeddings unavailable should still allow lexical ranking/FTS. If runtime `memory_search` returns fully unavailable on Gemini quota instead of degraded FTS results, that may be worth filing or checking against latest main.

Potential issue title:

> `memory_search` Gemini 429 should degrade to lexical FTS instead of returning unavailable

## Recommendation for Jarvis

Short-term:

1. Keep Gemini as the default because it is free and already configured.
2. Do not run heavy `openclaw memory index --force` loops during quota exhaustion.
3. Treat `memory_search` as best-effort; use active-context/wiki/reports as the dependable backbone.
4. Consider explicitly setting `memorySearch.provider: "gemini"` only for clarity, not as a fix.

Medium-term:

1. Test whether current 2026.4.26 actually falls back to lexical search when Gemini embeddings 429. If not, consider filing an upstream issue.
2. Evaluate local/Ollama embeddings after the Tokyo trip, when there is time to test without destabilizing travel workflows.
3. Keep `MEMORY.md` lean and use memory-wiki/repo reports for durable structured knowledge so continuity does not depend entirely on embeddings.

## Source files in this project

- `sources/github-official-repo-and-releases.txt` — official repo/release list snapshot.
- `sources/release-v2026.4.*.txt` — release note snapshots for April versions.
- `sources/issue-*.json` / `sources/issue-*.txt` — selected GitHub issue evidence.
- `sources/local-openclaw-docs-rg.txt` — local docs search around memory/search/embeddings.
- `sources/web-search-snapshots.txt` — DuckDuckGo result snapshot.

## Confidence

High confidence on:

- Current OpenClaw docs requiring on-demand `memory_search` / `memory_get` for ordinary daily memory access.
- April release notes making active-memory/memory retrieval much more visible.
- Existence of similar GitHub issues around external API keys, Gemini embedding failures, 429 handling, and local/QMD/runtime mismatches.

Medium confidence on:

- Exact internal timeline of when `memory_search` first shipped; public issues show it existed by 2026.1.30, but the current operator-visible architecture matured through February–April.
- Whether Jarvis’s January–March continuity came primarily from prompt/session persistence versus earlier OpenClaw memory internals. The user experience strongly suggests prompt/session continuity played a major role, but this report did not reconstruct old runtime code in detail.
