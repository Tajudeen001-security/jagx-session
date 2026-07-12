# JagX Session Generator

A tiny website that pairs your WhatsApp number and hands you back a portable **session ID** — paste it into your JagX bot's `.env` and it connects automatically, no QR scan or pairing code needed on the bot's side ever again.

## How it works

1. You enter your phone number and hit Connect.
2. The server starts a real WhatsApp connection and requests a pairing code, which shows on the page immediately.
3. You enter that code in WhatsApp → Linked Devices → Link with phone number, same as always.
4. The moment it connects, the server packages the session files into one compressed, base64 string (prefixed `JAGX~`) and streams it back to your browser. **Nothing is saved on the server** — the session ID that lands in your browser is the only copy.
5. You also get a WhatsApp message to yourself confirming the connection, so you know it's really your account.
6. Paste that string into your bot's `.env` as `SESSION_ID` and start the bot — it decodes it back into a working session automatically.

## ⚠️ Security — read this

The session ID **is** full access to that WhatsApp account — same as the QR code or the `session/` folder the bot normally creates. Anyone who has it can read and send messages as you. Treat it exactly like a password:

- Don't paste it in a public chat, a GitHub repo, or send it to anyone.
- Only put it in your own bot's `.env`, which should never be committed to git (already covered by `.gitignore` in the bot project).
- If you think it leaked, open WhatsApp → Linked Devices and log out the device immediately — that invalidates the session ID.

## Deploy to Vercel

```bash
npm install -g vercel
cd jagx-session
vercel
```

Follow the prompts (link/create a project, accept defaults). Vercel will detect the `/api` folder and `/public` folder automatically. Once deployed, visit the URL it gives you.

**A real limitation to know about:** Vercel serverless functions have a maximum run time. This project requests `maxDuration: 120` seconds in `vercel.json`, but on Vercel's **free Hobby plan the actual cap is lower (around 60 seconds)** — a Pro plan is needed for the full 120s window. If you find the connection times out while you're still typing the code in WhatsApp, either:
- Upgrade the Vercel project to Pro, or
- Deploy the exact same code to **Render.com** or **Railway** instead — both run this as a normal always-on Node server with no such time limit, and the code needs zero changes (just `npm install && npm start` isn't set up here since it's serverless-shaped, but a one-file Express wrapper takes minutes — ask if you'd like that version).

## Local testing

```bash
npm install -g vercel
cd jagx-session
npm install
vercel dev
```

Then open the local URL it prints.

## Files

- `public/index.html` — the page itself (form, live status log, code display, copy-to-clipboard session box)
- `api/pair.js` — the serverless function that does the actual WhatsApp pairing and session packaging
- `vercel.json` — extends the function's allowed run time
