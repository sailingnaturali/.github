# Sailing Naturali

A tech exec uses AI to build the kind of business AI can't deliver.

All-electric catamaran charter operation in the Pacific Northwest — Gulf Islands and San Juan Islands. AI runs the operations. Humans deliver the experience.

📺 [@sailingnaturali](https://www.youtube.com/@sailingnaturali) on YouTube · [sailingnaturali](https://sailingnaturali.substack.com) on Substack · [sailingnaturali.com](https://sailingnaturali.com) · [@sailingnaturali](https://www.instagram.com/sailingnaturali) on Instagram · [@sailingnaturali](https://www.tiktok.com/@sailingnaturali) on TikTok

---

## Boat Agent Stack

On-boat AI agent system — Navigator, Engineer, Logbook — wired to the vessel through SignalK and MCP servers. The agents are the product; the runtime underneath is replaceable and has already been replaced once.

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
| [club-moorage-mcp](https://github.com/sailingnaturali/club-moorage-mcp) | Find and overnight-rank yacht-club outstations — RVYC member moorage with size, rafting, and booking rules, ranked against the forecast by reusing pilotbook-mcp's anchorage scorer |

### Vaults

Knowledge bases the MCP servers read from.

| Repo | What it holds |
|------|-------------|
| [colregs-vault](https://github.com/sailingnaturali/colregs-vault) | Navigation-rules content for colregs-mcp — rule prose, a curated requirements decision table, and PNW regime polygons. Public because the sources are reproducible public-domain law |
| [currents-vault](https://github.com/sailingnaturali/currents-vault) | Tidal-gate reference for currents-mcp — one file per PNW pass with CHS/NOAA station identity plus flood/ebb hazard notes (whirlpools, holes, sets), and a destination routing table. Public: station data is open government, hazards are paraphrased |
| `pilotbook-vault` *(private)* | Anchorage knowledge for pilotbook-mcp — 673 anchorages across 7 SalishSeaPilot books, with exposed-sector data and source page back-links. Kept private: the source pilot books are copyrighted |
| `vessel-knowledge-vault` *(private)* | Equipment cards for vessel-knowledge-mcp — per-make/model spec cards extracted from manuals (propulsion drives, Victron Cerbo, watermaker), with zone bands in SignalK canonical SI units. Kept private: the source manuals are copyrighted |

### SignalK Plugins

Run inside the SignalK server itself — alarms and data feeds that don't wait for an
agent. Published on npm under [@sailingnaturali](https://www.npmjs.com/org/sailingnaturali).

| Repo | npm | What it does |
|------|-----|-------------|
| [signalk-ntfy-relay](https://github.com/sailingnaturali/signalk-ntfy-relay) | [`@sailingnaturali/signalk-ntfy-relay`](https://www.npmjs.com/package/@sailingnaturali/signalk-ntfy-relay) | Push SignalK notifications (alarms) straight to your phone via [ntfy](https://ntfy.sh) — zero dependencies, edge-triggered, severity-aware |
| [signalk-currents](https://github.com/sailingnaturali/signalk-currents) | [`@sailingnaturali/signalk-currents`](https://www.npmjs.com/package/@sailingnaturali/signalk-currents) | Publish CHS/NOAA tidal-current predictions to SignalK — interpolated `environment.current` at the vessel plus a `/currents` slack/flood/ebb event series, for a configured station list |
| [signalk-depth-offsets](https://github.com/sailingnaturali/signalk-depth-offsets) | [`@sailingnaturali/signalk-depth-offsets`](https://www.npmjs.com/package/@sailingnaturali/signalk-depth-offsets) | Derive `environment.depth.belowKeel` and `belowSurface` from a DBT-only sounder using measured transducer offsets — zero dependencies |
| [signalk-dsc](https://github.com/sailingnaturali/signalk-dsc) | [`@sailingnaturali/signalk-dsc`](https://www.npmjs.com/package/@sailingnaturali/signalk-dsc) | Receive, log, and alert on DSC (VHF digital selective calling) calls — distress, urgency, safety, routine — from NMEA 0183 (`$CDDSC`/`$CDDSE`) and NMEA 2000 (PGN 129808) |
| [signalk-ais-distress](https://github.com/sailingnaturali/signalk-ais-distress) | [`@sailingnaturali/signalk-ais-distress`](https://www.npmjs.com/package/@sailingnaturali/signalk-ais-distress) | Alert on AIS distress beacons — SART, MOB, and EPIRB survival devices (MMSI 970/972/974) — with emergency notifications, chart markers, a forensic log, and a ship's-log entry |
| [signalk-journey-replay](https://github.com/sailingnaturali/signalk-journey-replay) | [`@sailingnaturali/signalk-journey-replay`](https://www.npmjs.com/package/@sailingnaturali/signalk-journey-replay) | Replay published voyage data ([journey-data](https://github.com/sailingnaturali/journey-data) archives) on any SignalK server — download a trip, press play, timestamps rebased to now |
| [signalk-equipment-registry](https://github.com/sailingnaturali/signalk-equipment-registry) | [`@sailingnaturali/signalk-equipment-registry`](https://www.npmjs.com/package/@sailingnaturali/signalk-equipment-registry) | Serve a vessel equipment registry — manufacturer/model/serial per installed instance, plus the SignalK paths each owns — at `resources/equipment`, the identity slot SignalK's schema lacks |

### Agents & tooling

| Repo | What it does |
|------|-------------|
| [naturali-agents](https://github.com/sailingnaturali/naturali-agents) | Agent skills for Navigator, Engineer, and Logbook — shared persona, prompts, the Poseidon daemon, and HA voice integration |
| [vault-search](https://github.com/sailingnaturali/vault-search) | Local-first hybrid search over the Markdown vaults — BM25 + on-device embeddings (fastembed/ONNX) in one SQLite file, RRF-fused. The retrieval engine behind pilotbook-mcp's plain-language anchorage search |
| [marine-forecast](https://github.com/sailingnaturali/marine-forecast) | Shared Open-Meteo marine + wind forecast fetch, extracted from weather-mcp so it and pilotbook-mcp compose one forecast implementation instead of duplicating it |
| [signalk-distress-core](https://github.com/sailingnaturali/signalk-distress-core) | Shared distress plumbing ([`@sailingnaturali/signalk-distress-core`](https://www.npmjs.com/package/@sailingnaturali/signalk-distress-core)) — event store, chart-marker builder, spoken/logbook rendering, and notifications — extracted from signalk-dsc so it and signalk-ais-distress share one implementation |
| [naturali-mcp-netutil](https://github.com/sailingnaturali/naturali-mcp-netutil) | Tiny shared network helpers for the MCP fleet — resolves `.local` mDNS hosts to IPv4 at the URL boundary so SignalK reads don't hang on macOS Happy-Eyeballs |
| [journey-data](https://github.com/sailingnaturali/journey-data) | Real voyage data from S/V Naturali as SignalK delta archives — capture, scrub, and publish CLI; the archives signalk-journey-replay plays back |
| [claude-skills](https://github.com/sailingnaturali/claude-skills) | Claude Code skills from this build — `/plugin marketplace add sailingnaturali/claude-skills` |

### What the boat runs on (adopted, not built)

The rule here is **adopt before build** — the repos above exist only because nothing
else filled the gap. Most of what makes this boat work is other people's open source:

| Project | Role aboard |
|---------|-------------|
| [Signal K](https://github.com/SignalK/signalk-server) | The open marine data bus — every sensor, plugin, and agent reads and writes here |
| [Home Assistant](https://www.home-assistant.io) | Voice in/out, dashboards, and automations — the crew-facing surface |
| [signalk-logbook](https://github.com/meri-imperiumi/signalk-logbook) | The ship's log itself — logbook-mcp and signalk-dsc write to it rather than reinventing it |
| [signalk-tides](https://github.com/openwatersio/signalk-tides) | Tide extremes at the vessel — we send patches upstream instead of forking |
| [openmeteo-provider-plugin](https://github.com/SignalK/openmeteo-provider-plugin) | Free [Open-Meteo](https://open-meteo.com) forecasts into SignalK's weather API — we contributed the marine waves & swell (`water.*`) the point forecast was missing |
| [signalk-restricted-areas](https://github.com/dirkwa/signalk-restricted-areas) | ProtectedSeas marine-protected-area layers and geofence alerts — MPAs, rockfish conservation areas, whale slowdown zones |
| [Eclipse Mosquitto](https://mosquitto.org) | The MQTT spine — alarms, intents, and voice events between boat, agent host, and Home Assistant |
| [ntfy](https://ntfy.sh) | Push notifications to phones — the far end of signalk-ntfy-relay |
| [Whisper](https://github.com/rhasspy/wyoming-faster-whisper) / [Piper](https://github.com/rhasspy/piper) | Speech-to-text and text-to-speech for the voice pipeline, both running locally |
| [Ollama](https://github.com/ollama/ollama) | Local inference for small structured tasks — e.g. repairing model output into strict JSON |
| [Tailscale](https://tailscale.com) | Remote access to boat and shore hosts — WireGuard without the key ceremony |

When one of these falls short we file an issue or send a patch before writing a
replacement. A new repo in the tables above is the last resort.

### Travel storytelling

Off the boat — the trip-content side. Standalone and useful on their own.

| Repo | What it does |
|------|-------------|
| [flight-animator](https://github.com/sailingnaturali/flight-animator) | Animate multi-stop flight routes on a dark MapLibre map — great-circle arcs, animated trail, no server or sign-in. Built to screen-record and narrate for the Amsterdam boat-build trip videos · [flights.sailingnaturali.com](https://flights.sailingnaturali.com) |
| [flighty-mcp](https://github.com/sailingnaturali/flighty-mcp) | Read-only MCP server exposing personal Flighty flight history as geo-ready legs (departure/arrival coordinates) — query by date, year, or flight number, and browse aggregate stats |

### Tides & currents

Open, standalone — usable in any app or agent.

| Repo | What it does |
|------|-------------|
| [slackwater-engine](https://github.com/sailingnaturali/slackwater-engine) | Pure-Swift offline harmonic tide & current engine — heights and the high/low turns for any minute, years ahead, with zero network. Faithful port of [openwaters/neaps](https://github.com/openwatersio/tide-predictor), validated against NOAA's own published predictions to a few minutes and centimetres. MIT |
| [slackwater-web](https://github.com/sailingnaturali/slackwater-web) | Offline tide predictions for the Salish Sea as an installable web app — no account, no network needed once loaded. The PWA face of slackwater-engine |
| [chs-constituents](https://github.com/sailingnaturali/chs-constituents) | Fit tidal-current harmonic constituents for Canadian (CHS) stations from IWLS predictions ([`@sailingnaturali/chs-constituents`](https://www.npmjs.com/package/@sailingnaturali/chs-constituents)) — out-of-sample validation, confidence tiering, per-gate training-window selection. A pipeline instead of a dataset: the CHS licence forbids redistributing derived data, so you run it yourself and offline prediction follows. MIT |
| [current-stations](https://github.com/sailingnaturali/current-stations) | NOAA CO-OPS tidal-current station data ([`@sailingnaturali/current-stations`](https://www.npmjs.com/package/@sailingnaturali/current-stations)) — fetcher, extractor, schema, and the API's undocumented behaviour. The public-domain constituent source behind signalk-currents' offline fallback |
| [station-corrections](https://github.com/sailingnaturali/station-corrections) | Everything the agencies got wrong about tide and current stations ([`@sailingnaturali/station-corrections`](https://www.npmjs.com/package/@sailingnaturali/station-corrections)) — friendly names, the water body a station actually sits on, search aliases, and positions corrected off dry land. Provider-agnostic, so NOAA, CHS and current stations share one vocabulary. Corrections are one-line pull requests. MIT |

---

## How it fits together

Three replaceable layers. An agent reads vessel + environment data through MCP
servers, reasons over it, and speaks back through Home Assistant.

```
  Interaction        Home Assistant
                       voice in ──MQTT──▶  │   ◀──MQTT── Piper TTS out
                                           ▼
  Agent runtime      Poseidon daemon (Mac Studio)
                       Claude today — local model when the boat can power it
                                           │
                     ┌──────────┬──────────┼──────────┬──────────┬─────────┬──────────────┬───────────────┐
  Tool surface       ▼          ▼          ▼          ▼          ▼         ▼              ▼               ▼
                  signalk    weather   currents   pilotbook   colregs   logbook   vessel-knowledge   outstations
                   -mcp       -mcp       -mcp       -mcp       -mcp      -mcp           -mcp            -mcp
                     │          │          │          │          │         │              │               │
  Data / vaults   SignalK  Open-Meteo    CHS +   pilotbook-  colregs-  signalk-   vessel-knowledge-   RVYC data
                    bus       NDBC       NOAA       vault      vault    logbook         vault         (bundled)
                                                                       (on boat)
```

Each MCP server runs standalone and is independently useful — point your own agent
at any one of them. The vaults are plain Markdown/GeoJSON the servers read at
startup (`PILOTBOOK_VAULT_PATH`, `COLREGS_VAULT_PATH`, `CURRENTS_VAULT_PATH`,
`VESSEL_KNOWLEDGE_VAULT_PATH`). currents-mcp reads live currents from CHS/NOAA via
signalk-currents and gate/hazard reference from currents-vault.
vessel-knowledge-mcp also runs the loop in reverse: its build-time CLI pushes alarm-zone
metadata *into* SignalK, so the bus raises notifications on its own and the agent only
explains them.

## Getting started

Start at **[naturali-agents](https://github.com/sailingnaturali/naturali-agents)** — the
agent skills, shared persona, and daemon that wire the MCP servers into a working crew.
It ships a mock SignalK server, so the loop runs on a laptop with no boat:

```bash
git clone https://github.com/sailingnaturali/naturali-agents
cd naturali-agents
python dev/mock-signalk.py            # serves a Boundary Pass scenario on :8765
export SIGNALK_URL=http://localhost:8765
uv run scripts/briefing.py --dry-run  # real tides + mock vessel → the Navigator's morning prompt
```

Each repo's README carries its own install, env vars, and tool surface. Every MCP
server is a `uv`-runnable Python package with its own test suite.

---

Built in public. MIT licensed. Show the receipts. The build log lives on the
[engineering blog](https://engineering.sailingnaturali.com)
([source](https://github.com/sailingnaturali/engineering)), and the main site
at [sailingnaturali.com](https://sailingnaturali.com)
([source](https://github.com/sailingnaturali/web)).
