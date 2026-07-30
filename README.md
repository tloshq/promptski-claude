# Promptski for Claude Code

Turn your current session's context — or any file or pasted text — into a **[Promptski](https://www.promptski.ai) Handoff Link**: a single URL plus a paste-ready one-liner. The recipient drops that one line into *their* AI (Claude, ChatGPT, Gemini — anything that can fetch a URL) and it ingests the full context itself.

No files. No downloads. No "let me export a markdown doc and email it to you."

## Install

```
/plugin marketplace add tloshq/promptski-claude
/plugin install promptski@promptski
```

## Use

```
/handoff                      # package the current session
/handoff path/to/notes.md     # package a specific file
/handoff <pasted text>        # package pasted text
```

You get back:

1. A **ready-to-send message** containing the recipient one-liner — the line they paste into their AI.
2. The human-readable page URL (for recipients who aren't using an AI).

## How it works

The skill packages your context, checks it for secrets and sensitive content (with your sign-off before anything questionable ships), then POSTs it to `https://www.promptski.ai/api/handoff`. Promptski mints an unlisted public URL (`promptski.ai/h/<slug>`) with a raw markdown endpoint that any AI can fetch directly.

## Good to know

- **Links are public-unlisted.** Anyone with the URL can read the content — the skill treats minting as publishing and gates sensitive content accordingly. Secrets (API keys, credentials) are never shipped, ever.
- **v1 links are permanent.** Revocation, expiration, and end-to-end encryption are on the roadmap.
- **Nothing is sent for you.** The skill drafts the message; you send it.
- Free to use; the API is rate-limited per IP.

## License

MIT
