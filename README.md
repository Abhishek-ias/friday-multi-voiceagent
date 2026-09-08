Friday Voice --- Friday Multi-Voice Agent

A realtime voice customer-support agent designed for natural
Hindi-English (Hinglish) code-switching.

Friday Voice is a realtime voice AI system for conversations where users
naturally switch between Hindi and English instead of choosing one
language. It combines streaming speech recognition, LLM response
generation, realtime WebRTC transport, and streaming speech synthesis.

Architecture

User microphone
      ↓
React frontend
      ↓ WebRTC
LiveKit Cloud
      ↓
Python LiveKit Agent
      ├── VAD / turn handling
      ├── Deepgram Nova-3 STT
      ├── Groq GPT-OSS 20B
      └── Rime Coda TTS
      ↓
LiveKit audio stream
      ↓
User speaker

Pipeline

Speech → LiveKit → Deepgram STT → Groq LLM → Rime TTS → LiveKit → Speech

The objective is not to translate every sentence. The objective is to
respond naturally in the user's mixed Hindi-English conversational
style.

Example:

User: "Mera phone mein network nahi aa raha, can you help me?"

Agent: "समझ गई। पहले network settings check करते हैं।"

Third-Party Services

Service              Purpose

LiveKit Cloud        Realtime WebRTC rooms and media transport
LiveKit Agents       Realtime voice-agent orchestration
Deepgram             Speech-to-text
Groq                 LLM inference
Rime                 Text-to-speech
React + TypeScript   Frontend
Vite                 Frontend tooling
Express              Backend API and LiveKit token generation

Exact Rime Configuration

The current application uses Rime through the LiveKit Agents Rime
plugin.

tts = rime.TTS(
    model="coda",
    speaker="luna",
    lang="eng",
    sample_rate=22050,
    use_websocket=True,
    segment="bySentence",
    speed_alpha=0.95,
)

Parameter                Exact value

Model ID             coda
Speaker              luna
Language             eng
Sample rate          22050 Hz
Audio format         PCM
Transport            WebSocket
Segmentation         bySentence
Speed                0.95 (speed_alpha)
HTTP endpoint        https://users.rime.ai/v1/rime-tts
WebSocket endpoint   wss://users-ws.rime.ai

use_websocket=True enables Rime WebSocket streaming.
segment="bySentence" uses sentence-level segmentation. The LiveKit
Rime integration documents PCM as the default audio format and documents
the HTTP and WebSocket endpoints above.

Official references:

https://docs.livekit.io/agents/models/tts/rime/

https://docs.livekit.io/reference/python/livekit/plugins/rime/

STT Configuration

Deepgram is configured as:

deepgram.STT(
    model="nova-3",
    language="hi",
    interim_results=True,
    smart_format=True,
    endpointing_ms=100,
    keyterm=[...],
)

This provides interim transcripts and Hindi-oriented recognition while
key terms help preserve important English technical/product vocabulary.

LLM Configuration

The application currently uses Groq's OpenAI-compatible API:

openai.LLM(
    model="openai/gpt-oss-20b",
    api_key=os.getenv("GROQ_API_KEY"),
    base_url="https://api.groq.com/openai/v1",
    temperature=0.2,
    max_completion_tokens=60,
    top_p=0.9,
    reasoning_effort="low",
)

Important: the implementation currently uses openai/gpt-oss-20b.
Older documentation referring to GPT-OSS 120B should not be treated as
the current implementation.

Turn Handling and Interruptions

Current interruption configuration:

interruption={
    "enabled": True,
    "mode": "vad",
    "min_duration": 0.35,
    "min_words": 2,
    "false_interruption_timeout": 1.5,
    "resume_false_interruption": True,
}

Endpointing:

endpointing={
    "mode": "fixed",
    "min_delay": 0.0,
    "max_delay": 0.6,
}

Preemptive generation is disabled:

