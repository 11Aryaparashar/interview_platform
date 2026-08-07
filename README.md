# HireSensei.ai - AI Interview Platform

HireSensei.ai is an automated technical interviewing platform. It reviews a candidate's GitHub profile, conducts a live voice interview, and provides automated grading.

## Unique Selling Proposition (USP)

The core differentiator of this platform is its server-side "sideband" websocket architecture.

### Key Differentiators
- **Security:** Interview logic and prompts are stored securely on the backend, preventing client-side manipulation.
- **Stateful Evaluation:** Audio and conversation data are processed through the backend for real-time monitoring and grading.
- **Advanced Voice Integration:** Uses WebRTC with OpenAI's Realtime API for fast, human-like voice interaction, while recording the session.
- **Context-Aware Interviews:** Scrapes external data (like GitHub profiles) to generate personalized questions for the candidate.

## System Architecture

```mermaid
graph TD
    subgraph Frontend
        React[React UI]
        Mic[Microphone]
    end
    
    subgraph Backend
        Express[Express Server]
        WSS[WebSocket Handler]
    end
    
    subgraph Data
        DB[(PostgreSQL)]
    end
    
    subgraph External APIs
        OpenAI[OpenAI Realtime API]
        Gemini[Google Gemini API]
        Deepgram[Deepgram API]
        GitHub[GitHub API]
    end
    
    React <--> Express
    React --> Deepgram
    Mic --> React
    
    Express --> DB
    Express --> GitHub
    Express --> Gemini
    
    WSS <--> OpenAI
    WSS --> DB
    
    React <-->|WebRTC| OpenAI
```

### Components
1. **Frontend (apps/frontend):** React app that captures microphone audio. It uses WebRTC to stream audio to OpenAI and WebSockets to stream audio to Deepgram for live transcription.
2. **Backend (apps/backend):** Express server running on Bun. It manages interview state, scrapes GitHub data, handles WebRTC handshakes, and evaluates interviews using Google Gemini.
3. **Database (Prisma + PostgreSQL):** Stores interview metadata, transcripts, and final grades.

## End-to-End Flow

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant B as Backend
    participant DB as Database
    participant API as External APIs

    Note over U,API: Pre-Interview
    U->>F: Submits GitHub URL
    F->>B: POST /api/v1/pre-interview
    B->>API: Fetch GitHub Repos
    B->>DB: Create Interview Record
    B-->>F: Return Interview ID
    
    Note over U,API: Live Interview
    U->>F: Starts Interview
    F->>B: SDP Offer
    B->>API: Forward SDP to OpenAI
    API-->>B: SDP Answer
    B-->>F: Connect WebRTC
    
    par Audio Stream
        U->>F: Speaks
        F->>API: WebRTC Audio (OpenAI)
        F->>API: WebSocket Audio (Deepgram)
        API-->>F: Live Transcript
        F->>B: Save User Transcript
        API-->>B: WebSocket 'response.done' (AI Transcript)
        B->>DB: Save Transcripts
    end

    Note over U,API: Evaluation
    U->>F: Ends Interview
    F->>B: GET /api/v1/result/:id
    B->>API: Evaluate Transcript (Gemini)
    API-->>B: Score & Feedback
    B->>DB: Save Results
    B-->>F: Display Results to User
```

## Getting Started

1. **Install Dependencies**
   ```bash
   bun install
   ```

2. **Configuration**
   Create a `.env` file in `apps/backend` with:
   - `DATABASE_URL`
   - `OPENAI_KEY`
   - `GEMINI_API_KEY`
   - `PROXY_URL` (optional)

3. **Database Setup**
   ```bash
   cd apps/backend
   bunx prisma db push
   ```

4. **Run Application**
   ```bash
   bun run dev
   ```