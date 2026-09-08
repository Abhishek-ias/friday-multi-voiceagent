Friday — Multilingual Voice AI Agent

A real-time voice-native customer support agent built for natural Hindi-English (Hinglish) conversations.

Friday is a low-latency conversational AI agent designed for Indian users. It understands natural code-switching between Hindi and English and responds with a consistent Indian voice.

🎯 Problem

Traditional voice assistants often struggle with real-world Indian conversations.

Users naturally switch between languages:

"Mera order abhi tak deliver nahi hua, can you check?"

A voice agent must understand the language switch without forcing the user to manually select a language.

Friday is designed specifically for this interaction pattern.

💡 Solution

Friday provides a real-time conversational pipeline:

User Speech
     ↓
LiveKit
     ↓
Deepgram Nova-3
     ↓
Multilingual STT
     ↓
LLM
     ↓
Response Generation
     ↓
Rime TTS
     ↓
Indian Voice Response
     ↓
User

The system is optimized for:

Hindi-English code-switching

Low-latency conversations

Voice interruption

Natural turn-taking

Consistent voice identity

Short conversational responses

✨ Key Features

🗣️ Natural Hinglish

Friday can handle conversations where Hindi and English are mixed naturally.

Example:

User:

"Mera refund abhi tak nahi aaya, can you check?"

Friday:

"Sure, main aapka refund status check karta hoon."

⚡ Real-Time Voice

LiveKit provides the real-time communication layer between the browser and the voice agent.

🎙️ Multilingual Speech Recognition

Deepgram Nova-3 is used for speech-to-text with multilingual recognition.

🔊 Consistent Voice

Rime TTS generates the agent's spoken responses with an Indian voice.

🧠 Conversational LLM

The language model generates short, conversational responses optimized for voice interaction.

✋ Interruption Handling

Users can interrupt the agent naturally while it is speaking.

📊 Evaluation & Telemetry

The frontend includes workflow and evaluation views for observing conversation behavior and system performance.

🏗️ Architecture

                    ┌──────────────────┐
                    │      User        │
                    │   Microphone     │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │     Frontend     │
                    │ React + Vite     │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │     LiveKit      │
                    │ Realtime Audio   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │  Python Agent    │
                    │ LiveKit Agents   │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Deepgram │   │   LLM    │   │   Rime   │
        │   STT    │   │   Groq   │   │   TTS    │
        └──────────┘   └──────────┘   └──────────┘
              │              │              │
              └──────────────┼──────────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Spoken Response  │
                    └──────────────────┘

🛠️ Tech Stack

Frontend

React

TypeScript

Vite

LiveKit Client

Lucide React

Backend

Node.js

Express

LiveKit Server SDK

Voice Agent

Python

LiveKit Agents

Deepgram Nova-3

Rime TTS

Groq / OpenAI-compatible LLM endpoint

Silero VAD

📁 Project Structure

friday/
│
├── frontend/
│   └── friday-voice/
│       ├── src/
│       ├── public/
│       ├── package.json
│       └── ...
│
├── backend/
│   ├── agent.py
│   ├── server.js
│   ├── package.json
│   ├── requirements.txt
│   └── ...
│
├── .gitignore
└── README.md

⚙️ Setup

1. Clone the Repository

git clone https://github.com/Abhishek-ias/friday-multi-voiceagent.git
cd friday-multi-voiceagent

2. Backend Setup

cd backend
npm install

Create:

backend/.env

Add your local credentials:

LIVEKIT_URL=your_livekit_url
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_api_secret

DEEPGRAM_API_KEY=your_deepgram_api_key
RIME_API_KEY=your_rime_api_key

GROQ_API_KEY=your_groq_api_key
OPENAI_API_KEY=your_openai_api_key
GEMINI_API_KEY=your_gemini_api_key

Important: Never commit .env files or real API keys to GitHub.

3. Start the Backend Server

npm run dev

The backend server runs on:

http://localhost:5000

4. Start the Voice Agent

Open another terminal:

cd backend

Activate the Python virtual environment:

Windows:

.\.venv\Scripts\activate

Start the agent:

python agent.py dev

5. Start the Frontend

Open another terminal:

cd frontend/friday-voice
npm install
npm run dev

Open the Vite URL shown in the terminal.

🎤 Example Conversation

User

"Mera order kab tak aayega?"

Friday

"Sure, main aapka order status check karta hoon."

User

"Actually mujhe refund chahiye because product damaged tha."

Friday

"Sure, main refund process ke liye help karta hoon."

The important part is that the user does not need to manually switch between Hindi and English.

⚡ Latency Optimization

Friday uses several techniques to improve conversational responsiveness:

Streaming speech recognition

Voice activity detection

Early endpoint detection

Preemptive response generation

Streaming TTS

Short LLM responses

LiveKit real-time audio transport

Interruption detection

The goal is to make the interaction feel like a conversation rather than a request-response API.

🔐 Security

API credentials are stored locally in .env files.

The repository only contains .env.example files with placeholders.

Secrets should never be committed to Git.

🚀 Future Improvements

Better Hindi-English language-switch detection

More robust interruption recovery

Conversation memory

Customer/order API integration

Production deployment

Additional Indian languages

Advanced latency monitoring

Voice quality optimization

👥 Team

Built as a hackathon project focused on real-time multilingual voice interaction for Indian users.

📜 License

This project is intended for hackathon and educational use.
