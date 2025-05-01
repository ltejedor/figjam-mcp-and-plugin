# FigJam MCP Server and Plugin (Bridge)

A **minimal Multi‑Cursor Protocol (MCP) server** that lets any downstream client—such as the companion FigJam plug‑in—spawn sticky notes programmatically, powered by an AI agent using Anthropic's Claude.  
It's perfect for rapid prototyping, white‑boarding bots, or teaching workshops where you want live content to appear on a shared canvas.

---

## Features

* **`/tools/create_sticky`** – enqueue a sticky note (text + x/y) to be rendered in FigJam
* **`/pull`** – lightweight long‑polling endpoint the plug‑in hits every ~2 s for jobs
* **AI-powered agent** – interactive CLI that uses Claude to process natural language commands and create stickies
* **CORS‑friendly** – safe defaults for local dev; configurable for prod
* **Single‑file server** powered by **FastAPI + mcp‑server**

---

## Quick‑start

### 1 · Clone & set up Python env
```bash
$ git clone https://github.com/ltejedor/figjam-mcp-and-plugin.git && cd figjam-mcp-and-plugin
# Python ≥ 3.11 recommended
$ python -m venv .venv && source .venv/bin/activate
$ pip install -r requirements.txt   # fastapi, uvicorn, mcp‑server, smolagents, etc.
```

### 2 · Set up environment variables
Create a `.env` file with your Anthropic API key:
```bash
ANTHROPIC_API_KEY=your_api_key_here
```

### 3 · Run the MCP server
```bash
$ cd figjam-mcp-server
$ python main.py --port 8787
# ▶  Uvicorn running on http://0.0.0.0:8787
```

### 4 · Install & run the FigJam plug‑in
1. Open any FigJam file
2. **Resources → Development → Import plugin from manifest** and select the `MCPJam/manifest.json` file
3. Run the plugin from the Resources panel → Development → MCPJam

### 5 · Run the agent
In a new terminal:
```bash
$ cd ..  # back to project root
$ python main.py
```

Now you can interact with the agent using natural language commands to create stickies in FigJam:
```
Enter command (or 'exit' to quit): create 5 sticky notes about project planning milestones
```

The agent will process your request and create sticky notes that appear in your FigJam board!

---

## Architecture Overview

The system consists of three main components:

1. **MCP Server** (`figjam-mcp-server/main.py`) – FastAPI server that:
   - Manages a job queue for sticky creation commands
   - Provides HTTP endpoints for tools
   - Handles long-polling from the FigJam plugin

2. **FigJam Plugin** (`MCPJam/`) – TypeScript plugin that:
   - Polls the MCP server for new commands
   - Creates sticky notes in FigJam based on received commands

3. **AI Agent** (`main.py`) – Interactive CLI that:
   - Uses Anthropic's Claude to process natural language commands
   - Sends sticky creation requests to the MCP server via HTTP

---

## API Overview

```text
GET  /pull                # returns queued commands
GET  /ping                # health check
GET  /mcp/create_sticky   # HTTP endpoint for tool
POST /tools/call          # generic MCP wrapper (if mounted)
GET  /openapi.json        # full schema
```

### Examples

```bash
# Direct endpoint
curl -X GET "http://localhost:8787/mcp/create_sticky?text=Hello%20World&x=100&y=100"

# Using the agent (via main.py)
# In the interactive CLI:
# > Create a brainstorming session with 10 sticky notes about AI use cases
```

---

## Configuration

| Env Var | Default | Description |
|---------|---------|-------------|
| `PORT`  | `8787`  | Port Uvicorn listens on |
| `ALLOWED_ORIGINS` | `null` | Comma‑separated list for CORS |
| `ANTHROPIC_API_KEY` | - | Required for the AI agent |

Edit `figjam-mcp-server/main.py` if you need finer‑grained CORS control (credentials, headers, etc.).

---

## Usage Tips

* The AI agent can understand complex commands like:
  - "Create a SWOT analysis with 4 categories and 3 items each"
  - "Generate a project timeline with 5 key milestones"
  - "Make a brainstorming board with 10 ideas about sustainability"

* Stickies are positioned automatically, but you can specify coordinates in your commands if needed

* The plugin polls every 2 seconds, so there's a slight delay between command and appearance

---

## Troubleshooting

* **CORS errors in browser console** – if your FigJam iframe sends `credentials: "include"` you must set `allow_credentials=True` *and* avoid the wildcard origin
* **Stickies never appear** – check the server log for queued jobs and ensure the plug‑in's `/pull` requests return them
* **Agent errors** – ensure your Anthropic API key is set correctly in `.env`
* **Plugin not loading** – make sure you're using a FigJam file (not a regular Figma design file)

---

## Development

To modify the agent behavior, edit `main.py`. You can:
- Add custom tools by creating new `@tool` decorated functions
- Modify the agent's system prompt or behavior
- Add new imports or capabilities

To modify the plugin behavior, edit `MCPJam/code.ts` and rebuild:
```bash
$ cd MCPJam
$ npm run build
```
