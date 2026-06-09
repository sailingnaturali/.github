# Sailing Naturali

A tech exec uses AI to build the kind of business AI can't deliver.

All-electric catamaran charter operation in the Pacific Northwest — Gulf Islands and San Juan Islands. AI runs the operations. Humans deliver the experience.

📺 [@sailingnaturali](https://www.youtube.com/@sailingnaturali) on YouTube · [sailingnaturali](https://sailingnaturali.substack.com) on Substack · [sailingnaturali.com](https://sailingnaturali.com) · [@sailingnaturali](https://www.instagram.com/sailingnaturali) on Instagram · [@sailingnaturali](https://www.tiktok.com/@sailingnaturali) on TikTok

---

## Boat Agent Stack

On-boat AI agent system — Navigator, Engineer, Logbook — built on the Claude API, wired to SignalK via MCP servers.

### MCP Servers

| Repo | What it does |
|------|-------------|
| [signalk-mcp](https://github.com/sailingnaturali/signalk-mcp) | Read live vessel data from SignalK — wind, depth, position, battery, route — with path discovery and under-keel clearance |
| [weather-mcp](https://github.com/sailingnaturali/weather-mcp) | Marine wind, swell, and buoy observations — Open-Meteo + Stormglass + NDBC |
| [currents-mcp](https://github.com/sailingnaturali/currents-mcp) | Tidal-gate slack windows for PNW passages — CHS (BC) + NOAA (US) current predictions |
| [pilotbook-mcp](https://github.com/sailingnaturali/pilotbook-mcp) | Rank the most comfortable overnight anchorage by joining pilot-book exposure against the forecast — and search anchorages in plain language ("quiet spot to wait out a southerly with a beach") via on-device hybrid retrieval |
| [colregs-mcp](https://github.com/sailingnaturali/colregs-mcp) | Navigation rules of the road — resolve the regime by GPS, look up rules, and check lights/shapes compliance |
| [logbook-mcp](https://github.com/sailingnaturali/logbook-mcp) | The ship's log as agent tools — voice-marked moments and day reads over the [signalk-logbook](https://github.com/meri-imperiumi/signalk-logbook) plugin, with USCG/TC sea-time export on the roadmap |
| [vessel-knowledge-mcp](https://github.com/sailingnaturali/vessel-knowledge-mcp) | Turn equipment manuals into alarm zones and equipment lookups — ingests PDFs into a card vault, pushes zone bands into SignalK so it raises notifications autonomously, then explains them in plain language |

### Vaults

Knowledge bases the MCP servers read from.

| Repo | What it holds |
|------|-------------|
| [colregs-vault](https://github.com/sailingnaturali/colregs-vault) | Navigation-rules content for colregs-mcp — rule prose, a curated requirements decision table, and PNW regime polygons. Public because the sources are reproducible public-domain law |
| `pilotbook-vault` *(private)* | Anchorage knowledge for pilotbook-mcp — 673 anchorages across 7 SalishSeaPilot books, with exposed-sector data and source page back-links. Kept private: the source pilot books are copyrighted |
| `vessel-knowledge-vault` *(private)* | Equipment cards for vessel-knowledge-mcp — per-make/model spec cards extracted from manuals (Bellmarine drives, Victron Cerbo, watermaker), with zone bands in SignalK canonical SI units. Kept private: the source manuals are copyrighted |

### SignalK Plugins

Run inside the SignalK server itself — alarms and data feeds that don't wait for an
agent. Published on npm under [@sailingnaturali](https://www.npmjs.com/org/sailingnaturali).

| Repo | npm | What it does |
|------|-----|-------------|
| [signalk-ntfy-relay](https://github.com/sailingnaturali/signalk-ntfy-relay) | [`@sailingnaturali/signalk-ntfy-relay`](https://www.npmjs.com/package/@sailingnaturali/signalk-ntfy-relay) | Push SignalK notifications (alarms) straight to your phone via [ntfy](https://ntfy.sh) — zero dependencies, edge-triggered, severity-aware |
| [signalk-currents](https://github.com/sailingnaturali/signalk-currents) | [`@sailingnaturali/signalk-currents`](https://www.npmjs.com/package/@sailingnaturali/signalk-currents) | Publish CHS/NOAA tidal-current predictions to SignalK — interpolated `environment.current` at the vessel plus a `/currents` slack/flood/ebb event series, for a configured station list |
| [signalk-depth-offsets](https://github.com/sailingnaturali/signalk-depth-offsets) | [`@sailingnaturali/signalk-depth-offsets`](https://www.npmjs.com/package/@sailingnaturali/signalk-depth-offsets) | Derive `environment.depth.belowKeel` and `belowSurface` from a DBT-only sounder using measured transducer offsets — zero dependencies |

The ship's log itself runs here too: [meri-imperiumi/signalk-logbook](https://github.com/meri-imperiumi/signalk-logbook)
— adopted, not rebuilt. Use existing tools before building your own.

### Agents & tooling

| Repo | What it does |
|------|-------------|
| [naturali-agents](https://github.com/sailingnaturali/naturali-agents) | Hermes Agent skills for Navigator, Engineer, and Logbook — prompts, bridge scripts, and HA voice integration |
| [vault-search](https://github.com/sailingnaturali/vault-search) | Local-first hybrid search over the Markdown vaults — BM25 + on-device embeddings (fastembed/ONNX) in one SQLite file, RRF-fused. The retrieval engine behind pilotbook-mcp's plain-language anchorage search |
| [claude-skills](https://github.com/sailingnaturali/claude-skills) | Claude Code skills from this build — `/plugin marketplace add sailingnaturali/claude-skills` |

---

## How it fits together

Three replaceable layers. An agent reads vessel + environment data through MCP
servers, reasons over it, and speaks back through Home Assistant.

```
  Interaction        Home Assistant
                       voice in ──MQTT──▶  │   ◀──MQTT── Piper TTS out
                                           ▼
  Agent runtime      Hermes Agent (Mac Studio)
                       Claude API (Sonnet) — local model deferred
                                           │
                      ┌──────────┬─────────┼──────────┬──────────┬─────────┬────────────┐
  Tool surface        ▼          ▼         ▼          ▼          ▼         ▼            ▼
                   signalk    weather  currents  pilotbook   colregs   logbook   vessel-knowledge
                    -mcp       -mcp      -mcp      -mcp        -mcp      -mcp          -mcp
                      │          │         │          │          │         │            │
  Data / vaults    SignalK   Open-Meteo  CHS +   pilotbook-  colregs-  signalk-  vessel-knowledge-
                    bus       NDBC        NOAA    vault       vault    logbook         vault
                                                                      (on boat)
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
