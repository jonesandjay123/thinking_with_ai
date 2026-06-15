# Repo Security Audit: last30days-skill and Agent-Reach

Date: 2026-06-14 EDT

Requested by: Jones

Repos reviewed:

- `mvanhorn/last30days-skill`: https://github.com/mvanhorn/last30days-skill
- `Panniantong/Agent-Reach`: https://github.com/Panniantong/Agent-Reach

## Executive Summary

My recommendation is to try `last30days-skill` first, but only in a sandboxed research profile with dedicated credentials. It is closer to the workflow Jones actually wants: recent social signal research across Reddit, X, YouTube, GitHub, Polymarket, HN, TikTok/Instagram/Threads, and web, then a grounded summary. It has a large test suite, explicit configuration docs, MIT license, and clear security-oriented code patterns, but it is still a young, fast-moving project that can touch sensitive API keys, browser cookies, and local env files.

`Agent-Reach` is strategically interesting, but I would not install it into the main OpenClaw/Jarvis environment yet. It is more of an internet-capability router and installer than a single research tool. The design is useful: multi-backend routing, doctor checks, and platform-specific fallbacks. The risk is also larger: it can install global tools, configure MCP entries, read browser cookies, write token config, and route through third-party CLIs/OpenCLI. It should be studied as an architecture reference or tested in a disposable VM/secondary account setup before any real adoption.

Short version:

- Best first trial: `last30days-skill`
- Best long-term architecture inspiration: `Agent-Reach`
- Do not give either repo Jones's main social accounts, main browser profile, or broad personal credentials
- Use dedicated bot/sandbox accounts, per-project env files, and no production posting permissions

## Repo 1: mvanhorn/last30days-skill

Source: https://github.com/mvanhorn/last30days-skill

Snapshot observed via GitHub and local shallow clone:

- Default branch: `main`
- License: MIT
- Created: 2026-01-23
- Latest repo push seen by GitHub metadata: 2026-06-10 UTC
- Latest release seen by GitHub metadata: `v3.3.0`, published 2026-05-17
- Local cloned package version: `3.3.2`
- GitHub stars observed: about 42k
- Forks observed: about 3.4k
- Primary language/runtime: Python 3.12+ with stdlib-heavy implementation
- Dependency surface: `pyproject.toml` declares no runtime Python dependencies; dev deps are `pytest` and `pytest-cov`

### What It Does

`last30days-skill` is an Agent Skill plus Python engine that runs multi-source recent-signal research. Its README and `SKILL.md` describe searches across Reddit, X/Twitter, YouTube, TikTok, Instagram, Threads, Hacker News, Polymarket, GitHub, Digg, Bluesky, Truth Social, Xiaohongshu, Pinterest, Perplexity, and web search.

The product idea is strong for Jones's workflow: it searches what people recently said and engaged with, then asks the hosting agent/model to synthesize a brief. This maps well to:

- AI opportunity radar
- Jyn Null content topic discovery
- market/social narrative scanning
- pre-meeting research
- trend validation before building a tool or article

### Security-Relevant Findings

Positive signs:

- Runtime dependency surface is intentionally small. `pyproject.toml` has no normal runtime dependencies.
- CI includes a test workflow and a security workflow.
- The security workflow runs `pip-audit` and TruffleHog, though both are currently advisory with `continue-on-error: true`.
- It warns if `.env` secret files are group/world-readable.
- It supports macOS Keychain loading for known credential names.
- HTTP helper redacts `key`, `api_key`, `token`, and `secret` query parameters in logs.
- Subprocess helper uses argument lists, timeout handling, and process-group cleanup rather than shell string execution.
- Codex `~/.codex/auth.json` fallback is intentionally skipped in current code; users must set `OPENAI_API_KEY` explicitly.

Risk areas:

