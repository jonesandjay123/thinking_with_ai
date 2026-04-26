# Gemini Voice → Firestore / Trip Planner Integration Research

> Date: 2026-04-25  
> Question: Can Jones use Gemini voice / Google Home / Google-native AI entry points to control a Firebase Firestore-backed product such as `trip-planner`?

## Short answer

Yes, but not in the exact shape that older Google Assistant developers might expect.

The important distinction is:

- **Gemini API / Firebase AI Logic** can be embedded into our own app or backend. This is very feasible.
- **Gemini app / Gemini Live / Google Home voice as a native Google surface** does **not currently appear to expose a general public “custom Gemini connected app / plugin” API** where an arbitrary personal web app can register tools like `addTripCard()` and be invoked directly from the consumer Gemini app.
- **Google Home / Assistant smart-home voice** can call third-party cloud fulfillment, but it is designed around smart-home devices, traits, and `SYNC` / `QUERY` / `EXECUTE` intents, not arbitrary conversational app commands.

So the practical answer is:

> Build a *Gemini-powered voice interface for trip-planner* using Firebase AI Logic / Gemini Live API / function calling, and optionally expose a narrow Google Home integration through smart-home-style commands if Jones really wants “Hey Google” entry.

For `trip-planner`, the clean MVP is **not** “make the consumer Gemini app call Firestore directly.” It is:

```text
Voice UI we control
  ↓
Gemini model with function declarations
  ↓
Existing Jarvis/trip-planner callable functions
  ↓
Firestore trips/main
  ↓
React app updates through onSnapshot
```

## Current local architecture this can plug into

`trip-planner` already has the right backend shape for this.

```text
~/Downloads/code/trip-planner
├── React / Vite frontend
├── Firebase Hosting
├── Firestore document: trips/main
└── Cloud Functions: functions/index.js
    ├── jarvisAddCandidateCard
    ├── jarvisUpdateCandidateCard
    ├── jarvisMoveCardToSlot
    ├── jarvisRenameDayLabel
    ├── jarvisInspectTrip
    ├── jarvisInspectDay
    ├── jarvisInspectCard
    ├── jarvisCreateTripBackup
    └── jarvisRestoreTripBackup
```

`jarvis-firebase-ops` is the local operations layer:

```text
~/Downloads/code/jarvis-firebase-ops/projects/trip-planner
├── OPERATIONS.md
├── .env.local
└── scripts/*.mjs
```

Those scripts call the deployed `jarvis*` callable functions with a shared secret. A Gemini voice interface can reuse the same idea, but it should probably go through a more formal `voiceCommand` callable instead of exposing the Jarvis shared secret to a browser client.

## What Google officially supports now

### 1. Old Conversational Actions are gone

Google sunset **Conversational Actions** on June 13, 2023. These were the old way to build custom conversational Google Assistant experiences with webhooks.

Source: Google’s sunset page says Conversational Actions were removed and no longer available after June 13, 2023. It also says other paths like App Actions, Smart Home, Actions from web content, and Media Actions were unaffected.

Implication:

- We should not plan on building an old-style “Talk to Trip Planner” Google Assistant action.
- Any blog/tutorial using Dialogflow + Actions on Google conversational actions is likely obsolete for this use case.

### 2. Gemini Connected Apps are curated / platform-controlled

Gemini Apps can connect to apps like Gmail, Calendar, Keep, Tasks, Google Home, YouTube Music, Spotify, WhatsApp, and device utilities depending on device/account/region.

Sources:

- Gemini Help: Gemini Connected Apps can use connected services, and available apps vary by Gemini app, device, country, and account type.
- Gemini Live Help: Live chats support a limited list of connected apps such as Google Calendar, Google Tasks, Google Keep, and various phone vendor notes/calendar apps; Google Home support is separately constrained.

Implication:

- Gemini can work with Google Keep / Notes because Google has made those surfaces available as Connected Apps.
- I did not find a public consumer-facing “register your own Gemini Connected App with arbitrary API tools” path analogous to OpenAI GPT Actions.
- For now, treating Gemini app as a native extensible automation shell is risky.

### 3. Gemini can control Google Home, but through Home’s supported device model

Gemini mobile app can use Google Home to control supported smart-home devices. Google’s Help docs list lights, outlets, switches, thermostats, curtains/blinds, media devices, vacuums, washers, coffee makers, and similar device categories.

Important constraints from Google’s docs:

