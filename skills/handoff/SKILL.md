---
name: handoff
description: Turn the current session's context (or a named file / pasted text) into a live Promptski Handoff Link — a promptski.ai URL plus a paste-ready one-liner the recipient drops into THEIR AI, which fetches the context itself. No files, no downloads. Use when the user says "/handoff", "handoff this", "make a handoff", "package this for <person>", "hand this off", or wants to send session context or a doc to another person's Claude/ChatGPT/Gemini. Drafts the send message; never sends it anywhere.
---

# 🔗 /handoff — mint a Promptski Handoff Link

> Kill the handoff-file ritual: package context → POST to Promptski → get back a link + a recipient one-liner. The recipient pastes ONE line into their AI and it ingests everything itself.

## 1. Determine the payload (from the arguments / conversation)
- **No args or "this session"** → package the CURRENT session: what you're working on, decisions made, current state, open loops, exact file paths/IDs the next AI needs, and the next move. Write it for an AI reader — complete enough to resume cold.
- **A file path** → use that file's content verbatim (read it first).
- **Pasted/quoted text** → use it verbatim.
- Title: short + specific ("<topic> — handoff for <person>" when a recipient is named). Limits: title ≤ 200 chars, body ≤ 200,000 chars (trim oldest / least load-bearing context first if over).

## 2. 🔒 Sensitive-content gate (handoff links are PUBLIC unlisted URLs)
Anyone with the link can read it — treat minting as publishing. Before posting, scan the drafted body for:
- **Secrets**: API keys, tokens, passwords, private keys, connection strings, anything credential-shaped. **Never ship these — no override.** Strip or redact, then continue.
- **Private business/personal data**: internal project or client names, revenue/financial numbers, unreleased plans, personal contact details, health/legal matters.

On any non-secret hit: STOP and show the user exactly what tripped — offer (a) anonymize/remove it, or (b) an explicit "send it anyway" for a trusted recipient. Never silently ship sensitive content.

## 3. Mint the link
Write the body to a temp file, then build the JSON safely (never hand-escape the body into an inline string):

```bash
python3 - <<'EOF'
import json, urllib.request
title = open("/tmp/handoff-title.txt").read().strip()
body = open("/tmp/handoff-body.md").read()
req = urllib.request.Request(
    "https://www.promptski.ai/api/handoff",
    data=json.dumps({"title": title, "body": body}).encode(),
    headers={"content-type": "application/json"},
)
print(urllib.request.urlopen(req, timeout=30).read().decode())
EOF
```

(Or the `curl` equivalent with `--data @file.json`. Any HTTP tool works — it's a plain JSON POST.)

- Response: `{ slug, url, rawUrl, recipientPrompt }`. Use `recipientPrompt` verbatim — it is the canonical one-liner (the template lives server-side).
- Errors: **413** → body too big; trim and retry once. **429** → rate-limited; wait and tell the user. Network/5xx → report it and check that https://www.promptski.ai is up; do NOT retry-loop.

## 4. Verify before handing over (10 seconds, always)
- Fetch `rawUrl` → it must return the markdown verbatim with a direct 200 (no redirect).

## 5. Deliver
Give the user, in this order:
1. **The ready-to-send message** (one copy-paste block): a one-line human intro ("Here's the full context — paste the line below into your AI and it'll pick everything up:") + the `recipientPrompt`.
2. The human page URL (`url`) as the fallback for non-AI recipients.
3. What was packaged (one line) + anything anonymized or trimmed.

The user sends it themselves (iMessage/Slack/email) — this skill never posts anywhere.

## Guardrails
- The sensitive-content gate above is mandatory — a handoff link is publication.
- v1 links are permanent and unrevokable (revocation is on the roadmap) — say so when the content is at all sensitive.
- This skill only creates content links; it never sends messages, posts publicly on the user's behalf, or touches money.
