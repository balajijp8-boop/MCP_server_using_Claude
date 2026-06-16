# 🏆 World Cup 2026 Predictor — MCP server

A **zero-dependency [Model Context Protocol](https://modelcontextprotocol.io)
server** that plugs a World Cup 2026 prediction engine into **Claude Desktop**
as callable tools — so you just *chat* with it:

> 💬 "What are Brazil's odds to win the World Cup?"
> 💬 "Predict Spain vs Argentina."
> 💬 "Show me Group D's predicted standings."

Under the hood it runs a real statistical pipeline — **FIFA ratings + squad
strength + form → Dixon-Coles bivariate-Poisson → 12,000-run Monte-Carlo
tournament simulation**, with **live scores pulled from ESPN**. It speaks
JSON-RPC 2.0 over stdio by hand, so there's **no `npm install`**. Needs **Node 18+**.

---

## 🎬 Demo

Claude Desktop calling the predictor's tools live:

<video src="https://github.com/balajijp8-boop/MCP_server_using_Claude/raw/main/MCP_demo.MP4" controls width="100%"></video>

> ▶ If the player above doesn't load in your browser,
> [**watch / download `MCP_demo.MP4`**](MCP_demo.MP4) directly.

---

## ⚠️ Requires the engine from the main project

This repo contains **only the MCP server**. The prediction engine itself (the
ratings model, the Dixon-Coles + Monte-Carlo simulation, and the live ESPN score
feed) lives in the main project:

👉 **https://github.com/balajijp8-boop/Worldcup_live_predictor** — the `js/` folder.

The server loads four files from there: `config.js`, `data.js`, `engine.js`,
`livescore.js`. You point it at them with the `WC_ENGINE_DIR` environment variable.

---

## 🚀 Setup

1. **Install Node 18+** from <https://nodejs.org/>.

2. **Clone the main project** (this is the engine the server reuses):

   ```bash
   git clone https://github.com/balajijp8-boop/Worldcup_live_predictor.git
   ```

3. **Clone this repo:**

   ```bash
   git clone https://github.com/balajijp8-boop/MCP_server_using_Claude.git
   ```

4. **Register it with Claude Desktop.** Open
   `claude_desktop_config.json` (Windows: `%APPDATA%\Claude\claude_desktop_config.json`,
   macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`) and
   add the server, pointing `WC_ENGINE_DIR` at the main project's `js/` folder:

   ```json
   {
     "mcpServers": {
       "worldcup-predictor": {
         "command": "node",
         "args": ["C:\\path\\to\\MCP_server_using_Claude\\server.js"],
         "env": { "WC_ENGINE_DIR": "C:\\path\\to\\Worldcup_live_predictor\\js" }
       }
     }
   }
   ```

   > 💡 If you instead drop `server.js` into the main project as `mcp/server.js`,
   > `WC_ENGINE_DIR` is optional — it defaults to `../js`.

5. **Fully quit and reopen Claude Desktop** (tray → Quit — closing the window
   isn't enough; the config is only re-read on a full restart). The
   `worldcup-predictor` connector then appears with an on/off toggle, and its
   tools show up in the 🔌 menu of the chat box.

6. **Ask away** — e.g. *"What are Brazil's odds to win the World Cup?"* The first
   question of a session takes a few seconds (it fetches live scores and runs the
   full simulation), then answers are instant.

---

## 🛠️ Tools

You never call these directly — just ask Claude in plain English and it picks the
right one (and chains several when useful).

| Tool | What it does |
|------|--------------|
| `predict_match` | Win/draw/loss + expected goals (xG) + likely score for any two teams |
| `tournament_odds` | Championship & round-reach odds (one team, or all 48 ranked) |
| `group_standings` | Predicted final standings for a group (A–L) or all groups |
| `predicted_bracket` | Best-guess knockout bracket → predicted champion |
| `team_info` | Rank, ratings, squad index, form, group standing, headline odds |
| `live_scores` | Current real scores from ESPN (finished + live now) |
| `refresh` | Re-pull live scores and re-run the whole simulation |

---

## 🧪 Test it without Claude

```bash
# point at your clone of the main project's js/ folder
WC_ENGINE_DIR=/path/to/Worldcup_live_predictor/js node selftest.js
```

This drives the tools end-to-end (engine + live fetch) and prints sample output.

---

## ⚙️ Tuning & notes

- **Faster responses:** the sim is 12,000 runs by default. Lower it with an env
  var — add `"WC_MC_RUNS": "4000"` to the server's `env` block.
- **Offline:** if ESPN is unreachable the server logs a note to stderr and falls
  back to the bundled pre-tournament FIFA ratings — everything still works.
- **Logging:** server logs go to **stderr** (stdout is reserved for the MCP
  protocol). In Claude Desktop they land in `%APPDATA%\Claude\logs\`.

---

## 📄 License

MIT — see [LICENSE](LICENSE).
