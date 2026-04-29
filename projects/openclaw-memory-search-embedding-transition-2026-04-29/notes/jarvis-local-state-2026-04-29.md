# Jarvis local state snapshot — 2026-04-29

- OpenClaw version: 2026.4.26 (be8c246)
- `agents.defaults.memorySearch`: absent / unset
- Memory provider selected by OpenClaw: `gemini (requested: auto)`
- Model: `gemini-embedding-001`
- Reason: `GOOGLE_API_KEY` is discoverable in the environment; Codex OAuth does not satisfy embeddings.
- Current issue: Gemini embedding quota/429 makes `memory_search` unavailable during some runs.
- Fallback reality: Jarvis can still use `active-context.md`, `MEMORY.md`, daily notes, wiki pages, repo reports, and direct file reads.
