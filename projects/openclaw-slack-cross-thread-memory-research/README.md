# OpenClaw Slack Cross-Thread Memory Research

## Why this exists

Jones's actual Slack workflow is thread-heavy. Thread continuity inside one Slack thread is now working much better in OpenClaw, but that still does not automatically solve the next problem:

> If Jones opens a different Slack thread, how can Jarvis still feel like the same Jarvis without forcing Jones to re-explain everything?

This report researches existing design patterns, frameworks, and memory products that address the broader version of that problem: **thread-scoped short-term context + shared cross-thread memory**.

The goal is not to copy some random external tool blindly. The goal is to extract the best ideas and decide what is realistic for OpenClaw + Slack + Jones's workflow.

---

## Executive summary

The strongest recurring design pattern across modern agent systems is:

1. **Keep short-term memory thread-scoped**
   - one Slack thread or one conversation keeps its own local history
2. **Add a shared memory layer across threads/sessions**
   - user facts, preferences, active projects, recent decisions, and ongoing workstreams live outside the thread
3. **Do not dump everything into one giant memory blob**
   - separate stable long-term memory from changing working memory
4. **Retrieve selectively at thread start / task start**
   - new threads should not load the whole world, only the most relevant shared context

In other words, the industry answer is **not** “make all Slack threads secretly be one session.”
The industry answer is closer to:

> “Keep sessions local, but build a shared memory layer that can be recalled across sessions.”

This matches Jones's pain point almost exactly.

---

## What OpenClaw already gives us

From OpenClaw docs and actual local behavior:

- The Gateway is the system of record for routing and sessions
- Sessions are naturally scoped by sender/thread/channel context
- Workspace files are already the practical memory substrate
- `MEMORY.md` + `memory/YYYY-MM-DD.md` are already encouraged as persistence layers

So OpenClaw already has:
- **thread/session memory**
- **file-based long-term memory**

What it does **not** yet give by default is a polished middle layer like:
- shared working memory across Slack threads
- cross-thread recall at thread start
- automatic promotion of important thread conclusions into reusable memory

That missing middle layer is exactly the opportunity.

---

## External patterns worth borrowing

## 1. LangGraph / LangChain: thread-scoped short-term + namespace-scoped long-term memory

LangChain/LangGraph's memory docs describe two memory scopes very clearly:

- **Short-term memory**: thread-scoped, persisted per conversation/thread
- **Long-term memory**: shared across sessions/threads, stored in custom namespaces

This is one of the cleanest conceptual matches to the OpenClaw + Slack problem.

### Why this matters

It validates a key design principle:

- Slack thread continuity should remain local
- Cross-thread continuity should come from a second memory layer, not by merging all threads into one transcript

### Useful idea to borrow

Use at least **two memory scopes**:

1. **Thread memory**
   - local transcript and local working context
2. **Shared user/work memory**
   - reusable facts across all threads

### Practical translation to OpenClaw

Equivalent OpenClaw-style layers could be:

- `MEMORY.md` = stable long-term memory
- `memory/active-context.md` = current cross-thread working memory
- session JSONL logs = local thread history

### Source concepts

- LangChain memory overview: short-term memory is thread-scoped, long-term memory is shared across threads/sessions
- LangGraph persistence: checkpointers persist per-thread state keyed by `thread_id`
- LangGraph positioning: agents need persisted memory across future interactions

---

## 2. LlamaIndex: hybrid memory with short-term queue + long-term memory blocks

LlamaIndex documents a memory design where one agent memory object can include:

- recent short-term chat history
- extracted facts
- vector-retrieved past message batches
- static memory blocks

This is useful because it shows that “memory” is not one thing. It is a **composed stack**.

### Most relevant ideas

- keep a limited recent buffer for immediate context
- flush older material into long-term forms
- separate memory blocks by purpose:
  - static identity/context
  - extracted facts
  - retrieved prior episodes

### Practical translation to OpenClaw

OpenClaw could mimic this without fancy infrastructure by separating:

- **static memory**: `MEMORY.md`
- **active working memory**: `memory/active-context.md`
- **episodic recall**: search session logs or specific day logs when needed

This is much better than relying on one giant memory file.

---

## 3. Mem0: production memory engine framing

Mem0's framing is very direct:

- users should not need to repeat themselves
- memory should persist across users/agents/sessions
- memory should reduce prompt bloat while increasing continuity

### Why this matters

Mem0 is a useful product benchmark, even if OpenClaw does not adopt it directly.
It reinforces that the pain Jones feels is not niche. It is a mainstream agent product problem.

