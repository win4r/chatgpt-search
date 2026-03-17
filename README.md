# 🔍 chatgpt-search

An [OpenClaw](https://github.com/openclaw/openclaw) skill that searches the web using ChatGPT via browser automation. Instead of using traditional search APIs, this skill opens ChatGPT in your logged-in Chrome browser, submits a search query, waits for the AI-generated response, and extracts the results — giving you ChatGPT's web search capabilities directly from your OpenClaw agent.

## Why?

Sometimes you want a **second opinion** beyond standard web search APIs. ChatGPT's built-in web search synthesizes information differently — it reads multiple sources, reasons about them, and returns a structured summary with citations. This skill lets your OpenClaw agent tap into that capability automatically.

## How It Works

```
User: "用ChatGPT搜索 OpenClaw 最新版有什么新特性"
         │
         ▼
┌─────────────────────────┐
│  1. Open chatgpt.com    │  ← Uses your logged-in Chrome session
│  2. Type search query   │  ← Auto-constructs effective prompts
│  3. Submit & wait       │  ← Smart polling with timeout handling
│  4. Extract response    │  ← Parses ChatGPT's structured output
│  5. Summarize to user   │  ← Formats in user's language
└─────────────────────────┘
```

## Features

- **Browser automation** — Controls your real Chrome browser via `profile="user"`, leveraging your existing ChatGPT login
- **Smart polling** — Handles ChatGPT's variable response times (15–60s) with a robust wait-and-check strategy
- **Auto query construction** — Prefixes queries with "Search the web for:" to trigger ChatGPT's web search mode
- **Bilingual support** — Accepts queries in any language, presents results in the user's language
- **Structured extraction** — Parses headings, body text, and source links from ChatGPT's response
- **Error resilience** — Gracefully handles timeouts, login prompts, capacity limits, and partial responses

## Prerequisites

- [OpenClaw](https://github.com/openclaw/openclaw) installed and running
- Google Chrome with an active ChatGPT login (Plus/Team/Enterprise for best results)
- OpenClaw browser tool configured with `profile="user"` (attach to existing Chrome session)

## Installation

### Option 1: Copy to your workspace skills directory

```bash
git clone https://github.com/win4r/chatgpt-search.git
cp -r chatgpt-search/SKILL.md ~/.openclaw/workspace/skills/skills/chatgpt-search/SKILL.md
```

### Option 2: Clone directly into skills

```bash
cd ~/.openclaw/workspace/skills/skills
git clone https://github.com/win4r/chatgpt-search.git
```

Then restart the OpenClaw gateway:

```bash
openclaw gateway restart
```

## Usage

Once installed, the skill triggers automatically when you say things like:

| Trigger | Language |
|---|---|
| `用ChatGPT搜索 XXX` | Chinese |
| `让ChatGPT查一下 XXX` | Chinese |
| `ChatGPT search for XXX` | English |
| `Ask ChatGPT to search XXX` | English |

### Examples

```
用ChatGPT搜索 OpenClaw 最新版新增了哪些特性
ChatGPT search for the latest Claude Code features in March 2026
让ChatGPT查一下 Anthropic 最近发布了什么新模型
```

The agent will:
1. Open a new tab at chatgpt.com
2. Type and submit the search query
3. Wait for ChatGPT to finish generating (with smart polling)
4. Extract and summarize the response
5. Present the results in your language

## Skill File Structure

```
chatgpt-search/
├── SKILL.md      # Skill definition (triggers + workflow instructions)
└── README.md     # This file
```

## How the Polling Works

ChatGPT's web search responses can take 15–60+ seconds. The skill uses a polling strategy:

1. **Initial wait** (20s) — gives ChatGPT time to search and start generating
2. **Snapshot check** — looks for completion indicators:
   - ✅ Done: `"Copy response"` / `"Sources"` buttons appear
   - ⏳ Still generating: `"Stop streaming"` button present
3. **Repeat** up to 3 more times (total ~80s max)
4. **Fallback** — if still generating after 4 cycles, extracts partial content

## Limitations

- Requires an active ChatGPT login in Chrome (not headless)
- Response times depend on ChatGPT's server load
- ChatGPT may hit rate limits on frequent searches
- Browser must be running and accessible via Chrome DevTools Protocol

## Related

- [OpenClaw](https://github.com/openclaw/openclaw) — The open-source AI agent framework
- [OpenClaw Skills](https://clawhub.com) — Browse and discover more skills

## License

MIT
