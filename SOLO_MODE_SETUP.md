# Internet Sucks — Solo Mode AI prototype

This branch (solo-mode-ai-starter) adds a first Solo Mode prototype: one human host and two AI competitors. It deliberately does not deploy to GitHub Pages or modify the production main branch.

## Architecture

- index.html remains the existing client and game UI.
- server.ts is a separate Node/Express endpoint for /api/ai-turn and serves the same index.html when this branch is published as an AI Studio full-stack app.
- Gemini calls happen only in the server runtime. The API key is read from GEMINI_API_KEY; it is not embedded in the browser.
- The existing Firebase Realtime Database remains the room state store. The human host's authenticated browser writes bot actions to the room using its existing host authority, so this prototype does not require Firebase Admin credentials or Cloud Functions.
- A small in-memory rate limiter/action cache and curated fallback responses help avoid duplicate generation and prevent a phase from getting stuck when Gemini is unavailable.

## Run / deploy using Google AI Studio Starter Tier

1. In Google AI Studio Build mode, import the repository and select branch solo-mode-ai-starter.
2. Ask Build to preserve the existing single-page index.html, use server.ts as the Node entry point, install dependencies from package.json, and run npm start.
3. Preview the app and check /api/health on the same origin. Confirm the app page is served by the Node runtime rather than GitHub Pages. The server uses FIREBASE_PROJECT_ID=internet-sucks and FIREBASE_DATABASE_URL=https://internet-sucks-default-rtdb.firebaseio.com by default; override these only if you intentionally use another database.
4. In AI Studio Settings / Secrets, verify that GEMINI_API_KEY is available to the server runtime. Do not put it in index.html or a client .env variable.
5. Test Solo Mode in preview first. If it works, publish the app using the Starter Tier and use the resulting AI Studio / Cloud Run app URL for Solo Mode.
6. The GitHub Pages URL remains the existing multiplayer build until you intentionally decide whether to move or merge the feature later.

The Solo Mode button relies on the same-origin /api/ai-turn endpoint. It will not call an unknown separate URL from GitHub Pages; use the AI Studio-published app URL for the prototype.

## Security and limits

- The server validates theme IDs, phase names, bot IDs, match IDs, payload size, and the current round state.
- Each AI turn requires a Firebase ID token. The server uses that token to read the room through the existing Realtime Database Rules, then confirms the caller is the connected host of that Solo match and the requested AI action is still due. The Gemini prompt is built from live database state, not from the client-supplied copy.
- Same-origin requests are also required. This blocks normal cross-origin browser calls, but does not alone protect an endpoint from direct HTTP clients.
- Rate limits and the action cache are in-memory and reset when the service restarts or uses another instance. This is suitable for a small prototype, not robust public abuse prevention.
- The backend generates responses but does not write to Firebase. The browser host does the writes. A malicious client controlling their own host room could still manipulate that room; this prototype is not a competitive anti-cheat system.
- Gemini quota/rate-limit errors fall back to theme-specific canned lines so a Solo match can continue. Responses from the fallback path show a subtle toast.
- The GEMINI_MODEL environment variable can override the default gemini-3.8-flash model.

## Firebase Rules

This branch adjusts photo validation to compare a selected photo against the submitting player's options (photoOptions/$player), required when the host writes a bot's Round 4 selection or a host-generated skipped photo. Before testing the photo finale against the existing Firebase project, publish this branch's database.rules.json in Firebase Console → Realtime Database → Rules. This changes the shared project's rules for production rooms too, so review the whole JSON before publishing. GitHub does not publish RTDB rules automatically.

## Known limitations

- Cloud Run / Starter Tier deployment has not been performed from this repository. It must be completed interactively in your Google AI Studio account.
- The in-memory cache does not survive restarts or span multiple instances.
- The endpoint verifies the caller's Firebase identity indirectly through a rules-protected Realtime Database read and confirms host/room/phase state. For a larger public game, add shared rate limiting, monitoring, and a more durable idempotency store.
- Solo mode is implemented on this branch but is not live on is9coding.github.io/internet_sucks.