### Useful idea to borrow

Memory should be treated as a **distinct system layer**, not as an accidental side effect of transcript storage.

For OpenClaw, this suggests:
- stop treating thread history as the only real memory
- introduce an intentional shared-memory layer for Slack workflows

---

## 4. Anthropic guidance: prefer simple, composable patterns over overbuilt frameworks

Anthropic's “Building effective agents” guidance repeatedly pushes a simple theme:

- start with the simplest architecture that works
- use composable building blocks
- avoid unnecessary framework complexity

### Why this matters here

It argues against prematurely overengineering a huge memory platform.

For Jones's use case, a practical first implementation is probably:

- file-based shared memory
- explicit memory scopes
- retrieval at thread start
- selective promotion of important conclusions

not:

- a giant autonomous memory graph system on day one
- full vector stack unless truly needed

This is important because it means a good first solution may be relatively lightweight and still materially improve UX.

---

## 5. MCP direction: standardizing access to external memory systems

MCP itself is not a memory solution, but it matters because it standardizes how AI clients connect to external tools and data systems.

### Why this matters

If OpenClaw eventually wants to connect to:

- a dedicated memory store
- a notes system
- a vector database
- a project-state service

MCP-compatible tooling is a strong future path.

### Practical implication

The first implementation for Jones probably should **not** start with MCP. But if later we want:

- shared memory services
- cross-tool memory access
- reusable memory components across multiple AI clients

then MCP becomes strategically interesting.

---

## Slack-specific reality check

Slack threads are great for local organization, but they create a natural fragmentation problem for agents:

- humans understand “these are all still me talking to Jarvis”
- the agent runtime often sees “these are separate thread/session contexts”

So the core problem is not Slack itself. The problem is the mismatch between:

- **human mental model**: one ongoing relationship
- **runtime model**: many separate thread-scoped sessions

This is why thread continuity alone is not enough.

---

## Candidate design patterns for OpenClaw + Slack

## Option A. Shared file-based working memory layer (recommended first)

### Core idea

Add one cross-thread memory file, for example:

- `memory/active-context.md`

This file is not a full journal. It is a curated shared working state.

### Suggested sections

- Current focus
- Ongoing workstreams
- Recent important decisions
- Open loops / pending follow-up
- What Jones probably expects Jarvis to remember across threads
- Thread map (optional lightweight form)

### How it would work

When a new Slack thread begins:

1. read normal startup files
2. also read `memory/active-context.md`
3. optionally read today's daily memory
4. continue with that shared context in mind

### Strengths

- simplest meaningful upgrade
- aligns with OpenClaw's existing file-memory culture
- easy to inspect and debug
- low implementation risk
- no extra infra required

### Weaknesses

- manual/agent-maintained curation required
- may drift if not updated consistently
- not semantic search by default

### Recommendation

This is the best first move.

---

## Option B. Shared working memory + promotion rules

### Core idea

Same as Option A, but add explicit rules for what gets promoted where:

- thread-local details stay in thread/session logs
- important same-day details go to `memory/YYYY-MM-DD.md`
- cross-thread active context goes to `memory/active-context.md`
- durable preferences/identity/lessons go to `MEMORY.md`

### Why this is strong

This creates a real memory architecture instead of random note dumping.

### Strengths

- clean information lifecycle
- reduces clutter in long-term memory
- helps preserve “same Jarvis” feel across threads

### Weaknesses

- requires discipline
- still mostly file-based, so retrieval is only as good as file structure

### Recommendation

This should probably be paired with Option A from day one.

---

## Option C. Retrieval-augmented shared memory index

### Core idea

Build a searchable cross-thread memory index over:

- `MEMORY.md`
- `memory/*.md`
- selected session summaries
- maybe tagged thread summaries

At new thread start, retrieve the most relevant shared memories.

### Strengths

- more scalable over time
- less reliance on one curated file
- better for recalling older related work without exact filenames

### Weaknesses

- more implementation complexity
- retrieval quality becomes another system to tune
- may surface too much or too little without good ranking

### Recommendation

Good second-stage upgrade, not first move.

---

## Option D. Full external memory platform (Mem0 / vector memory / graph memory)

### Core idea

Use a dedicated memory engine outside the filesystem.

### Strengths

- potentially strongest cross-session personalization
- flexible retrieval, filtering, metadata, namespace support

### Weaknesses

- more infra and integration work
- new operational dependency
- may be overkill for current OpenClaw + Slack needs
- harder to debug than files

### Recommendation

Interesting benchmark, but probably too heavy as the first answer for Jones.

