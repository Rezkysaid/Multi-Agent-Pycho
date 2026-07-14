# Mind Crew

A single-file, isometric pixel-art **personal AI crew** you run in the browser — a
team that helps with everyday missions: daily life, learning, decisions, side
projects, business, content, research and problem solving. You talk to one Chief
(Aiman / "Ketua") and a crew of specialist agents handle the rest behind the scenes.
Deployed as static hosting on GitHub Pages — the whole app is `index.html` (HTML +
CSS + JS, no build step, no backend).

## The team

| Agent | Role | Does |
|-------|------|------|
| Aiman | **Ketua** (Chief Coordinator) | Understands intent, auto-delegates, combines results into one update |
| Meera | Penyelidik (Research Analyst) | Research, fact-check, summaries, comparisons — **the only agent allowed to web-search** |
| Wei | Komunikasi | Email / WhatsApp / Telegram drafts, social captions & hashtags |
| Danish | Penasihat Strategi | Analysis, brainstorming, decisions |
| Kak Sumi | Perancang & Jadual (Schedule Planner) | Plans, timelines, **reminders & alarms** |
| Hafiz | Jurutera (Engineering) | Code, debug, automation, integration, review |

You never pick an agent — the Chief routes automatically. Naming one ("Hafiz, …")
still works but is optional.

## API keys (BYOK)

Keys are your own and are stored **in this device's browser localStorage** — press the
**API** button (top right) to manage them.

- **LLM brain** (required): Google Gemini (`AIza…`), DeepSeek (`sk-…`) or OpenRouter
  (`sk-or-…`). Provider is auto-detected from the key shape. Without it the app runs
  in demo mode (canned replies).
- **Tavily** (optional): live web search for the Research Agent.

## Live web search (Tavily)

### Getting a key
1. Sign up at <https://tavily.com> and copy your key (`tvly-…`).
2. Press **API** → after the LLM key prompt you'll be asked for a Tavily key.
3. The key is validated with a lightweight test search; status shows in the API
   button tooltip (masked, last 4 chars only).

### How it works
- **Intelligent trigger** — the orchestrator decides if live data is needed. On a
  DeepSeek/OpenRouter brain the model is given a `web_search` tool and calls it only
  when it needs current data. On a Gemini brain a keyword heuristic decides.
- **Manual override** — start a message with `/web`, `/search` or `/live` to force a
  search.
- **Should trigger**: latest / terkini / semasa / current / today / price / harga /
  trend / news / statistics / official docs.
- **Should NOT trigger**: translation, rewriting, grammar, coding from the project,
  brainstorming, file summarization, maths.

### Quota rules (enforced in code, not just prompts)
- **Only the Research Agent (Meera)** may call `web_search` — enforced via
  `canUseTool(agent, 'web_search')`. Other agents reuse the shared research context.
- **Max 3 searches per task**, **max 5 results per search**.
- Results are **cached per task**; a repeated/normalized query returns the cached
  result and does **not** consume quota.
- One retry on a transient network blip; **never** retried on invalid key (401) or
  quota (429).
- When Tavily is missing or fails, live search is disabled and **the app keeps
  working** on the model's own knowledge.

### Sources
A **Sumber Web** panel lists every source found (title, domain, published date,
clickable URL) and shows searches used (`carian n/3`).

### Security notes
- Query is sanitized and length-capped.
- Web results are treated as **untrusted**: the agent is instructed to use them as
  reference facts only and never to obey instructions embedded in a page (prompt-
  injection guard).

## Reminders

Set in plain Malay: `ingatkan aku call client dalam 30 minit`, `esok pukul 3 petang`,
`bayar sewa 1hb`. Kak Sumi confirms and, when due, announces it in chat plus a browser
notification (if you allow it). Reminders live in a collapsible **Peringatan** panel,
stored on-device.

## Deploy

Push to `main`; GitHub Pages serves `index.html`. No build/lint/typecheck step (a
single hand-written HTML file).

## Testing

Playwright smoke tests drive the app with **mocked** providers (no real API calls).
The Tavily suite covers: successful search, missing key, 401, 429, timeout, malformed
response, empty results, cache dedupe, permission denied, max-searches, Tavily
disabled, and final sources rendering, plus the end-to-end DeepSeek tool-calling loop.

## Limitations vs. a backend spec

This app is **static (GitHub Pages) with no server**, so a few requirements from a
backend-oriented spec cannot apply and are intentionally not implemented:

- **Keys can't be hidden from the frontend** — there is no server to hold them. They
  live in this device's `localStorage` (the app's established BYOK model). True
  server-side secrecy, DB encryption, per-authenticated-user isolation and endpoint
  rate-limiting require a backend proxy (e.g. a small Vercel function). That's a
  separate piece of work; everything feasible client-side (search logic, the
  Research-Agent-only rule, quota, cache, tool-calling, sanitization, untrusted-result
  handling) is implemented here.
- Tavily is called directly from the browser; if Tavily blocks cross-origin requests
  from your origin, live search degrades gracefully and the app continues.
