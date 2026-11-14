# AI Email Triage & Voice Approval Agent (Demo)

A portfolio demo by **Midnight Softwares**. It shows how an AI agent can tame a chaotic inbox for a fictional UK construction company, *Brickline Construction Ltd*.

## What it does (in plain English)

1. **Reads** new emails in a test Gmail inbox.
2. **Sorts** them by priority: urgent, normal or low.
3. **Routes** each one to the right person (Managing Director, Operations Director, Accounts, Ops Manager) and says why.
4. **Drafts a reply** in the right tone for that person.
5. **Alerts** the person on Telegram when something is urgent.
6. **Waits for approval**: reply YES or send a voice note. Nothing is ever sent without a human saying so.

## Built with free tools only

| Part | Tool |
|---|---|
| Email + automation | Google Apps Script (via `clasp`) |
| AI | Google Gemini API (free tier) |
| Alerts + approvals | Telegram bot |
| Website demo | Vercel (free Hobby plan) |

## Folder layout

```
src/                 Apps Script code (pushed to Google with clasp)
  Config.gs          team, tones, priority rules, limits
  Gemini.gs          talks to the AI
  Classify.gs        priority + routing + reason
  Main.gs            the once-a-minute heartbeat, labels, decision log
  Drafts.gs          reply drafts (never auto-sent)
  Telegram.gs        alerts, polling, voice notes
  Approvals.gs       YES / confirm / EDIT / IGNORE / wrong-route logic
  Corrections.gs     the learning loop
  Brief.gs           on-site briefing
  Setup.gs           run-by-hand helpers (setup, triggers, sample emails)
  SampleEmails.gs    generated from public/sample-emails.json
public/              the website demo (static files)
api/triage.js        website's serverless function (Gemini key stays server-side)
scripts/             make-samples.js (regenerate samples), check.js (npm test)
```

Run `npm test` to check the core logic, including the rule that nothing is sent until "yes" has been said twice.

## Security

API keys live in Apps Script **Script Properties** (or Vercel environment variables). They never go in code files or git. Only fictional demo emails are used.

## Progress

- [x] Phase 0 - Setup
- [~] Phase 1 - Read and classify (code written, not yet run against real Gmail)
- [~] Phase 2 - Telegram alerts (code written, not yet run)
- [~] Phase 3 - Drafted replies (code written, not yet run)
- [~] Phase 4 - Approvals, text and voice (code written; confirmation logic unit-tested, voice untested)
- [~] Phase 5 - Extras: corrections and /brief (code written, not yet run). Audio summaries: not started
- [~] Phase 6 - Website demo (built and tested with a fake AI; not yet deployed)
- [ ] Phase 7 - Portfolio material
