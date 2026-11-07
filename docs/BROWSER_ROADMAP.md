# Moldova 4X Browser — GitHub implementation roadmap

Status: approved implementation plan
Working branch: `feature/browser-foundation`
Base branch: `Development`
Rule: additive changes only; preserve the existing OpenCiv3 project.

## Branch and pull-request strategy

- `Development`: current repository base and OpenCiv3 integration branch.
- `feature/browser-foundation`: browser foundation and initial documentation.
- Subsequent work uses focused branches such as `feature/browser-map`, `feature/game-clock`, `feature/economy-core`.
- Browser work lands through pull requests into `Development`; no direct pushes to the base branch for feature work.
- Keep existing OpenCiv3 workflows intact. Browser CI is a separate workflow with path filters.
- Never force-push or rewrite repository history.

## Milestones

### M0 — Browser foundation
Deliverables:
- Root pnpm workspace metadata, without changing existing .NET/Godot build commands.
- `apps/browser`: React + TypeScript + Vite shell.
- `packages/game-engine`, `packages/game-data`, `packages/shared` package boundaries.
- Browser-specific CI: install, lint, typecheck, unit tests and production build.
- Browser-specific README and third-party notices.
- Minimal responsive shell with a working route and health/status panel.

Acceptance:
- `pnpm install --frozen-lockfile` works.
- Browser app starts in development and produces a production build.
- Existing OpenCiv3 workflow remains unchanged.
- Browser CI runs only when relevant paths change (plus manual dispatch if useful).

### M1 — Administrative map
Deliverables:
- MapLibre integration with Vite worker configured for production.
- Versioned GeoJSON and a geography manifest (source/licence/date/attribution).
- Regional map view, zoom/pan, city labels, region selection and detail panel.
- Explicit data model for territorial status and game control/influence.
Acceptance:
- Region selection works with mouse and touch.
- No API key is required for the local fixture.
- Map attribution and data provenance are visible.
- WebGL/tile failure has a usable fallback.

### M2 — Character and save foundation
Deliverables:
- Free-role character creation.
- New game with fixed scenario date and reproducible seed.
- GameState schema and Zod validation.
- IndexedDB save slots using Dexie.
- Versioned save metadata and safe load validation.
Acceptance:
- New game, save, reload and continue work.
- Invalid or incompatible save files fail safely without destroying other save slots.

### M3 — Economy core
Deliverables:
- Private and public budgets.
- Cashflow, basic companies, employment, production and recurring costs.
- Region economic indicators.
- Daily/monthly simulation updates.
Acceptance:
- All monetary calculations use explicit currency units.
- Invariants prevent negative/duplicated ledger postings.
- Tests cover income, expenses, bankruptcy and deterministic replays.

### M4 — Politics, factions and NPCs
Deliverables:
- Government, parliament, parties and public support.
- Fictional clans and organisations.
- NPC goals, relationships, utility scoring and event memory.
- Player affiliations and career progression.
Acceptance:
- NPCs make autonomous and reactive decisions.
- Same seed and same command sequence produce identical results.
- No LLM/API dependency is required for NPC simulation.

### M5 — Hybrid clock and events
Deliverables:
- Pause, 1x/2x/4x speed and bounded tick accumulator.
- Hourly/daily/monthly simulation cadence.
- Event triggers, choices, effects and event history.
Acceptance:
- Background-tab throttling does not cause huge jumps.
- Pausing freezes simulation.
- Event choices are validated and persist after reload.

### M6 — Diplomacy and strategic conflicts
Deliverables:
- Regional relations, trade agreements and diplomatic pressure.
- Abstract military forces, readiness and logistics.
- Turn-based strategic crisis/conflict resolution.
Acceptance:
- Conflict phases are deterministic and affected by readiness, supply, terrain context and morale.
- The rest of the hybrid simulation continues according to defined pause/event rules.

### M7 — MVP release
Deliverables:
- Tutorial and onboarding.
- Responsive desktop/Android UI.
- Performance and accessibility pass.
- Save migration tests.
- Release notes and Netlify deployment.
Acceptance:
- End-to-end new game → play → save → reload path passes on desktop and Android browser.
- All critical CI checks pass.
- Data and third-party notices are complete.

## Initial GitHub issue backlog

Create issues in this order (labels can be added after a label taxonomy is agreed):

1. M0: Add root pnpm workspace and browser package boundaries.
2. M0: Scaffold React + TypeScript + Vite browser client.
3. M0: Add browser-specific GitHub Actions CI.
4. M0: Add browser README, third-party notices and contributor instructions.
5. M1: Add MapLibre GL JS with Vite worker setup.
6. M1: Define GeoJSON region schema and geography provenance manifest.
7. M1: Build responsive administrative map and region detail panel.
8. M2: Define GameState, Character and SaveGame schemas.
9. M2: Implement seeded new-game creation.
10. M2: Implement versioned IndexedDB save slots.
11. M3: Implement ledger-based economy core.
12. M4: Implement faction and utility-AI core.
13. M5: Implement deterministic hybrid clock.
14. M5: Implement event triggers and player decisions.
15. M6: Implement diplomacy and strategic conflict phases.
16. M7: Add Playwright gameplay path and Android viewport checks.

## Proposed GitHub CI boundaries

Create `.github/workflows/browser-ci.yml` separately from `export-c7.yml`.

Trigger on:
- pull requests targeting `Development` when browser/workspace paths change;
- pushes to browser feature branches and `Development` when browser/workspace paths change;
- manual dispatch.

Do not require the browser workflow to check out the OpenCiv3 asset submodules or Git LFS. Do not edit the existing Godot workflow as part of M0.

## First implementation PR

The first PR should be deliberately small:
1. Browser workspace and app scaffold.
2. Pure game-engine package scaffold.
3. Browser CI workflow.
4. Browser-specific README and notices.
5. Minimal app shell, no gameplay claims yet.

Do not include the complete game map, real-world datasets, NPC simulation, or economy in the foundation PR.

## Definition of done for every issue

- Acceptance criteria are testable.
- Scope is limited to one subsystem.
- Tests are added or updated.
- Lint, typecheck, tests and production build pass.
- No existing OpenCiv3 paths/assets/workflows were removed or silently changed.
- Third-party code/data/assets have source and licence records.
- PR links to its issue and includes screenshots for UI changes.
