---
name: chatgpt-search
description: "Search the web using ChatGPT via browser automation. Open chatgpt.com in the user's logged-in Chrome, type a search query, wait for ChatGPT to finish generating, and extract the response. Use when: (1) the user asks to search something with ChatGPT, (2) the user says '用ChatGPT搜索', '让ChatGPT查一下', 'ask ChatGPT to search', 'ChatGPT search', (3) the user wants a second-opinion web search from ChatGPT instead of Brave/Google. NOT for: simple web searches (use web_search), questions that don't need web search, or when the user wants to chat with ChatGPT interactively."
---

# ChatGPT Search via Browser (profile="user")

Search the web through ChatGPT's interface by automating the user's logged-in Chrome browser.

## Prerequisites

- User must be logged into chatgpt.com in Chrome
- All browser calls MUST use `profile="user"`

## Workflow

### Step 1: Open ChatGPT

```
browser.open → url: "https://chatgpt.com", profile: "user"
```

Note the `targetId` from the response — use it for ALL subsequent calls.

### Step 2: Snapshot and locate the input textbox

```
browser.snapshot → profile: "user", targetId: <targetId>
```

Find the `textbox` element (typically `ref` like `1_14`). Also note if ChatGPT is ready (look for heading like "Ready when you are." or the input area).

### Step 3: Click the textbox to focus

```
browser.act → kind: "click", ref: <textbox_ref>, profile: "user", targetId: <targetId>
```

### Step 4: Type the search query

```
browser.act → kind: "type", ref: <textbox_ref>, text: "<query>", profile: "user", targetId: <targetId>
```

Query construction rules:
- Prefix with `Search the web for: ` to trigger ChatGPT's web search mode
- Be specific and detailed in the query
- Use English for broader search coverage (unless user explicitly wants Chinese results)
- If the user's original question is in Chinese, translate the core query to English but note "Please provide the summary" at the end

Example query for a Chinese request "用ChatGPT搜索OpenClaw最新特性":
```
Search the web for: What are the latest new features added in the newest version of OpenClaw (open source AI agent framework, GitHub repo openclaw/openclaw)? Please provide a comprehensive summary.
```

### Step 5: Submit the query

```
browser.act → kind: "press", key: "Enter", profile: "user", targetId: <targetId>
```

### Step 6: Wait for ChatGPT to generate the response

ChatGPT's response takes 15–60 seconds depending on query complexity (especially with web search + thinking).

**Polling strategy:**

1. Wait 20 seconds initially:
   ```
   browser.act → kind: "wait", timeMs: 20000, profile: "user", targetId: <targetId>
   ```
   (This will likely timeout — that's expected, ignore the timeout error.)

2. Take a snapshot to check progress:
   ```
   browser.snapshot → profile: "user", targetId: <targetId>, maxChars: 15000
   ```

3. Check for completion indicators:
   - ✅ **Done:** Buttons like `"Copy response"`, `"Good response"`, `"Bad response"`, `"Sources"` appear at the end of the response → response is complete
   - ⏳ **Still generating:** `"Stop streaming"` button present, or status text says "ChatGPT is still generating a response..." → wait more

4. If still generating, wait another 15–20 seconds and snapshot again. Repeat up to 3 more times (total ~80 seconds max wait).

5. If after 4 polling cycles it's still generating, extract whatever is available and note it's partial.

### Step 7: Extract the response

From the final snapshot, extract all `statictext` content under the `article` with heading `"ChatGPT said:"`. Reconstruct the full response by concatenating the text nodes in order.

Key elements to capture:
- Main body text (all `statictext` nodes)
- Headings (`heading` elements) — these structure the response
- Source links (`link` elements with "GitHub", domain names, etc.)

Ignore:
- The "You said:" article (user's query echo)
- Navigation elements, buttons, sidebar content
- The thinking/reasoning summary button text

### Step 8: Present to user

Format the extracted content as a clean, structured summary in the user's language:
- If the user asked in Chinese, summarize in Chinese
- Preserve the logical structure (numbered lists, sections)
- Include key facts, version numbers, dates
- Mention that the full response is viewable in the browser

## Error handling

| Scenario | Action |
|---|---|
| Login required (redirected to login page) | Tell user to log in manually, then retry |
| ChatGPT is at capacity | Report to user, suggest trying later |
| Response never completes (>80s) | Extract partial content, note it's incomplete |
| Textbox not found in snapshot | Try `browser.navigate → url: "https://chatgpt.com"` to refresh, re-snapshot |
| Browser timeout errors on `wait` | These are expected — proceed to snapshot and check status |

## Notes

- The `wait` action will almost always timeout for long ChatGPT responses — this is normal. The timeout error is harmless; just proceed to snapshot.
- ChatGPT may use a "Thinking" phase before responding (visible as "Thought for Xs" button). This is normal and the response will follow.
- Always use the same `targetId` throughout the workflow to stay on the same tab.