preemptive_generation={
    "enabled": False,
}

Intended behavior:

Agent speaking
      ↓
User starts speaking
      ↓
VAD detects interruption
      ↓
Agent speech is interrupted
      ↓
New user turn becomes active
      ↓
STT → LLM → TTS

Project Structure

friday/
├── frontend/
│   └── Friday Voice/
│       ├── src/
│       │   ├── services/
│       │   │   └── livekitClient.ts
│       │   ├── App.tsx
│       │   └── ...
│       ├── package.json
│       └── ...
│
├── backend/
│   ├── agent.py
│   ├── server.js
│   ├── package.json
│   ├── requirements.txt
│   ├── .env
│   └── .env.example
│
├── README.md
├── RIME_EVIDENCE.md
└── .gitignore

Prerequisites

Recommended development environment:

Node.js + npm

Python 3.12

Git

LiveKit Cloud project

Deepgram API key

Groq API key

Rime API key

Python 3.12 is the currently used environment for the agent.

Environment Variables

Create backend/.env:

LIVEKIT_URL=wss://your-project.livekit.cloud
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_api_secret

DEEPGRAM_API_KEY=your_deepgram_api_key
GROQ_API_KEY=your_groq_api_key
RIME_API_KEY=your_rime_api_key

Frontend:

VITE_BACKEND_HTTP_URL=http://localhost:5000

Never put real credentials in .env.example, source code, or the
frontend bundle.

Setup

Clone

git clone https://github.com/Abhishek-ias/friday-multi-voiceagent.git
cd friday

Backend

cd backend
npm install
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt

Create backend/.env with the required credentials.

Frontend

Open another terminal:

cd frontend\Friday Voice
npm install

Running Locally

Three processes are used during development.

Terminal 1 --- Express backend

cd C:\Users\sukan\friday\backend
npm run dev

Expected:

Server running on http://localhost:5000

Terminal 2 --- Python LiveKit agent

cd C:\Users\sukan\friday\backend
.\.venv\Scripts\activate
python agent.py dev

Newer LiveKit CLI versions may recommend:

lk agent dev

Terminal 3 --- Frontend

cd C:\Users\sukan\friday\frontend\Friday Voice
npm run dev

Open the Vite URL shown in the terminal.

For testing, use one browser tab/session at a time.

Frontend ↔ Backend Connection

The browser does not receive the LiveKit API secret.

Browser
   │
   │ GET /api/livekit/token
   ▼
Express backend
   │
   │ signs token using LIVEKIT_API_KEY
   │ + LIVEKIT_API_SECRET
   ▼
Temporary LiveKit token
   │
   ▼
Browser
   │
   ▼
LiveKit Cloud

The frontend requests:

GET /api/livekit/token?room=<room-name>

The backend returns:

{
  "token": "...",
  "url": "...",
  "room": "..."
}

This keeps the LiveKit signing secret server-side.

Failure Behavior

Friday Voice is a multi-provider realtime system, so failures occur at
specific pipeline stages.

LiveKit failure

If LiveKit connection fails, the frontend cannot establish the realtime
voice session and reports a disconnected/error state.

Token-generation failure

If LiveKit credentials are missing or token generation fails, Express
returns an HTTP 500 response and the browser cannot connect.

Deepgram/STT failure

If STT is unavailable, the agent cannot obtain a reliable user
transcript, so an LLM response cannot safely be generated.

Groq/LLM rate limit

A Groq 429 Too Many Requests can occur when the applicable quota/rate
limit is exhausted.

The agent retries according to its configured retry behavior. If the LLM
error remains unrecoverable, the current LiveKit agent session can close
without producing a response.

This can result in:

STT transcript succeeds
        ↓
Groq request fails
        ↓
No LLM response
        ↓
No TTS response

A Groq quota problem is therefore different from a microphone, STT, or
Rime problem.

Rime/TTS failure