- It can use many powerful credentials: `SCRAPECREATORS_API_KEY`, `OPENAI_API_KEY`, `XAI_API_KEY`, `OPENROUTER_API_KEY`, `AUTH_TOKEN`, `CT0`, `BSKY_APP_PASSWORD`, `TRUTHSOCIAL_TOKEN`, `BRAVE_API_KEY`, `EXA_API_KEY`, and more.
- X/Twitter access can use `AUTH_TOKEN` and `CT0`, which are browser-session level credentials.
- Browser credential extraction is supported through Firefox/Safari/Chrome/Brave paths depending on config.
- The repo vendors a Bird/X client subset under `skills/last30days/scripts/lib/vendor/bird-search/`.
- It touches many third-party platforms whose anti-automation policies can change quickly.
- Its own security scans are visibility-first, not blocking gates.

### Fit for Our Use Cases

High fit:

- Quickly answer "what are people saying now?"
- Validate whether a repo/product/topic is organically hot or just GitHub-trending inflated.
- Build repeatable Jyn Null research briefs before posting.
- Feed `thinking_with_ai` reports with source-backed current context.

Medium fit:

- Automated daily/weekly watchlists, if we carefully cap cost and avoid over-querying paid APIs.
- Competitive research across products.

Low fit:

- Anything requiring precise legal/financial/medical claims without human review.
- Any workflow that needs posting, liking, replying, or acting as Jones.

### Adoption Recommendation

Use as a controlled research tool, not a trusted always-on autonomous scanner yet.

Recommended trial path:

1. Install in a disposable/sandbox agent environment, not the main OpenClaw home directory.
2. Start with zero/low-risk sources: GitHub, HN, Polymarket, Reddit public paths if available, YouTube via `yt-dlp`, and web search.
3. Do not enable browser-cookie extraction on the first trial.
4. If X is needed, use a dedicated low-risk account and explicit `AUTH_TOKEN`/`CT0` stored in a per-project file with `chmod 600`.
5. Put `LAST30DAYS_MEMORY_DIR` inside an explicit project folder so generated briefs and raw results do not scatter.
6. Run a few known topics and compare results against manual web/GitHub checks before trusting it.

Risk rating: Medium.

My verdict: worth a sandbox trial.

## Repo 8: Panniantong/Agent-Reach

Source: https://github.com/Panniantong/Agent-Reach

Snapshot observed via GitHub and local shallow clone:

- Default branch: `main`
- License: MIT
- Created: 2026-02-24
- Latest repo push seen by GitHub metadata: 2026-06-12 UTC
- Latest release seen by GitHub metadata: `v1.5.0`, published 2026-06-11
- Local cloned package version: `1.5.0`
- GitHub stars observed: about 28.7k
- Forks observed: about 2.3k
- Primary language/runtime: Python 3.10+
- Runtime dependencies: `requests`, `feedparser`, `python-dotenv`, `loguru`, `pyyaml`, `rich`, `yt-dlp`
- Optional dependencies include `playwright`, `mcp[cli]`, and `browser-cookie3`

### What It Does

`Agent-Reach` is a capability router for agents. Instead of being one search engine, it tries to select and maintain working access paths for many platforms:

- web via Jina Reader
- GitHub via `gh`
- YouTube via `yt-dlp`
- Bilibili via `bili-cli` or OpenCLI
- X/Twitter via `twitter-cli`, OpenCLI, or Bird-like fallbacks
- Reddit via OpenCLI or `rdt-cli`
- Xiaohongshu via OpenCLI, xiaohongshu-mcp, or legacy xhs tools
- RSS via `feedparser`
- Exa through `mcporter`
- LinkedIn, V2EX, Xueqiu, Xiaoyuzhou podcast, and more

The strongest idea is `agent-reach doctor`: each channel has active backend detection and clear repair hints. This is the part most worth learning from for Jarvis/OpenClaw.

### Security-Relevant Findings

Positive signs:

- Has `SECURITY.md` with private advisory reporting guidance and response timeline.
- CI tests Python 3.10 through 3.13 and includes a wheel smoke-install gate.
- Keeps persistent config under `~/.agent-reach/config.yaml`.
- Writes config with owner-only permissions, using `os.open(..., 0o600)` to avoid a brief world-readable race.
- Provides `--safe` and `--dry-run` install modes.
- Installation docs explicitly warn not to use `sudo` unless user approved.
- Docs explicitly recommend secondary accounts for cookie-based platforms.
- Does not use `shell=True` in the inspected subprocess paths.

