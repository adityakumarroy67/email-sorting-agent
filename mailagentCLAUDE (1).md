# Project: AI Email Triage & Voice Approval Agent (Demo)

This file gives you (Claude Code) the full context for this project. Read it fully before doing anything.

---

## 1. Who you are working with

- The user is a co-founder of **Midnight Softwares** (midnightsoftwares.in), a new B2B SaaS / tech-solutions agency.
- **The user does not know how to code yet** (no Python or JavaScript experience). They are learning as they build.
- This project is a **portfolio demo** to show on the agency website and to prospective clients.

### How to work with this user
- Explain what you are doing in simple, plain English before each step. Avoid jargon, or explain it briefly when you must use it.
- Work **one phase at a time**. Do not jump ahead. At the end of each phase, stop, summarise what was built, and tell the user exactly how to test it.
- When something needs the user to act manually (creating an account, clicking in a browser, copying a key, approving Google permissions), **stop and give numbered, click-by-click instructions**, then wait for them to confirm.
- After finishing each phase, give a short "What you learned" section (3–5 lines) explaining the key concepts in the code, so the user builds real understanding.
- Ask before running anything destructive (deleting files, overwriting work, removing triggers).
- Keep code clean, well-commented, and beginner-readable. Prefer simple over clever.

---

## 2. HARD CONSTRAINT: zero budget

**The user cannot spend any money.** Every tool, service and API used must be free.
- Never suggest a paid plan, paid API, paid hosting, or anything requiring a credit card.
- If a free tier has limits, design around them (batching, caching, fewer calls).
- Free-tier rules, model names and limits change often. **Check current official documentation** before relying on a specific model name, quota, or API feature. Do not hardcode assumptions from memory.
- Only fictional demo data goes through free AI tiers (free tiers may use data to improve products). Never real client emails.

---

## 3. The problem being solved

Inspired by a real-world case (a UK construction company). This is our **own original build**, not a copy of anyone else's code, design, or branding.

The situation:
- A UK construction company has **two email inboxes** and constant chaos between **directors, accounts, and an ops manager** who is juggling live job updates.
- Nothing is organised. Urgent emails get buried. Everyone wastes time working out who should even be looking at what.
- Key people are often **on building sites**, not at a laptop.

What the agent must do:
1. **Read** incoming emails automatically.
2. **Sort by priority** (urgent / normal / low).
3. **Route** each email to the right person (director, accounts, ops manager).
4. **Draft replies**, adjusting tone depending on who will be responding.
5. **Instant alerts** for urgent emails to the right person's phone.
6. **Hands-free voice approval**: the person can listen to a summary / draft and approve or edit it by voice, useful on site.

---

## 4. What makes our version better (build these in)

- **Explained decisions**: every routing decision includes a one-line reason, e.g. "Sent to Ops Manager: mentions Site 14 and a delivery tomorrow."
- **Human always approves**: nothing is ever sent without explicit approval. No auto-sending.
- **Correction loop**: if the user says an email was misrouted, store that correction and include recent corrections as examples in future AI prompts so routing improves.
- **On-site briefing**: an on-demand (and optional scheduled) summary: what's urgent, what's awaiting approval, what can wait.
- **Security from the start**: API keys in Script Properties / environment variables only, never in code or git. Minimal permissions.
- **Safe voice approval**: require a clear confirmation step before sending (e.g. bot replies "Send this reply to X? Say or type YES to confirm"). Ignore unclear voice commands and ask again.

---

## 5. Fictional demo company

Use a fictional UK company for all demo data, e.g. **"Brickline Construction Ltd"** (feel free to refine the name with the user).

Team (fictional):
| Role | Handles | Tone for drafted replies |
|---|---|---|
| Managing Director | Contracts, big clients, disputes, escalations | Formal, concise, authoritative |
| Operations Director | Strategy, major suppliers, planning | Professional, direct |
| Accounts | Invoices, payments, statements, credit control | Polite, precise, numbers-focused |
| Ops Manager | Live jobs, site issues, deliveries, subcontractors, safety | Friendly, short, practical |

Priority rules (starting point, refine with the user):
- **Urgent**: site safety, same/next-day delivery problems, work stoppages, client complaints, payment disputes, anything with a same-day deadline.
- **Normal**: quotes, scheduling, routine client/supplier questions.
- **Low**: newsletters, marketing, automated notifications.

Create a set of **20–30 realistic sample emails** (UK English, construction context: sites, deliveries, subcontractors, invoices, snagging, H&S, planning) for testing and for the website demo.

---

## 6. Tech stack (all free)

