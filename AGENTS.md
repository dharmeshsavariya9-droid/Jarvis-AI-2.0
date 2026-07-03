# AEGIS — Personal AI Assistant Architecture (AGENTS.md)

This document provides a comprehensive system architecture and development guidelines for AI agents working on this codebase in future sessions.

## Project Overview
AEGIS is an interactive, highly tailored personal AI assistant designed with a futuristic sci-fi terminal theme (calm, dry wit, highly efficient). The interface is high-fidelity and uses real-time browser features (Speech Recognition and Speech Synthesis) to deliver an immersive HUD-style experience.

### Tech Stack
- **Framework:** TanStack Start (file-based routing & server-side API generation)
- **Runtime:** Node.js (Vite & ES modules)
- **AI Integration:** TanStack AI with multi-provider fallbacks (Anthropic, OpenAI, Gemini, Ollama)
- **Styling:** Custom CSS with sci-fi variables (dark-mode theme with cyber-cyan and amber accents)
- **Features:** Client-side speech-to-text (Web Speech API) and text-to-speech (synthesis), instant quick commands, session logging, and interactive responsive rotating rings.

---

## Key Directories and Files

```
├── .agents/
├── public/                 # Static assets
├── src/
│   ├── lib/
│   │   ├── ai-hook.ts      # Custom hook wrapping TanStack useChat fetching from /api/chat
│   │   └── weather-tools.ts# Weather tool definition for AI integration
│   ├── routes/
│   │   ├── __root.tsx      # Root document layout with meta title and styling imports
│   │   ├── api.chat.ts     # POST API handler which configures LLM providers, tools, and streams SSE
│   │   └── index.tsx       # Home route containing the main AEGIS React application and states
│   ├── router.tsx          # TanStack Router configuration
│   └── styles.css          # Core cyberpunk UI definitions, variables, and animations
├── package.json
└── vite.config.ts
```

---

## Architectural & State Design

### 1. Unified State Flow
The application's active visual and functional state is determined dynamically based on three reactive flags:
- `isListening`: browser is actively capturing voice input (microphone active).
- `isLoading`: `useAIChat` is streaming/fetching the model's response.
- `isSpeaking`: Speech Synthesis is reading out the response.

Calculated state hierarchy:
```typescript
let activeState: 'idle' | 'listening' | 'thinking' | 'speaking' = 'idle'
if (isListening) {
  activeState = 'listening'
} else if (isLoading) {
  activeState = 'thinking'
} else if (isSpeaking) {
  activeState = 'speaking'
}
```
This is bound directly to the container class `<div id="aegis-app" className={`state-${activeState}`}>` which drives the rotating SVG rings and pulsations in CSS.

### 2. Speech Synthesis Integration
AEGIS reads all generated assistant messages out loud. This is handled gracefully inside a `useEffect` watching changes on `isLoading` and `messages`. By maintaining a `lastSpokenMsgIdRef` reference, we ensure that:
- We only read newly-completed messages exactly once.
- We do not trigger speech synthesis during the stream, but instead wait until completion.
- Speech is automatically cancelled if the user cancels or clears the log.

### 3. Clear Log Mechanism
Instead of attempting to mutate the internal state of `useChat` directly (which is read-only), AEGIS uses a high-performance slice window offset:
- A local `clearedCount` offset is stored in state.
- The UI renders `messages.slice(clearedCount)`.
- Clicking "CLEAR LOG" sets `clearedCount = messages.length`, instantly clearing the terminal screen without breaking hook state or session history.

---

## Style Guidelines and Conventions

- **Theme Variables:** Colors, borders, shadows, and animations are strictly controlled via CSS variables in `src/styles.css` (e.g., `--void`, `--cyan`, `--amber`, `--red`). Always use these variables when editing components to maintain design consistency.
- **Microphone Support:** Always verify `micSupported` on mount since SpeechRecognition is a browser-dependent feature. Fallback to text input gracefully with appropriate browser notice.
- **Formattings:** AEGIS instructions mandate *never using markdown formatting* in responses because responses are read aloud via TTS and displayed as plain logs. This is strictly reinforced via the backend `SYSTEM_PROMPT`. Keep responses to 1-4 sentences.
