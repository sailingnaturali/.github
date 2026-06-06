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
| [signalk-mcp](https://github.com/sailingnaturali/signalk-mcp) | Read live vessel data from SignalK — wind, depth, position, battery, route — with path discovery and under-keel clearance |
| [weather-mcp](https://github.com/sailingnaturali/weather-mcp) | Marine wind, swell, and buoy observations — Open-Meteo + Stormglass + NDBC |
| [tide-mcp](https://github.com/sailingnaturali/tide-mcp) | Tidal gate slack windows for PNW passages — CHS (BC) + NOAA CO-OPS (US) |
| [pilotbook-mcp](https://github.com/sailingnaturali/pilotbook-mcp) | Rank the most comfortable overnight anchorage by joining pilot-book exposure against the forecast |
| [colregs-mcp](https://github.com/sailingnaturali/colregs-mcp) | Navigation rules of the road — resolve the regime by GPS, look up rules, and check lights/shapes compliance |
| [logbook-mcp](https://github.com/sailingnaturali/logbook-mcp) | Capture marked moments and sea-day entries to a local SQLite logbook |
| [vessel-knowledge-mcp](https://github.com/sailingnaturali/vessel-knowledge-mcp) | Turn equipment manuals into alarm zones and equipment lookups — ingests PDFs into a card vault, pushes zone bands into SignalK so it raises notifications autonomously, then explains them in plain language |

### Vaults

Knowledge bases the MCP servers read from.

| Repo | What it holds |
|------|-------------|
| [colregs-vault](https://github.com/sailingnaturali/colregs-vault) | Navigation-rules content for colregs-mcp — rule prose, a curated requirements decision table, and PNW regime polygons. Public because the sources are reproducible public-domain law |
| `pilotbook-vault` *(private)* | Anchorage knowledge for pilotbook-mcp — 673 anchorages across 7 SalishSeaPilot books, with exposed-sector data and source page back-links. Kept private: the source pilot books are copyrighted |
| `vessel-knowledge-vault` *(private)* | Equipment cards for vessel-knowledge-mcp — per-make/model spec cards extracted from manuals (Bellmarine drives, Victron Cerbo, watermaker), with zone bands in SignalK canonical SI units. Kept private: the source manuals are copyrighted |

### SignalK Plugins

Alarms don't wait for an agent — these run inside the SignalK server itself.

| Repo | What it does |
|------|-------------|
| [signalk-ntfy-relay](https://github.com/sailingnaturali/signalk-ntfy-relay) | Push SignalK notifications (alarms) straight to your phone via [ntfy](https://ntfy.sh) — zero dependencies, edge-triggered, severity-aware. On npm as [`signalk-ntfy-relay`](https://www.npmjs.com/package/signalk-ntfy-relay) |

### Agents

| Repo | What it does |
|------|-------------|
| [naturali-agents](https://github.com/sailingnaturali/naturali-agents) | Hermes Agent skills for Navigator, Engineer, and Logbook — prompts, bridge scripts, and HA voice integration |

---

## How it fits together

Three replaceable layers. An agent reads vessel + environment data through MCP
servers, reasons over it, and speaks back through Home Assistant.

```
  Interaction        Home Assistant (Pi 5)
                       voice in ──MQTT──▶  │   ◀──MQTT── Piper TTS out
                                           ▼
  Agent runtime      Hermes Agent (Mac Studio / Mac mini)
                       Hermes 3 8B local via Ollama  +  Claude API
                                           │
                      ┌──────────┬─────────┼─────────┬──────────┬─────────┬─────────────┐
  Tool surface        ▼          ▼         ▼         ▼          ▼         ▼             ▼
                   signalk    weather     tide   pilotbook   colregs   logbook   vessel-knowledge
                    -mcp       -mcp       -mcp     -mcp        -mcp      -mcp          -mcp
                      │          │         │         │          │         │             │
  Data / vaults    SignalK   Open-Meteo  CHS +   pilotbook-  colregs-   sqlite   vessel-knowledge-
                    bus       NDBC        NOAA    vault       vault                   vault
```

Each MCP server runs standalone and is independently useful — point your own agent
at any one of them. The vaults are plain Markdown/GeoJSON the servers read at
startup (`PILOTBOOK_VAULT_PATH`, `COLREGS_VAULT_PATH`, `VESSEL_KNOWLEDGE_VAULT_PATH`).
vessel-knowledge-mcp also runs the loop in reverse: its build-time CLI pushes alarm-zone
metadata *into* SignalK, so the bus raises notifications on its own and the agent only
explains them.

## Getting started

Start at **[naturali-agents](https://github.com/sailingnaturali/naturali-agents)** — it
wires the MCP servers into a working agent and ships a mock SignalK server, so the whole
loop runs on a laptop with no boat:

```bash
git clone https://github.com/sailingnaturali/naturali-agents
cd naturali-agents
python dev/mock-signalk.py            # serves a Boundary Pass scenario on :8765
export SIGNALK_URL=http://localhost:8765
hermes chat -s naturali/navigator     # ask the Navigator anything
```

Each repo's README carries its own install, env vars, and tool surface. Every MCP
server is a `uv`-runnable Python package with its own test suite.

---

Built in public. MIT licensed. Show the receipts. The build log lives on the
[engineering blog](https://engineering.sailingnaturali.com)
([source](https://github.com/sailingnaturali/engineering)), and the main site
at [sailingnaturali.com](https://sailingnaturali.com)
([source](https://github.com/sailingnaturali/web)).