- Google Home in Gemini requires the Gemini mobile app on Android, a personal Google account, Keep Activity on, and connection to the same Google Home account.
- The feature is not available in Gemini web, Google Messages, or Live chats in the cited Help page.
- It is for convenience, not safety/security-critical actions.
- It cannot add/delete devices and cannot complete some security actions.

Implication:

- If we make `trip-planner` look like a “device” or “scene” inside Google Home, voice control may be possible.
- But this is semantically awkward for travel planning. It works best for narrow commands like “activate Tokyo planning mode” or “start today’s itinerary,” not for rich card editing.

### 4. Google Home Cloud-to-cloud can call our fulfillment service

Google Home Cloud-to-cloud integrations let Google Assistant / Google Home send smart-home intents to a third-party fulfillment service. The supported intent types include:

- `SYNC` — list devices/capabilities
- `QUERY` — ask current device state
- `EXECUTE` — execute a command
- `DISCONNECT` — account unlink event

Google Home passes a third-party OAuth2 access token in the Authorization header.

Implication:

- Technically, we can build a Cloud Function fulfillment endpoint and map smart-home `EXECUTE` commands to Firestore mutations.
- But it requires account linking / OAuth and must fit Google Home’s device/trait model.
- This is better for “voice trigger buttons” than for open-ended planning.

### 5. Firebase AI Logic gives first-class Gemini API access inside Firebase apps

Firebase AI Logic provides client SDKs for Gemini models on web, Android, iOS, Flutter, Unity, etc. It supports:

- natural-language and multimodal prompts
- chat experiences
- structured output
- function calling
- Gemini Live API / bidirectional streaming audio
- App Check and per-user rate limits
- integration with Firebase services like Firestore, Storage, and Remote Config

Implication:

- Since `trip-planner` is already Firebase-backed, Firebase AI Logic is the most natural way to add Gemini-powered AI features.
- It does **not** automatically make the consumer Gemini app call our app, but it lets our app become a Gemini-powered voice app.

### 6. Gemini function calling is the right bridge from natural language to existing functions

Firebase AI Logic function calling lets a Gemini model choose from declared functions and return structured arguments. The app/backend then calls the actual function and sends the result back to Gemini.

For `trip-planner`, examples of function declarations could be:

```ts
addTripCard({
  title: string,
  date?: string,
  zone?: "morning" | "afternoon" | "evening" | "flexible",
  area?: string,
  duration?: string,
  note?: string,
  tags?: string[],
  location?: {
    placeName?: string,
    address?: string,
    lat?: number,
    lng?: number,
    confidence?: "high" | "medium" | "low"
  }
})

moveCardToSlot({
  cardId: string,
  planId: string,
  date: string,
  zone: "morning" | "afternoon" | "evening" | "flexible",
  index?: number
})

inspectDay({
  planId: string,
  date: string
})

renameDayLabel({
  planId: string,
  date: string,
  label: string
})
```

Then a voice prompt like:

> “把秋葉原排到 5/2 下午，然後幫這天標成 Agent Body 採買日。”

can become two tool calls:

1. `moveCardToSlot({ cardId: "...", planId: "...", date: "2026-05-02", zone: "afternoon" })`
2. `renameDayLabel({ planId: "...", date: "2026-05-02", label: "Agent Body 採買日" })`

## Integration options for Jones

### Option A — Recommended MVP: Gemini voice inside `trip-planner`

Build a microphone button or “voice command” panel inside the `trip-planner` web app.

Architecture:

```text
Trip Planner web app
  ↓ mic / text command
Firebase AI Logic Gemini model
  ↓ function calling decision
Cloud Function: tripPlannerVoiceCommand
  ↓ validates user + resolves card/date/plan ambiguities
Existing jarvis* mutation helpers or internal mutation functions
  ↓
Firestore trips/main
  ↓
React onSnapshot updates UI
```

Pros:

- Fits current Firebase architecture.
- Avoids obsolete Assistant Actions.
- Can support rich trip-planning language.
- Can use existing Firestore and callable function logic.
- Can ask clarification questions in UI when ambiguous.
- Works on desktop/mobile browser if microphone permissions are available.

Cons:

- It is “Gemini inside our product,” not “native Gemini app controls our product.”
- Background/lock-screen behavior depends on browser/PWA limitations.

Best first commands:

- “今天行程念給我聽。”
- “把中野百老匯加到 5/2 下午。”
- “幫 5/3 加一張午餐候選卡：鳥貴族。”
- “查一下 5/2 下午有哪些卡沒有座標。”
- “備份目前行程，label 叫 before-voice-test。”

### Option B — Native-feeling Android route: small Android companion app + Gemini / Assistant entry

Build a tiny Android companion app for Jones’s phone.