---

## Which tools / ideas seem most reusable for Slack even if not Slack-specific?

These are the most transferable ideas:

### Very reusable
- LangGraph's split between **thread memory** and **shared long-term memory**
- LlamaIndex's split between **recent buffer**, **fact extraction**, and **retrieved older episodes**
- Mem0's framing of memory as a **dedicated product/system layer**

### Potentially reusable later
- MCP-based memory services or connectors
- vector memory stores for recalled episodic context
- structured project-state stores

### Less likely to be the right first step
- giant autonomous agent-memory frameworks with lots of hidden machinery
- overbuilt graph memories before the file-memory lifecycle is even clean

---

## 2. Practical options Jones could actually choose from

Below is the condensed decision version.

## Choice 1. Lightweight file-memory upgrade

### What it is

Add:

- `memory/active-context.md`

and teach Jarvis to read/write it as the shared cross-thread working memory.

### What you get

- much better continuity across Slack threads
- very low engineering risk
- fully transparent and editable by humans

### What you do not get

- automatic semantic retrieval
- fancy ranking or memory extraction

### Best for

- immediate UX improvement
- low-risk rollout
- preserving debuggability

---

## Choice 2. File-memory upgrade + structured promotion policy

### What it is

Same as Choice 1, plus strong rules:

- thread detail -> thread/session
- same-day detail -> daily log
- cross-thread active work -> `active-context.md`
- durable memory -> `MEMORY.md`

### What you get

- coherent memory architecture
- less duplication
- much better cross-thread carryover

### Best for

- Jones's real workflow
- immediate practical implementation
- minimum viable durable memory design

### Recommendation

This is the best near-term choice.

---

## Choice 3. Retrieval-enhanced memory later

### What it is

Add search/retrieval over:

- memory files
- selected thread summaries
- maybe project-specific notes

### What you get

- better recall for older related work
- less dependence on one hand-curated active file

### Risks

- more engineering complexity
- more tuning work
- more chances to retrieve irrelevant context

### Best for

- phase 2 after Choice 2 works

---

## Choice 4. External memory product/infrastructure

### What it is

Use tools like Mem0 or build an equivalent memory layer with vector/DB backing.

### What you get

- strongest future extensibility
- cross-client / cross-agent possibilities

### Risks

- operational overhead
- integration complexity
- harder debugging

### Best for

- later stage, once the simpler architecture proves insufficient

---

## 3. If we implement this in OpenClaw, what should the design be?

## Recommendation: a 3-layer memory model

### Layer 1. Thread-local session memory

**Purpose:** preserve continuity within the current Slack thread

**Backed by:**
- existing OpenClaw session/thread transcript behavior
- session JSONL logs

**Rule:**
- do not try to merge all threads into one giant session

---

### Layer 2. Shared working memory

**Purpose:** carry current relevant context across Slack threads

**Backed by:**
- `memory/active-context.md`

**Suggested structure:**

```md
# Active Context

## Current focus
## Ongoing workstreams
## Recent decisions
## Pending follow-ups
## Important context to carry across threads
## Optional thread map
```

**Rule:**
- this is the default cross-thread handoff layer
- should stay concise and current

---

### Layer 3. Durable long-term memory

**Purpose:** preserve stable preferences, identity, lessons, and long-lived project facts

**Backed by:**
- `MEMORY.md`

**Rule:**
- only durable information goes here
- avoid dumping volatile thread details into it

---

## Supporting layer. Daily event log

**Purpose:** retain chronological details without polluting the shared layers

**Backed by:**
- `memory/YYYY-MM-DD.md`

**Rule:**
- raw daily events go here
- useful as a recall source, not the main cross-thread handoff object

---

## Proposed memory lifecycle

### On every important thread outcome

Ask:

1. is this only useful inside this thread?
   - keep in thread/session only
2. is this relevant later today or across nearby threads?
   - add to `active-context.md`
3. is this a durable preference, lesson, or stable project fact?
   - promote to `MEMORY.md`
4. is this just chronological traceability?
   - log in daily note

This gives OpenClaw a real memory policy instead of memory chaos.

---

## Proposed startup / recall behavior for Slack main-session contexts

When Jarvis starts work in a main-session Slack context, the ideal recall order would be:

1. `SOUL.md`
2. `USER.md`
3. today's daily memory
4. yesterday's daily memory
5. `MEMORY.md`
6. `memory/active-context.md`

If relevant, optionally search:
- session logs
- project-specific notes
- selected research/report files

### Why this works

- stable persona and user context load first
- recent chronology next
- durable memory after that
- current cross-thread work state last, right before action

