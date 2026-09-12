# Realtime Multiplayer Quiz Platform

A full-stack, realtime multiplayer quiz application. Hosts create and run live quiz sessions; players join via a room code and compete on a shared, timed leaderboard.

## Stack

- **Backend:** Go, gorilla/websocket, stdlib net/http (or chi)
- **Frontend:** React + Vite, Tailwind CSS + DaisyUI (theme: abyss)
- **Database:** PostgreSQL (Supabase)
- **Auth:** Email/password (bcrypt/argon2), JWT sessions for hosts

## Fonts

- Headings/UI — Space Grotesk
- Body — Inter
- Timers, scores, room codes — JetBrains Mono

## Architecture

- REST API handles authoring: host auth, quiz CRUD, question CRUD.
- WebSocket layer handles realtime session events only.
- Each active session runs its own goroutine-managed hub, isolating state and broadcast per session.
- A single dedicated goroutine batches answer writes to Postgres across all sessions, reached via a shared channel.
- DB writes retry with backoff on transient failure.

## Data Model

- **Quizzes** — host-owned, ordered list of questions.
- **Questions** — options, correct answer, time limit.
- **Sessions** — a live instance of a quiz; room code, status (waiting/active/ended).
- **Session Questions** — a question as played within a session; status (pending/active/completed/invalidated).
- **Participants** — tied to a session; public ID (display/leaderboard) and private ID (auth/reconnect), score, connection status.
- **Answers** — per participant, per question, server-timestamped for speed-bonus scoring.

## Participant Identity

Each participant receives two identifiers on join:
- **Private ID** — a random UUID, stored client-side, used to authenticate reconnects and answer submissions.
- **Public ID** — shown to other players on the leaderboard.

## Host Flow

Sign up, log in, create a quiz, start a live session, control question flow, and view live participant counts and answers per question.

## Player Flow

Join via room code and display name, wait for the host to start, answer each question within its time limit, see immediate feedback and running leaderboard position, and view the final leaderboard at session end.

## Scoring

Speed-bonus scoring: correct answers submitted faster earn more points, computed from the server-side answer timestamp.

## Resilience

Each session question carries a status field, allowing a host to mark a question invalid without deleting recorded answers — used to handle interruptions mid-question.

## Setup

```bash
docker compose up --build
```

Backend runs on `:8080`, frontend on `:3000`.