Possible pieces:

- Firebase Auth / Firestore / callable functions
- Firebase AI Logic Gemini Live API for voice
- Android shortcuts / App Actions-style surfaces if current Android Assistant/Gemini integration supports the desired intents
- Push/deep links back into the web app

Pros:

- Better voice/microphone/background integration than web.
- More plausible path for “Hey Google” / Gemini mobile assistant interactions than a pure web app.
- Could later become the general Jarvis mobile control surface.

Cons:

- More engineering than a web MVP.
- Android App Actions / Gemini mobile behavior changes over time and may not give full arbitrary API invocation.
- Requires installing an app.

### Option C — Google Home Cloud-to-cloud pseudo-device

Expose `trip-planner` as one or more virtual Google Home devices/scenes.

Examples:

```text
Device: Tokyo Trip Planner
Trait: OnOff / Scene / StartStop / Modes (depending on fit)
Commands:
- “Hey Google, turn on Tokyo Trip Planner”
- “Hey Google, start Tokyo Day Briefing”
- “Hey Google, activate Tokyo backup”
```

Fulfillment:

```text
Google Home / Assistant
  ↓ action.devices.EXECUTE
Cloud Function fulfillment endpoint
  ↓ OAuth validation
Trip Planner command mapper
  ↓
Firestore / callable mutation
```

Pros:

- Real Google Home / “Hey Google” path.
- Good for narrow, routine-like commands.
- Can trigger routines.

Cons:

- Smart-home model is not designed for arbitrary travel planning.
- Requires OAuth/account-linking and device/trait mapping.
- Potential certification/policy overhead if made public.
- Rich commands like “move this card after that card” are a bad fit.

Recommendation:

- Do not start here unless the goal is specifically Google Home voice trigger demos.
- If used, keep commands narrow and reversible.

### Option D — Google Keep / Notes bridge

Because Gemini can write Google Keep / Notes, Jones could say something like:

> “Add to Keep: Trip Planner command — add Nakano Broadway to May 2 afternoon.”

Then some automation reads notes and converts them to Firestore operations.

Pros:

- Uses a Gemini-native app path Jones already has.
- Very low friction for capture.

Cons:

- This is a bridge/hack, not a first-class command channel.
- Consumer Google Keep automation APIs are limited compared with Firestore/Calendar/Gmail ecosystems.
- Parsing notes asynchronously creates ambiguity and delay.
- Easy to accidentally execute drafts or stale notes unless a strong command protocol is used.

Recommendation:

- Useful as an **inbox capture** path: collect trip ideas into notes, then Jarvis ingests them later.
- Not recommended for direct mutation commands unless every command requires confirmation.

### Option E — Keep current Slack/Jarvis path, but add voice input on Jones’s side

Jones can already dictate into Slack/mobile keyboard and Jarvis executes through `jarvis-firebase-ops`.

Pros:

- Already works.
- Human-readable audit trail.
- Jarvis can clarify and confirm.
- Lowest risk.

Cons:

- Not native Gemini/Home.
- Depends on Jarvis being online and the message channel.

Recommendation:

- Keep as fallback and control plane even after a voice MVP exists.

## Recommended phased plan

### Phase 1 — Web voice command panel in `trip-planner`

Goal: prove natural-language voice → structured function call → Firestore update.

Build:

1. Add a `VoiceCommandPanel` component to `trip-planner`.
2. Use browser speech capture or Firebase AI Logic Live API.
3. Add a server-side callable like `tripPlannerVoiceCommand`.
4. The callable should:
   - verify Firebase Auth user is allowed
   - call Gemini with a constrained tool schema
   - resolve card names to IDs
   - require confirmation for destructive or bulk actions
   - call existing mutation helpers
   - return a natural-language summary
5. Write every voice command to an audit log.

Suggested Firestore log:

```text
trips/main/activityLog/{logId}  // future split model
```

Current MVP can append an `activityLog` field or reuse whatever logging exists in Cloud Functions, but avoid growing `trips/main` indefinitely.

### Phase 2 — Text-first command box before full voice

Before microphone complexity, add a simple text command box:

```text
“把中野百老匯排到 5/2 下午”
```

If this works, voice is just another input modality.

### Phase 3 — Gemini Live / audio mode

Use Firebase AI Logic Live API for low-latency back-and-forth voice.

Use cases:

- “Read me 5/2.”
- “Move that later.”
- “Undo.”
- “What is missing coordinates?”

Guardrails:

- Read-only commands can execute immediately.
- Single-card reversible changes can execute with spoken confirmation.
- Delete/reset/restore/bulk changes must require explicit confirmation and ideally create backup first.