This is likely enough to dramatically reduce “wait, what were we doing?” moments across threads.

---

## Optional next enhancement: thread summaries

If later needed, add a lightweight per-thread summary mechanism:

- one short summary per meaningful Slack thread
- promoted only when a thread had actual decisions/outcomes

These could live in:
- `memory/thread-summaries/`
- or be embedded into daily notes

This would improve episodic retrieval without having to inspect raw session logs every time.

---

## Optional next enhancement: retrieval at thread start

Once the file-memory system is disciplined, a retrieval step could be added:

- use a search tool over `MEMORY.md`, `active-context.md`, daily notes, and thread summaries
- inject only top relevant results into new thread startup context

This is where semantic search or memory indexing becomes genuinely helpful.

But only after the underlying memory hygiene is already good.

---

## What I think Jones actually wants

Not “unlimited memory.”

What Jones really seems to want is:

- if a topic matters across threads, Jarvis should carry it forward
- if something was just decided, a new thread should not feel amnesiac
- if the work is ongoing, Jarvis should behave like one continuous collaborator

That means the target is **continuity UX**, not abstract memory maximalism.

So the best design is probably:

### Desired architecture

1. **Slack thread = local working context**
2. **active shared memory = cross-thread current context**
3. **MEMORY.md = durable identity/preferences/lessons**
4. **daily notes = chronological recall / raw event log**
5. **optional retrieval over notes/logs = deeper recall when needed**

This is simple, legible, and aligned with both OpenClaw's current design and broader agent-memory best practices.

---

## Suggested concrete implementation path

## Phase 1, immediately practical

Create and maintain:

- `memory/active-context.md`

Use it for:
- current projects
- current important threads
- recent decisions that should survive thread switching
- pending follow-ups

Then modify Jarvis operating behavior so that in main-session Slack contexts it reads:

1. `SOUL.md`
2. `USER.md`
3. today's + yesterday's daily memory
4. `MEMORY.md`
5. `memory/active-context.md`

This alone would materially improve cross-thread continuity.

---

## Phase 2, better information hygiene

Adopt explicit promotion rules:

- thread-local detail -> leave in session/thread
- same-day log -> daily note
- cross-thread active work -> `active-context.md`
- long-term durable learning -> `MEMORY.md`

This is the minimum viable memory architecture.

---

## Phase 3, optional retrieval upgrade

If needed later:

- add tagging to active workstreams
- add search/retrieval over selected memory files and thread summaries
- optionally build a project-indexed or workstream-indexed recall layer

Only do this if Phase 1 and 2 still feel insufficient.

---

## Recommended keywords and areas to keep researching

Useful search terms for future research:

- `slack ai agent cross thread memory`
- `thread scoped memory long term memory agent`
- `langgraph short term memory long term memory threads`
- `mem0 cross session memory agents`
- `llamaindex memory blocks short term long term`
- `slack bot thread context memory`
- `agent shared working memory vs long term memory`
- `conversation memory namespaces agents`
- `model context protocol memory tools`

Useful tool / framework categories:

- LangGraph / LangChain memory
- Mem0
- LlamaIndex memory
- MCP-compatible memory tools
- vector-store-backed agent memory
- shared workspace file-memory patterns

---

## Final recommendation

For OpenClaw + Slack, the best next move is **not** to hunt for a magic Slack-specific plugin first.

The strongest near-term design is:

1. keep Slack threads as local session containers
2. add a shared cross-thread working memory file
3. define promotion rules between thread logs, daily notes, active context, and long-term memory
4. later, optionally add retrieval/search over those layers

So the immediate recommendation is:

> Build a lightweight shared working-memory layer inside the existing OpenClaw file-memory system, rather than trying to force all Slack threads into one session.

That gives the best tradeoff between:
- continuity
- transparency
- low implementation cost
- low operational risk
- future extensibility

---

## Sources consulted

- OpenClaw docs: multi-channel gateway, session-centric architecture, file-memory conventions in templates/docs
- LangChain / LangGraph memory overview: thread-scoped short-term vs shared long-term memory
- LangGraph persistence docs: thread-keyed checkpoints and thread state
- LangGraph memory docs: short-term thread memory + long-term shared memory
- LlamaIndex memory docs: mixed short-term + long-term memory blocks
- Mem0 overview: memory as a dedicated continuity layer across sessions
- Anthropic engineering guidance on building effective agents: prefer simple composable patterns before heavy frameworks
- MCP docs: standardized access to external tools and data systems, relevant for future memory-service integration
