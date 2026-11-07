# Moldova 4X Browser — technical specification

Status: approved product direction / implementation baseline
Date: 2026-11-07

## Product decisions

- Setting: modern, initial scenario year 2026.
- Player: free role; the player controls a character, not necessarily a state.
- Difficulty: realistic.
- Time model: hybrid; continuous simulation with turn-based strategic conflict resolution.
- Map: administrative regions and cities, not hexes.
- NPC behaviour: mixed autonomous and reactive.
- Geography: Moldova is simulated in detail; Romania, Ukraine, and surrounding regional actors are represented at a more abstract level in the MVP.
- Initial platform: responsive browser app, desktop-first layout with Android touch support.
- Initial mode: single-player, local saves; no account or online multiplayer requirement.

## Technical principles

1. Simulation rules are pure TypeScript and independent from React, MapLibre, browser APIs, and persistence.
2. UI issues validated game commands; it does not mutate game state directly.
3. Random outcomes use a seeded PRNG stored in the save.
4. Every save has a schema version and explicit migration path.
5. All geography, scenario content, and NPC data are data-driven and versioned.
6. External map styles/tiles and real-world datasets must have documented source, attribution, licence, date, and usage constraints.
7. The OpenCiv3 project remains an independent legacy/upstream subtree and is not a runtime dependency of the browser game.

## Proposed workspace

```text
apps/
  browser/                 # React + Vite client
packages/
  game-engine/             # pure deterministic simulation
  game-data/               # scenario, geography, factions and event data
  shared/                  # shared types and runtime schemas
docs/
  BROWSER_REPOSITORY_AUDIT.md
  BROWSER_TECHNICAL_SPEC.md
  BROWSER_ROADMAP.md
```

The existing root folders (`C7`, `C7Engine`, `EngineTests`, `Blast`, `ConvertCiv3Media`, `QueryCiv3`, `_Console`) are not moved or modified by this architecture.

## Browser client

- React + TypeScript + Vite.
- MapLibre GL JS for the administrative map.
- Zustand for presentation/UI state only; game state transitions go through engine commands.
- Zod for runtime validation of commands, content and save payloads.
- Dexie/IndexedDB for local save storage.
- Vitest for engine and UI unit tests.
- Playwright for browser end-to-end tests.

MapLibre GL JS v6 uses an external worker asset. In Vite, configure the worker URL using the official Vite worker URL pattern and verify production builds; a dev-only working map is not sufficient.

## Game engine modules

- `clock`: simulation time, pause and speed.
- `world`: region, city, infrastructure and territorial status.
- `economy`: public/private budgets, production, employment, prices and trade.
- `characters`: player/NPC attributes, skills, career and relationships.
- `factions`: parties, clans, companies and civil organisations.
- `politics`: institutions, government, parliament, elections, laws and public support.
- `military`: abstract forces, readiness, supply and turn-based strategic conflicts.
- `diplomacy`: regional relations, treaties, trade and diplomatic pressure.
- `intelligence`: abstract information quality, secrecy and counter-intelligence.
- `events`: event conditions, choices, effects and history.
- `ai`: utility scoring, goals, memory of key events and action selection.
- `simulation`: deterministic tick orchestration and command application.

## Initial domain model

Core aggregates:
- `GameState`: seed, schema version, simulation date, player, world and subsystem state.
- `Character`: identity, skills, money, reputation, influence, affiliations and relationships.
- `Region`: stable ID, name, geometry reference, administrative status, population and economic indicators.
- `Faction`: type, goals, resources, influence, relations and decision policy.
- `GameEvent`: trigger, options, effects, visibility and resolution state.
- `SaveGame`: metadata, versioned state, content version and checksum.

Use stable IDs, integer minor currency units where practical, explicit units for all quantities, and no hidden floating-point money arithmetic.

## Time and simulation cadence

Initial design target (tunable during playtesting):
- 1x: one in-game hour per 5 real seconds.
- 2x and 4x speeds.
- Pause always available.
- Hourly tick for time-sensitive queues and actions.
- Daily tick for production, payroll, consumption and routine NPC actions.
- Monthly tick for macroeconomic indicators, political cycles and budget summaries.
- Strategic conflicts resolve through explicit player/AI decision phases.
- No offline catch-up in MVP; pausing/closing the browser stops the local simulation.

Avoid setInterval-driven state mutation. Use an elapsed-time accumulator with bounded catch-up steps so background-tab throttling does not cause giant simulation jumps.

## NPC architecture

Use deterministic utility AI/state machines in the MVP, not an LLM as the authority for decisions.
Each NPC faction has:
- goals and priorities;
- resources and constraints;
- relationships and risk tolerance;
- a small memory of important events;
- candidate actions with preconditions, cost, expected utility and cooldown.

NPCs act autonomously on scheduled ticks and react to player actions/events. Use stable tie-breaking based on the game PRNG.

## Map and data policy

- GeoJSON is the initial exchange format for region boundaries.
- Keep data source, licence, retrieval date, version and attribution in a manifest next to the data.
- Separate territorial geometry from legal/administrative status and from game control/influence.
- Represent Transnistria as a distinct territorial/governance status with explicit uncertainty/context.
- Do not bundle copyrighted OpenCiv3/Civ3 media in the browser application.
- MapLibre is a renderer, not a tile hosting service. Choose a compliant tile/style source or package suitable data (e.g. PMTiles) before production.

## Save and command architecture

- UI submits commands such as `CREATE_COMPANY`, `PROPOSE_LAW`, `JOIN_FACTION`, `ADVANCE_TIME`, `SELECT_REGION`.
- Engine validates preconditions and applies state changes through a deterministic reducer/transaction boundary.
- Save payload includes engine schema version, content version, PRNG state and simulation clock.
- Autosave is debounced and transactional in IndexedDB.
- Import/export save file is a later MVP+ capability; local save slots are required for MVP.

## Out of scope for first MVP

- Multiplayer, accounts and server authority.
- LLM-controlled NPC decisions.
- Real-time tactical combat.
- 3D map/terrain.
- Full simulation of neighbouring countries.
- Offline simulation while the browser is closed.
- Reuse of OpenCiv3 assets without separate clearance.

## MVP acceptance criteria

A user can create a free-role character, start a seeded 2026 scenario, navigate/select Moldova's administrative regions, inspect region details, advance/pause simulation, see a working basic economy and autonomous/reactive NPC factions, resolve a small strategic crisis, save, reload, and continue the same deterministic game.