If Rime fails after the LLM generates text, the response may exist
internally as text but synthesized voice will not reach the user.

Known Limitations

Provider quotas: External API limits can interrupt an otherwise
correct implementation.

Internet dependency: LiveKit, Deepgram, Groq, and Rime require
network connectivity.

Hinglish variability: Hinglish has no single standardized
grammar; speakers mix languages differently.

Rime language setting: The current configuration explicitly uses
lang="eng". Mixed Hindi-English pronunciation must therefore be
validated against real application utterances; this setting is not a
guarantee of perfect pronunciation for every mixed-script input.

Pronunciation: Names, acronyms, product names, numbers, and
codes may require additional normalization.

Probabilistic LLM: Prompt constraints reduce unwanted behavior
but do not make responses deterministic.

Scale: This is a hackathon/engineering prototype, not a
demonstrated high-concurrency production system.

Voice-quality study: No statistically significant MOS study has
been completed.

Load testing: Large-scale concurrent-user, packet-loss,
long-duration, and provider-outage testing remains future work.

Provider failover: Automatic fallback across all AI providers is
not currently implemented.

Security

The repository ignores environment files:

.env
.env.*
!.env.example

Never commit:

LIVEKIT_API_SECRET
GROQ_API_KEY
DEEPGRAM_API_KEY
RIME_API_KEY

The frontend should receive temporary access tokens only, never provider
secrets.

If a credential is accidentally exposed, revoke/rotate it immediately.

Testing and Acceptance Evidence

The repository includes:

RIME_EVIDENCE.md

It records:

exact Rime model and speaker

transport

audio settings

test procedure

observed behavior

acceptance criteria

limitations

reproduction information

This provides reproducible engineering evidence instead of relying only
on a demo screenshot.

Development Commands

Backend:

cd backend
npm run dev

Python agent:

cd backend
.\.venv\Scripts\activate
python agent.py dev

Frontend:

cd frontend\Friday Voice
npm run dev

Production frontend build:

npm run build

Hackathon Demo Flow

Start Express.

Start the Python LiveKit agent.

Start the Vite frontend.

Connect the voice session.

Say:

"Mera phone mein network nahi aa raha."

Follow with:

"I already restarted it but still not working."

Interrupt the agent while it is speaking.

Demonstrate telemetry/evaluation views.

Explain the STT → LLM → TTS pipeline and exact Rime configuration.

The strongest demonstration is the realtime conversational behavior, not
only the UI.

Roadmap

Production deployment

Explicit Hindi/English language-switch detection

Better multilingual TTS routing

Persistent conversation memory

Customer-support knowledge base / RAG

CRM/ticketing integrations

Automatic evaluation datasets

Formal latency benchmarks

Human voice-quality evaluation

Provider fallback/recovery

Large-scale concurrency testing

Advanced observability and tracing

Authentication and user identity

Engineering Position

Friday Voice is built around one principle:

The voice agent should adapt to how people naturally speak, rather
than forcing people to adapt to the system.

The architecture separates transport, speech recognition, reasoning, and
speech synthesis so each provider can be replaced without redesigning
the complete realtime interaction.

Realtime Transport
       │
       ▼
    LiveKit
       │
       ├──────────────┐
       ▼              ▼
      STT            LLM
   Deepgram          Groq
       │              │
       └──────┬───────┘
              ▼
             TTS
             Rime
              │
              ▼
        LiveKit Audio

Official Documentation

LiveKit Agents: https://docs.livekit.io/agents/

LiveKit Rime integration:
https://docs.livekit.io/agents/models/tts/rime/

LiveKit Rime Python reference:
https://docs.livekit.io/reference/python/livekit/plugins/rime/

Deepgram: https://deepgram.com/

Groq: https://groq.com/

React: https://react.dev/

Vite: https://vite.dev/

Repository

https://github.com/Abhishek-ias/friday-multi-voiceagent

License

Add the project's intended open-source license before public release.
