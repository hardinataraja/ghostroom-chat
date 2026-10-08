# GhostRoom
Ephemeral P2P chat. No account, no database, no API key. Static files + one Vercel serverless function (`api/signal.js`).

## Deploy
1. Extract ZIP  2. Push to GitHub  3. Import repo in Vercel (no build command, no settings)  4. Deploy  5. Open the domain.

## Use
Create: CREATE ROOM → codename → lifetime → GENERATE → copy/share link/QR → ENTER. Join: open link (or enter code) → codename → CONNECT.

## Architecture
Browser A ⇄ `/api/signal` (offer/answer/ICE only) ⇄ Browser B, then Browser A ⇄ WebRTC DataChannel ⇄ Browser B. Chat text never goes through the server. Mesh topology: best for 2–5 users, server cap 10.

## Important technical limitation (architecture decision)
Vercel functions are stateless; signaling uses **in-memory state** (no database allowed). Both peers' requests must reach the same warm instance. This works for typical low traffic, but cold starts/scale-out can drop a room ("ROOM NOT FOUND" or stuck WAITING); the client auto-recreates and retries. For guaranteed reliability, add a tiny shared store (e.g. Upstash Redis) later. Not tested against a live Vercel deployment.
No TURN server is provided (needs credentials): users behind strict/symmetric NAT or some mobile carriers may fail to connect. Only public STUN is used.

## Privacy
No accounts or analytics. Messages are stored only in each browser's localStorage until they expire (and cleared on leave). IPs are visible to peers/STUN/network. Not absolute anonymity. Encryption is standard WebRTC DTLS only. QR code uses a cdnjs library (needs internet); everything else works without third-party scripts.
