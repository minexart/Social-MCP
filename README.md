# Social-MCP
A Pet Project for using social media effectively without burnout.
# social-mcp

An MCP server that lets Claude — or any AI assistant — draft, schedule, and publish social media posts on your behalf, just by having a conversation.

> *"Draft a LinkedIn post about how I just shipped my first MCP server, then publish it."*
> — and it does exactly that.

---

## What is this, exactly?

A few terms get thrown around in projects like this. Here's what they actually mean, and which ones apply here.

### MCP — Model Context Protocol
The foundation of this project. MCP is an open standard (created by Anthropic) that gives AI assistants like Claude a structured way to use external tools — APIs, databases, file systems, anything. Without MCP, Claude can only talk. With MCP, Claude can *act*.

This project is an **MCP server**: a small program that registers a set of tools (publish a post, schedule a post, etc.) and makes them available to Claude. Claude Desktop connects to it, sees the tools, and can call them mid-conversation.

Think of it like giving Claude a set of hands.

### AI Agent
An AI agent is an AI that doesn't just answer questions — it takes actions in the world, sometimes across multiple steps, on your behalf. This project qualifies: Claude reasons about what you want, decides which tool to call, calls it, checks the result, and reports back. That loop is what makes it an agent rather than a chatbot.

The key distinction from pure automation: Claude is *deciding* what to do, not just executing a fixed script.

### Automation vs AI Agent
Traditional automation (cron jobs, Zapier flows, scripts) follows rigid if-this-then-that rules written in advance. It can't adapt or reason — it just executes.

An AI agent like this one can interpret vague instructions, adjust tone, draft content, handle errors conversationally, and make judgment calls. The scheduling and publishing parts are automation; the drafting, refining, and decision-making are AI.

This project is both: **AI for the thinking, automation for the execution.**

### UGC — User-Generated Content (the LinkedIn API term)
On LinkedIn's side, all posts created through their API are submitted via the **UGC Posts API** (`ugcPosts`). UGC here is just LinkedIn's internal name for the endpoint — it doesn't mean the content is crowdsourced. It means you (or your app, acting as you) are creating it programmatically. When this server publishes a post to LinkedIn, it's calling that UGC endpoint.

---

## What it does

You connect it to Claude Desktop once. After that, you just talk to Claude:

- *"Write a short post about [topic] and publish it to LinkedIn"*
- *"Schedule that for tomorrow at 9am"*
- *"What posts do I have queued?"*
- *"Cancel the one scheduled for Friday"*

Claude handles the drafting, calls the right tool, and posts on your behalf. You never open LinkedIn.

---

## Platform status

| Platform   | Mode              | Status          |
|------------|-------------------|-----------------|
| LinkedIn   | Direct API        | ✅ implemented  |
| Bluesky    | Direct API        | ⏳ next         |
| X          | Direct API        | ⏳ planned      |
| Threads    | Direct API        | ⏳ planned      |
| Instagram  | Aggregator        | ⏳ planned      |
| TikTok     | Aggregator        | ⏳ planned      |

"Direct API" means this server talks to the platform directly. "Aggregator" means it routes through a third-party service (like Blotato or Ayrshare) that handles the platform's more complex auth requirements.

---

## Tools exposed to Claude

These are the actions Claude can call during a conversation:

| Tool                | What it does                                           |
|---------------------|--------------------------------------------------------|
| `publish_post`      | Publish text to one or more platforms immediately      |
| `schedule_post`     | Schedule a post for a future time                      |
| `list_scheduled`    | List all queued posts                                  |
| `cancel_scheduled`  | Remove a queued post                                   |
| `get_post_status`   | Check whether a publish succeeded                      |

---

## Quick start

```bash
# 1. Clone and install
git clone https://github.com/yourusername/social-mcp.git
cd social-mcp
npm install

# 2. Set up your credentials
cp .env.example .env
# Open .env and fill in your LinkedIn Client ID and Client Secret
# See docs/linkedin-setup.md for exactly where to get these

# 3. Authenticate with LinkedIn (one-time)
npm run auth:linkedin
# Opens a browser → you approve → access token is saved to .env automatically

# 4. Build and start the MCP server
npm run build
npm run start
```

---

## Connecting to Claude Desktop

Add this to your `claude_desktop_config.json` (usually at `~/Library/Application Support/Claude/` on Mac):

```json
{
  "mcpServers": {
    "social": {
      "command": "node",
      "args": ["/absolute/path/to/social-mcp/dist/server.js"]
    }
  }
}
```

Restart Claude Desktop. You'll see a tools icon (🔧) in the chat — that means it's connected.

---

## LinkedIn setup

LinkedIn's OAuth setup has a few non-obvious steps — the Company Page requirement, choosing the right API product, and getting the credentials without hitting `invalid_client`.

**Full walkthrough → [docs/linkedin-setup.md](docs/linkedin-setup.md)**

That guide covers:
- Creating a LinkedIn Developer App
- Which product to request ("Share on LinkedIn", not "Marketing Developer Platform")
- Getting your Client ID and Client Secret
- The common errors and exactly what causes each one

---

## Project structure

```
social-mcp/
├── src/
│   ├── server.ts              # MCP server entry — registers all tools
│   ├── tools/                 # One file per tool Claude can call
│   │   ├── publish-post.ts
│   │   ├── schedule-post.ts
│   │   ├── list-scheduled.ts
│   │   ├── cancel-scheduled.ts
│   │   └── get-analytics.ts
│   ├── platforms/             # One file per social platform
│   │   ├── types.ts           # Shared Platform interface
│   │   ├── linkedin.ts        # LinkedIn implementation (reference)
│   │   ├── bluesky.ts
│   │   └── aggregator.ts      # Fallback for platforms needing aggregators
│   ├── storage/
│   │   └── scheduled.ts       # SQLite — stores scheduled posts locally
│   └── lib/
│       ├── auth.ts            # OAuth token management
│       └── format.ts          # Per-platform character limits and formatting
├── docs/
│   └── linkedin-setup.md      # Full OAuth walkthrough
├── .env.example               # Template — copy to .env, never commit .env
└── tests/
```

---

## Adding a new platform

Implement the `Platform` interface from `src/platforms/types.ts` in a new file, then register it in `src/server.ts`. The LinkedIn implementation (`src/platforms/linkedin.ts`) is the reference.

Each platform needs three things: `publish()`, `getPostStatus()`, and its own auth setup. That's it.

---

## Security

- **Never commit `.env`** — it's in `.gitignore` and should stay there
- **Never commit your Client Secret or access tokens** — treat them like passwords
- The `.env.example` file uses placeholders (`<your-client-id>`) to show structure without exposing real values
- If you demo this project, blur or fake credentials in screenshots and screen recordings

---

## License

MIT — see [LICENSE](LICENSE)
