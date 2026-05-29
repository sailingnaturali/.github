# Sailing Naturali

A tech exec uses AI to build the kind of business AI can't deliver.

All-electric catamaran charter operation in the Pacific Northwest — Gulf Islands and San Juan Islands. AI runs the operations. Humans deliver the experience.

📺 [sailingnaturali.com](https://sailingnaturali.com) · [@sailingnaturali](https://www.youtube.com/@sailingnaturali) on YouTube · [@sailingnaturali](https://www.instagram.com/sailingnaturali) on Instagram

---

## Boat Agent Stack

On-boat AI agent system — Navigator, Engineer, Logbook — built on local inference + Claude API, wired to SignalK via MCP servers.

### MCP Servers

| Repo | What it does |
|------|-------------|
| [signalk-mcp](https://github.com/sailingnaturali/signalk-mcp) | Read live vessel data from SignalK — wind, position, battery, route |
| [weather-mcp](https://github.com/sailingnaturali/weather-mcp) | Marine wind, swell, and buoy observations — Open-Meteo + Stormglass + NDBC |
| [tide-mcp](https://github.com/sailingnaturali/tide-mcp) | Tidal gate slack windows for PNW passages — CHS (BC) + NOAA CO-OPS (US) |
| [pilotbook-mcp](https://github.com/sailingnaturali/pilotbook-mcp) | Rank the most comfortable overnight anchorage by joining pilot-book exposure against the forecast |
| [logbook-mcp](https://github.com/sailingnaturali/logbook-mcp) | Capture marked moments and sea-day entries to a local SQLite logbook |

### Agents

| Repo | What it does |
|------|-------------|
| [naturali-agents](https://github.com/sailingnaturali/naturali-agents) | Hermes Agent skills for Navigator, Engineer, and Logbook — prompts, bridge scripts, and HA voice integration |

---

Built in public. MIT licensed. Show the receipts.
