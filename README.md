<div align="center">

# UniTalks — Real-Time WebRTC Video Chat & Interactive Chess Platform

**A full-stack, peer-to-peer communication platform featuring custom queue-based matchmaking, real-time video/audio/text chat, and synchronized in-chat chess gameplay.**

![React](https://img.shields.io/badge/React-18.2-3B82F6?style=flat-square&logo=react&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.3-4F46E5?style=flat-square&logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-20.x-10B981?style=flat-square&logo=node.js&logoColor=white)
![WebRTC](https://img.shields.io/badge/WebRTC-P2P-F59E0B?style=flat-square&logo=webrtc&logoColor=white)
![Socket.io](https://img.shields.io/badge/Socket.io-4.7-7C3AED?style=flat-square&logo=socketdotio&logoColor=white)
![Styled Components](https://img.shields.io/badge/Styled--Components-6.1-DB2777?style=flat-square&logo=styledcomponents&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-10B981?style=flat-square)

</div>

---

## The Problem

**Standard chat applications lack interactive engagement.** Typical video chat platforms (like Omegle or traditional chatrooms) limit interactions to just streaming audio/video and text chat. This can result in brief, passive, or awkward encounters. Creating a richer, active shared experience — such as playing a board game together in real-time — requires low-latency state synchronization alongside the media streams.

**Handling media streams, signaling, and game state simultaneously is complex.** Standard WebSockets are great for text and signaling, but routing high-bandwidth audio/video through a central server leads to severe bottlenecks and high infrastructure costs. A hybrid peer-to-peer (P2P) architecture is needed: routing heavy media and game-control frames directly between peers (via WebRTC Data Channels), while using a lightweight signaling server for matchmaking and connection negotiation.

---

## What This Does

**UniTalks** is a full-stack real-time video chat and gaming platform. It matches users dynamically via a centralized matchmaking queue and establishes a direct peer-to-peer WebRTC connection. Once connected, users can video chat, send text messages, and play an interactive game of chess on a shared board synchronized directly over WebRTC Data Channels.

- **Dynamic Matchmaking Queue** — Server-side queueing system (`matchmaking.ts`) that matches online users and assigns them to private rooms
- **Peer-to-Peer WebRTC Media** — Low-latency direct audio and video streams using `simple-peer` to bypass centralized server relays
- **Synchronized In-Chat Chess** — A built-in chessboard powered by `chess.js` that synchronizes moves, turns, and game state directly between players over P2P data channels
- **Robust Room & Game State Manager** — Backend state tracker (`stateManager.ts`) monitoring active connections, signal handshakes, and game states
- **Real-Time Audio Visualizer** — Visual canvas rendering active audio frequency feedback to indicate microphone status
- **Modern Styled Interface** — Beautiful, premium dark mode UI built with `styled-components`

---

## System Architecture

```mermaid
graph TD
    subgraph ClientA["Client A (React Frontend)"]
        UI_A["UI Components<br/>Video / Audio / Text Chat"]
        CHESS_A["ChessBoard Component<br/>chess.js Engine"]
        SP_A["Simple-Peer (WebRTC)"]
        SIO_A["Socket.IO Client"]
    end

    subgraph ClientB["Client B (React Frontend)"]
        UI_B["UI Components<br/>Video / Audio / Text Chat"]
        CHESS_B["ChessBoard Component<br/>chess.js Engine"]
        SP_B["Simple-Peer (WebRTC)"]
        SIO_B["Socket.IO Client"]
    end

    subgraph Server["Signaling & Matchmaking Server (Node.js + TS)"]
        WS["WebSocket Server (ws / Express)"]
        MM["Matchmaking Engine<br/>matchmaking.ts (Queue-based)"]
        SM["State Manager<br/>stateManager.ts (Room & Game State)"]
    end

    subgraph STUN["NAT Traversal"]
        STUN_SRV["STUN/TURN Server"]
    end

    %% Signaling phase
    SIO_A <-->|1. Sign-in & Match Request| WS
    SIO_B <-->|1. Sign-in & Match Request| WS
    WS <--> MM
    MM -->|2. Create Room & Pair Peers| SM
    SM -->|3. Signaling: SDP Offer/Answer| WS
    WS <--> SIO_A
    WS <--> SIO_B

    %% ICE Candidate Negotiation
    SP_A <-->|4. Get ICE Candidates| STUN_SRV
    SP_B <-->|4. Get ICE Candidates| STUN_SRV
    SP_A <-->|5. Exchange Candidates via WS| WS
    SP_B <-->|5. Exchange Candidates via WS| WS

    %% P2P WebRTC Connection
    SP_A ====|6. P2P Direct WebRTC Stream (Video/Audio/Data)| SP_B
    CHESS_A <==>|7. Synchronize Game State via P2P Data Channel| CHESS_B

    style ClientA fill:#1e1b4b,stroke:#4F46E5,color:#e0e7ff
    style ClientB fill:#1e1b4b,stroke:#4F46E5,color:#e0e7ff
    style Server fill:#1e1b4b,stroke:#7C3AED,color:#e0e7ff
    style STUN fill:#1e1b4b,stroke:#F59E0B,color:#e0e7ff
```

---

## Tech Stack

| Layer | Technology | Role / Usage |
|:---|:---|:---|
| **Frontend Framework** | React 18.2 | Component-based interactive UI, SPA routing |
| **Styling** | Styled Components 6.1 | Scoped CSS-in-JS design system with premium dark theme |
| **Real-Time signaling** | Socket.IO Client 4.7 | P2P negotiation, match notifications, room signals |
| **P2P WebRTC** | Simple-Peer 9.11 | High-level abstraction wrapper for WebRTC streams & data channels |
| **Game Engine** | Chess.js 1.4 | Chess rules validation, turn tracking, and move verification |
| **Backend Language** | Node.js + TypeScript 5.3 | Type-safe server logic and state operations |
| **Backend Framework** | Express 4.18 | Static file serving and HTTP utilities |
| **WebSockets** | ws 8.17 | High-performance WebSocket server handling raw signaling |
| **Authentication** | JSON Web Tokens (JWT) 9.0 | Session tokens for secure matchmaking requests |

---

## Key Features

### 🤝 Queue-Based Matchmaking
The backend implements a matching queue (`matchmaking.ts`). When a user enters the queue, the matchmaking engine checks for compatible available peers. Once a pair is matched, a unique `roomId` is generated, and both clients are notified to start the WebRTC signaling handshake.

### 🎮 P2P Synchronized Chess
Instead of routing chess moves to the server and back to the opponent, UniTalks uses a **WebRTC Data Channel**. Moves made on the custom `ChessBoard.js` interface are validated locally via `chess.js` and immediately serialized and sent directly to the matched peer. This results in zero-latency gameplay.

### 🔊 Real-Time Audio Visualizer
The `AudioVisualizer.js` component captures the user's local microphone stream using the browser's `AudioContext` and `AnalyserNode`. It converts frequency data into a dynamic HTML5 Canvas animation, providing immediate visual feedback of active voice communication.

---

## Getting Started

### Prerequisites
- Node.js (v18.x or later)
- npm or yarn

### Installation & Local Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Hazz-Y/unitalks-video-chat-1.git
   cd unitalks-video-chat-1
   ```

2. **Setup the Signaling Server (Backend):**
   ```bash
   cd server
   npm install
   
   # Create a .env file and set PORT (e.g. PORT=5000) and JWT_SECRET
   echo "PORT=5000" > .env
   echo "JWT_SECRET=super_secret_key" >> .env
   
   # Start the backend in development mode
   npm run dev
   ```

3. **Setup the React App (Frontend):**
   Open a new terminal window in the root directory:
   ```bash
   npm install
   
   # Start the development server
   npm start
   ```
   The application will open automatically at `http://localhost:3000`.

---

## Project Structure

```
unitalks-video-chat-1/
├── server/                      # TypeScript Backend (Signaling Server)
│   ├── src/
│   │   ├── types/               # Type definitions
│   │   ├── utils/               # Logger & Helpers
│   │   ├── index.ts             # Express & WebSocket entry point
│   │   ├── matchmaking.ts       # Queue-based matchmaking engine
│   │   └── stateManager.ts      # Room & chess game state management
│   ├── package.json
│   └── tsconfig.json
│
├── src/                         # React Frontend
│   ├── components/
│   │   ├── layout/              # Header & Footer
│   │   ├── pages/               # Homepage, VideoChat, AudioChat, TextChat
│   │   └── ui/                  # AudioVisualizer, ChessBoard, HamburgerMenu
│   ├── config/
│   │   └── theme.js             # Shared styled-components theme config
│   ├── utils/
│   │   ├── chessEngine.js       # Game move sync logic
│   │   ├── socketService.js     # WebSocket signaling client
│   │   └── webrtcStun.js        # STUN/TURN configurations
│   ├── App.js                   # Application Router & Entry
│   └── index.js                 # DOM target mount
│
└── package.json
```

---

## License

This project is licensed under the [MIT License](LICENSE).
