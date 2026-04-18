# 2026-04-18 Implementation Addendum — Hermes Memory Refactor

This addendum records the actual overnight implementation that followed the earlier research/design report.

## What was implemented
- Added a **working-memory bridge** for recent validated operational facts:
  - `~/.hermes/memories/working-memory/recent-validated-operational-facts.md`
- Added a **maintenance policy** defining memory placement and anti-bloat rules:
  - `~/.hermes/memories/policies/memory-maintenance-policy.md`
- Added a **stable system/operations note** for durable operational anchors:
  - `~/.hermes/memories/projects/jarvis-operations.md`
- Rewrote active prompt memory files to point to the correct layers instead of relying on broad rediscovery.
- Tightened config guardrails:
  - `memory_char_limit: 6000 -> 3600`
  - `user_char_limit: 2200 -> 1800`
- Backed up important memory files before rewrite under:
  - `~/.hermes/migration/memory-maintenance/20260418T002834/`

## Key design outcome
The missing piece was not just “smaller MEMORY.md”, but an explicit layer between prompt memory and raw history. The new bridge keeps recent operational facts (paths, scripts, service state, repo anchors) accessible without promoting all of them into prompt memory.

## Example of the solved pain point
A path like `~/.hermes/scripts/codex_status.py` no longer has to be rediscovered from session transcripts; it now has a canonical operational-memory home and is referenced by the working-memory bridge.

## Snapshot repos updated
The canonical Hermes snapshot was synchronized into:
- `jarvis-os`
- `jarvis-mind-backup`

This turns the report’s recommendations into a documented, versioned implementation rather than just design guidance.