Risk areas:

- Installer can run global package installs, including `npm install -g mcporter`, `npm install -g @jackwener/opencli`, `pipx install`, and `uv tool install`.
- It can configure MCP entries through `mcporter config add`.
- It encourages agent-driven installation from `main.zip`, which is convenient but weakens reproducibility versus pinned release artifacts.
- It reads browser cookies via `rookiepy` or `browser-cookie3` if enabled.
- It stores cookies/API keys in `~/.agent-reach/config.yaml` rather than a system keychain.
- OpenCLI intentionally reuses a real browser login session through a browser extension and local daemon.
- Xiaohongshu MCP flow can involve Docker/container cookie injection.
- It routes through many third-party tools with their own trust and maintenance profiles.

### Fit for Our Use Cases

High fit:

- Architecture reference for Jarvis: doctor checks, backend selection, platform status reporting.
- A sandbox research VM where we want a broad "internet reader" stack.
- Chinese platform research experiments, especially Xiaohongshu/Bilibili route exploration.

Medium fit:

- One-off research tasks when we need a platform it handles better than our existing tools.
- Learning how current public projects solve multi-platform agent access.

Low fit:

- Direct installation into the main Jarvis/OpenClaw environment.
- Anything involving Jones's main browser, main social accounts, or high-trust credentials.
- Production automation where deterministic reproducibility matters.

### Adoption Recommendation

Do not install into the main environment yet.

Recommended trial path:

1. Test inside a disposable macOS user, VM, or container-like throwaway environment.
2. Use `agent-reach install --safe --dry-run` first.
3. Avoid `--channels=all`.
4. Enable one channel at a time and record every file/tool it adds.
5. Use dedicated secondary accounts for X, Xiaohongshu, Reddit, and any cookie-based platform.
6. Do not connect it to the `user` Chrome profile. If browser automation is needed, use the existing OpenClaw `openclaw` profile boundary or a fresh browser profile.
7. Treat OpenCLI/MCP entries as privileged local automation. Audit before enabling.

Risk rating: Medium-high.

My verdict: valuable to study, not yet safe to adopt wholesale.

## Side-By-Side Decision Table

| Dimension | last30days-skill | Agent-Reach |
|---|---|---|
| Core shape | Research skill + Python engine | Capability router + installer + doctor |
| Immediate use value | High | Medium |
| Main risk | Many API keys/browser tokens | Global installs, MCP/router surface, browser cookies |
| Credential handling | Env files + project env + macOS Keychain support | `~/.agent-reach/config.yaml` with 0600 permissions |
| CI maturity | Large pytest suite, advisory security scans | Multi-Python CI + wheel smoke gate |
| Runtime dependency surface | Very small Python runtime deps | Larger, plus optional browser/MCP deps |
| Best use for us | Trend/social signal research | Architecture reference and sandbox platform access |
| Install into main Jarvis? | Not yet; sandbox first | No |
| Overall recommendation | Trial carefully | Study and sandbox only |

## Recommended Next Steps

1. Create a small sandbox branch or separate test environment for `last30days-skill`.
2. Run three evaluation prompts:
   - `OpenClaw vs Hermes`
   - `AI social media automation tools`
   - `Xiaohongshu AI creator workflow`
3. Compare its output against manual web/GitHub checks and our existing browser/web_fetch workflow.
4. Do not add X cookies on day one. If needed later, use a throwaway X account.
5. For `Agent-Reach`, extract design ideas into a Jarvis note: platform router, doctor output, active backend, repair hints, safe/dry-run install mode.
6. Consider building a smaller internal "Jarvis Reach" pattern rather than importing Agent-Reach wholesale.

## Bottom Line

`last30days-skill` is the repo I would actually try first because it solves a concrete, repeatable research job. It is not risk-free, but its scope is easier to contain.

`Agent-Reach` is clever and directionally aligned with where agent tooling is going, but it wants to become infrastructure. Infrastructure deserves a slower trust path.
