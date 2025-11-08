# CalmVoice AI

CalmVoice is an empathetic, voice-first companion that turns every conversation into real-time emotional support. The app captures live speech, understands the emotional context, and replies with adaptive text and synthesized speech so users can follow a guided well-being journey end-to-end.

- Voice input → Whisper transcription → Emotion analysis → Context-aware coaching.
- AI replies are delivered as both text and personalized audio, tuned to match the user’s emotional state.
- Built with a lightweight React/Vite front end and a Node.js WebSocket backend powered by OpenAI APIs.

---

## Table of Contents
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Runbook](#runbook)
- [Project Structure](#project-structure)
- [WebSocket Contract](#websocket-contract)
- [Development Notes](#development-notes)
- [Troubleshooting](#troubleshooting)
- [Roadmap Ideas](#roadmap-ideas)
- [Contributing](#contributing)

---

## Features
- **Emotion-aware coaching** – Detects happiness, sadness, anger, anxiety, or neutral tone from user speech and adapts techniques accordingly.
- **Adaptive voice synthesis** – Generates OpenAI TTS responses with dynamic speed and pitch to mirror the detected emotion.
- **Session state machine** – Tracks conversation phases (greeting, assessment, intervention, follow-up) and recent history to maintain consistent guidance.
- **Real-time UX** – WebSocket back channel keeps transcription, AI text, and audio replies flowing with latency-friendly updates.
- **Resettable sessions** – Users can restart at any time, triggering a fresh context, welcome message, and tailored response.
- **Production-ready hooks** – Modular architecture for monitoring, logging, and future analytics integrations.

---

## Architecture
1. **Client capture** – The React client records audio via `MediaRecorder` and streams it as base64 to the backend when the user stops speaking.
2. **Transcription** – The Node.js server forwards audio to OpenAI Whisper (`audio.transcriptions`) and emits the transcript back to the UI.
3. **Emotion inference** – The transcript is analyzed with GPT-4o to derive a single dominant emotion.
4. **State orchestration** – A lightweight reducer (`stateMachine.js`) updates conversation context, phases, and history.
5. **Response generation** – GPT-4o-mini produces the coaching reply leveraging contextual history and emotion-specific techniques.
6. **Speech synthesis** – OpenAI TTS renders the reply as MP3 audio using emotion-matched voice settings; audio and text are pushed to the client over WebSocket.

---

## Tech Stack
- **Frontend:** React 19, Vite, custom hooks, CSS modules.
- **Backend:** Node.js (ESM), Express (HTTP), `ws` for WebSockets.
- **AI Services:** OpenAI Whisper, GPT-4o, GPT-4o-mini, TTS-1.
- **Audio Tooling:** `MediaRecorder`, `@ffmpeg-installer/ffmpeg` (for broader platform support).

---

## Getting Started

### Prerequisites
- Node.js 18+ (OpenAI SDK requires modern fetch support).
- npm 9+ or compatible package manager.
- OpenAI API key with access to Whisper, GPT-4o, GPT-4o-mini, and TTS-1 endpoints.
- (Optional) `ffmpeg` on your PATH if you plan to expand local audio processing.

### 1. Clone and Install
```bash
git clone https://github.com/KasraBakht/CalmVoice-FinalVersion.git
cd CalmVoice-FinalVersion

# Backend
cd server
npm install

# Frontend
cd ../client
npm install
```

### 2. Configure Environment
Create `server/.env`:
```
OPENAI_API_KEY=sk-...
PORT=5000                 # optional, defaults to 5000
```

Create `client/.env` to point the UI at your backend (optional if defaults are fine):
```
VITE_WS_URL=ws://localhost:5000/ws
```

### 3. Run Locally (two terminals)
```bash
# Terminal 1 - backend
cd server
npm run dev

# Terminal 2 - frontend
cd client
npm run dev
```

Open the Vite dev server URL (usually `http://localhost:5173`) and allow microphone permissions when prompted.

---

## Configuration

| Variable | Location | Description | Default |
|----------|----------|-------------|---------|
| `OPENAI_API_KEY` | `server/.env` | OpenAI credential for Whisper, GPT-4o, TTS-1 | — |
| `PORT` | `server/.env` | HTTP/WebSocket port for Express server | `5000` |
| `VITE_WS_URL` | `client/.env` | WebSocket endpoint consumed by the React app | `ws://localhost:5000/ws` |

---

## Runbook
- **Start backend:** `npm run dev` (or `npm start` for production mode).
- **Start frontend:** `npm run dev` (hot reload) or `npm run build && npm run preview`.
- **Lint client code:** `npm run lint` inside `client/`.
- **Deploy:** Host the server on any Node-friendly platform (Render, Railway, Fly.io) and serve the Vite build statically (Netlify, Vercel). Ensure `VITE_WS_URL` points to the deployed WebSocket endpoint.

---

## Project Structure
```
CalmVoice-FinalVersion/
├─ client/                # React/Vite single-page app
│  ├─ src/
│  │  ├─ App.jsx         # Root UI integrating recorder, chat, and status banners
│  │  ├─ ws.jsx          # Reusable WebSocket hook
│  │  └─ components/     # ChatBox, MicRecorder, assets
│  └─ public/
└─ server/                # Express + WebSocket + OpenAI orchestration
   ├─ server.js          # HTTP + WS bootstrap
   ├─ wsHandler.js       # WebSocket message lifecycle
   ├─ openaiService.js   # Whisper, GPT, TTS helpers & emotion logic
   └─ stateMachine.js    # Conversational context reducer
```

---

## WebSocket Contract

**Inbound messages (client → server):**
- `{"type": "AUDIO_COMPLETE", "audioBuffer": "<base64>"}` – final chunk of recorded speech.
- `{"type": "RESET_SESSION"}` – restart conversation state.

**Outbound messages (server → client):**
- `{"type": "USER_TRANSCRIPT", "text": "...", "emotion": "sad"}` – transcription and detected emotion.
- `{"type": "TEXT", "text": "..."}` – AI text reply.
- `{"type": "AUDIO", "audio": "<base64-mp3>"}` – synthesized speech matching the reply.
- `{"type": "ERROR", "message": "..."}` – operational or AI errors surfaced to the UI.

---

## Development Notes
- The backend uses lazy OpenAI client instantiation to prevent redundant connections; reuse `getOpenAI()` for new features.
- `stateMachine.js` is intentionally simple—extend it to model multi-turn therapy protocols or track more user metrics.
- When experimenting with longer conversations, consider persisting `context.history.fullConversation` externally or pruning client-side.
- The UI intentionally stops any currently playing AI audio when a user begins recording, ensuring natural turn-taking.

---

## Troubleshooting
- **Microphone blocked:** Ensure the browser has microphone permission and is served over HTTPS (or `localhost`).
- **WebSocket closed:** Confirm backend is running and `VITE_WS_URL` matches the server origin/path.
- **OpenAI 401 errors:** Double-check `OPENAI_API_KEY` and that required models are enabled for your account.
- **Audio playback issues:** Some browsers require a user gesture before playing audio; start by clicking the page once if playback is silent.

---

## Roadmap Ideas
- Streaming partial transcripts and responses for lower latency.
- Emotion detection from raw audio features in addition to text analysis.
- Persistent user profiles with optional journal exports.
- Mobile-friendly layout and dark mode.
- Analytics dashboard for session insights (with privacy safeguards).

---

## Contributing
Pull requests are welcome! If you plan a larger change:
1. Open an issue outlining your proposal.
2. Discuss the approach and agree on scope.
3. Submit a PR with clear commits and testing notes.

---