| Part | Tool |
|---|---|
| Email reading, labelling, drafts | **Google Apps Script** with `GmailApp`, on a dedicated **test Gmail account** (never a personal or company inbox) |
| Local development of Apps Script | **clasp** (`@google/clasp`, free Google CLI) so code lives in this folder and is pushed to Apps Script |
| AI (classify, route, draft, understand voice) | **Google Gemini API free tier** (key from aistudio.google.com). Gemini can take audio input directly, so it can also understand voice notes. |
| Alerts + voice approvals | **Telegram Bot** (free, created via @BotFather) |
| Scheduling | Apps Script **time-driven triggers** (minimum every 1 minute) |
| Storage (settings, corrections, state) | Apps Script **PropertiesService**, and a Google Sheet as a simple log/database if useful |
| Website demo | Small web app deployed on **Vercel free (Hobby) plan**, with the Gemini key stored as a Vercel environment variable |

Notes:
- The original case has two inboxes. For the demo, start with **one test inbox**; optionally add a second test Gmail later to show multi-inbox support.
- **Telegram updates**: prefer **polling with `getUpdates`** from a 1-minute time trigger (store the last `update_id` offset in PropertiesService). Apps Script web apps return redirects that can make Telegram webhooks retry and duplicate messages. Verify before choosing webhooks.
- In the demo, all roles can map to the user's own Telegram chat (with the role shown in each message), with the config structured so each role could have its own chat ID in a real deployment.
- **Voice output (audio summaries)** is a stretch goal. Start with text alerts. Then check current Gemini docs for a free text-to-speech option and what audio format Telegram accepts (`sendVoice` expects OGG/Opus, `sendAudio` accepts MP3/M4A). If there's no clean free path, keep voice *input* (approving by voice note) and text output, and tell the user honestly.

---

## 7. Build plan (one phase at a time)

### Phase 0 — Setup
- Check Node.js is installed; install `clasp` globally.
- Walk the user through: creating the test Gmail, enabling the Apps Script API (script.google.com/home/usersettings), `clasp login` with the test account, creating the Apps Script project, getting the Gemini API key, storing it in Script Properties.
- Create the project folder structure and a `.gitignore` that excludes `.clasprc.json`, `.env`, and any secrets.
- Create a `README.md` explaining the project in plain English.

### Phase 1 — Read and classify
- Script reads new unread emails from the test inbox.
- Sends each to Gemini with a clear prompt asking for **strict JSON only**: `priority`, `assigned_role`, `reason`, `summary` (1–2 sentences), `needs_reply` (true/false).
- Parse JSON safely (strip code fences, handle errors, retry once, fall back to "needs human review").
- Apply Gmail labels like `Priority/Urgent`, `Route/Ops Manager`, and mark as processed so emails aren't handled twice.
- Log every decision to a Google Sheet (time, sender, subject, priority, role, reason).
- Help the user send sample emails to the test inbox to test.

### Phase 2 — Telegram alerts
- Walk the user through creating a bot with @BotFather and getting their chat ID.
- Urgent emails trigger an instant Telegram message: priority, assigned role, sender, subject, summary, reason.
- Store the bot token in Script Properties.

### Phase 3 — Drafted replies
- For emails that need a reply, Gemini drafts one in the assigned role's tone (profiles from section 5).
- Save it as a Gmail draft reply (`createDraftReply`), never sent automatically.
- Send the draft to Telegram with options: approve / edit / ignore.

### Phase 4 — Approvals via Telegram (text, then voice)
- Poll Telegram for replies. Support text commands first: `YES` (send), `EDIT: <instructions>` (regenerate draft), `IGNORE`.
- Add voice notes: download the voice file from Telegram, send the audio to Gemini to understand the intent (approve / edit with instructions / ignore / unclear).
- Always confirm before sending. Unclear → ask again.

### Phase 5 — Extras
- Correction loop ("this should have gone to Accounts") stored and used as few-shot examples.
- On-site briefing command (e.g. `/brief`).
- Optional: audio summaries (see stretch-goal note in section 6).

### Phase 6 — Website demo
- A clean, professional single-page demo branded for Midnight Softwares, deployable free on Vercel.
- Visitor picks a sample email or pastes their own → sees priority, assigned person, reason, and a drafted reply in the right tone.
- Include a simple visual of the full flow (inbox → AI → routing → Telegram alert → voice approval) and a short description of the real system.
- The Gemini key stays server-side (Vercel serverless function + environment variable), never in browser code.
- Add basic protection: input length limit, simple rate limiting, and a friendly message if the free quota is hit.
- No real inbox connection in the public demo.
- Must be mobile-friendly. Help the user link it from midnightsoftwares.in.

### Phase 7 — Portfolio material
- Help the user write a case-study page and a LinkedIn post describing *their own* build.
- Plan a short screen-recorded demo video showing the full flow, including a voice approval on Telegram.

---

## 8. Definition of done

- Sample emails are sorted, labelled, and routed correctly with clear reasons.
- Urgent emails alert Telegram within about a minute.
- Drafts are created in the right tone and never sent without confirmation.
- Voice-note approval works with a confirmation step.
- Public website demo runs on free tiers with no secrets exposed.
- The user can explain, in simple terms, how each part works.
