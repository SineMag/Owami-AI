# Owami — Your AI Cooking Companion

A voice-first AI cooking assistant: find recipes, cook hands-free with real-time
step-by-step voice guidance, set timers by voice, handle substitutions, and keep
every suggestion safe against your allergies.

Built for the AssemblyAI Voice Agent Hackathon. **No fakes:** real microphone
input, real streaming transcription, real tool-calling agent.

## How the voice loop works

```
Browser mic ──PCM 16kHz──▶ AssemblyAI v3 WebSocket (real-time streaming STT)
                                 │ end-of-turn transcript
                                 ▼
        POST processVoiceTurn (Base44 backend function)
                                 │
                     Owami agent (tool loop over recipes,
                     cooking sessions, timers, preferences)
                                 │
                                 ▼
                 reply ──▶ browser speechSynthesis (spoken aloud)
```

1. `createVoiceSession` (backend) authenticates the user, checks the voice-turn
   entitlement, and creates a **temporary AssemblyAI streaming token** (the
   permanent API key never reaches the browser). The browser opens
   `wss://streaming.assemblyai.com/v3/ws` with that token and streams raw
   16 kHz PCM from the microphone.
2. AssemblyAI's turn detection delivers the completed utterance; the client sends
   it to `processVoiceTurn`, which runs an agent (AI SDK over Base44's AI
   gateway) with real tools: search recipes, start/advance cooking sessions,
   set/stop timers, substitutions, servings scaling, saved recipes, and
   persistent allergy/dietary preferences.
3. The reply is spoken with the browser's speech synthesis. While Owami speaks,
   the mic stream is muted to avoid echo; a single tap interrupts speech and
   resumes listening.
4. All state (recipes, cooking sessions, timers, conversation history,
   preferences, subscriptions) lives in Base44 entities with per-user row-level
   security.

## Safety

- Allergies are filtered **deterministically** in `search_recipes` (declared
  allergens + ingredient-level keyword matching), not just by prompt. The agent
  is additionally instructed never to suggest or guide cooking of allergen
  dishes, and it persists newly-mentioned allergies to the user's profile.
- The AssemblyAI API key is a server-side secret; the browser only ever holds a
  short-lived temporary token.

## Monetization

Free plan: 30 voice turns/day (enforced server-side). Pro: unlimited. The
Profile page has a clearly-labeled demo toggle to provision Pro so the gating
can be demoed; in production that button starts a real Stripe checkout session
(Stripe is the supported payment provider for this app's region).

## Environment

- `ASSEMBLYAI_API_KEY` — set as a Base44 app secret (dashboard → settings →
  environment variables). Get a key at https://www.assemblyai.com/dashboard.


## Tech stack

- **Frontend:** React + Tailwind CSS + shadcn/ui, WebSocket mic capture via Web
  Audio API (downsampled to 16 kHz PCM), speech synthesis output.
- **Voice intelligence:** AssemblyAI Real-Time Streaming (v3) for STT with
  intelligent turn detection; Base44 AI gateway + AI SDK agent loop for
  understanding and tool calling.
- **Backend:** Base44 entities (with RLS), backend functions, app secrets.