### Phase 4 — Optional Google Home shortcut layer

Only after the app-level voice loop works, add Google Home Cloud-to-cloud or routine triggers for narrow actions:

- “Start Tokyo trip briefing.”
- “Backup Tokyo trip planner.”
- “What is today’s first stop?”

Do not try to make Google Home the primary rich editing interface.

## Security / safety notes

Voice control should not reuse the current `TP_JARVIS_SHARED_SECRET` in browser code.

Recommended production shape:

- Frontend uses Firebase Auth.
- Callable verifies `context.auth` and allowed owner/editor role.
- Server owns privileged Firestore mutation logic.
- Gemini function declarations are narrow and typed.
- Destructive commands require confirmation.
- Any restore/reset/bulk mutation creates a backup first.
- Every command logs:
  - transcript
  - parsed intent
  - selected function(s)
  - before/after summary
  - actor UID/email
  - timestamp

## Concrete design for `trip-planner`

### New callable: `tripPlannerVoiceCommand`

Input:

```json
{
  "utterance": "把中野百老匯排到 5/2 下午",
  "activePlanId": "plan_1777054282117",
  "timezone": "America/New_York",
  "clientContext": {
    "selectedDate": "2026-05-02",
    "selectedCardId": null
  }
}
```

Output:

```json
{
  "ok": true,
  "needsConfirmation": false,
  "summary": "已把中野百老匯排到 Jarvis Suggest 的 5/2 下午。",
  "actions": [
    {
      "type": "moveCardToSlot",
      "cardId": "nakano_broadway",
      "planId": "plan_1777054282117",
      "date": "2026-05-02",
      "zone": "afternoon"
    }
  ]
}
```

If ambiguous:

```json
{
  "ok": false,
  "needsClarification": true,
  "question": "你說的秋葉原是指『秋葉原電器街』還是『秋葉原 Agent Body 採買』？",
  "choices": [
    { "label": "秋葉原電器街", "cardId": "akihabara_electric_town" },
    { "label": "Agent Body 採買", "cardId": "agent_body_shopping" }
  ]
}
```

### Tool schema set for MVP

Start with only safe, high-value tools:

- `inspectTrip`
- `inspectDay`
- `inspectCard`
- `addCandidateCard`
- `moveCardToSlot`
- `renameDayLabel`
- `backupTrip`

Delay these until guardrails are strong:

- `deleteCandidateCard`
- `deletePlan`
- `resetPlan`
- `restoreTripBackup`
- bulk import / repair scripts

## My recommendation

For Jones’s actual workflow, I would build this order:

1. **Text command box in `trip-planner` using Gemini function calling.**  
   Fastest validation. No microphone/browser voice edge cases yet.

2. **Voice button in `trip-planner` web/PWA.**  
   Use browser speech input or Gemini Live API depending on how “conversational” we want it.

3. **Gemini Live mode for hands-free trip editing.**  
   This gives the “talk to my trip board” feeling without waiting for Google to open custom Gemini Connected Apps.

4. **Optional Google Home Cloud-to-cloud mini integration for a few routine commands.**  
   Good demo / daily convenience, but not the main editing interface.

This gives us the real benefit Jones wants — voice-native control over a Firestore-backed product — while avoiding the trap of depending on a Google consumer surface that is not currently open as a general custom API platform.

## Sources checked

- Google Assistant Conversational Actions sunset overview: https://developers.google.com/assistant/ca-sunset
- Google Assistant Conversational Actions overview: https://developers.google.com/assistant/conversational/overview
- Gemini Connected Apps Help: https://support.google.com/gemini/answer/13695044
- Gemini Live Help / Connected Apps in Live: https://support.google.com/gemini/answer/15274899
- Gemini Google Home control Help: https://support.google.com/gemini/answer/15335456
- Google Home APIs overview: https://developers.home.google.com/apis
- Google Home Cloud-to-cloud get started: https://developers.home.google.com/cloud-to-cloud/get-started
- Google Home Cloud-to-cloud intents: https://developers.home.google.com/cloud-to-cloud/primer/intents
- Firebase AI Logic overview: https://firebase.google.com/docs/ai-logic
- Firebase AI Logic getting started: https://firebase.google.com/docs/ai-logic/get-started
- Firebase AI Logic function calling: https://firebase.google.com/docs/ai-logic/function-calling
- Firebase AI Logic Live API: https://firebase.google.com/docs/ai-logic/live-api
- Vertex AI Gemini function calling: https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/function-calling
