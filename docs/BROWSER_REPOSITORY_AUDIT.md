# Moldova 4X Browser — repository audit

Audit date: 2026-11-07
Audited ref: `Development`
Audited commit: `376ff4dc650e9977284d8e5c0df523d5a5e318f8`

## Executive decision

Keep the existing OpenCiv3 project intact and add the browser game as an independent application under `apps/browser/`, with a separate TypeScript simulation engine under `packages/game-engine/`. Do not port or rewrite the Godot/C# engine as part of the first browser milestone.

## Verified repository facts

- Repository: `bossartemii/Moldova-4x`; public; default branch `Development`.
- The audited tree contains 647 entries and GitHub reports a repository size of approximately 81 MB.
- Existing project is OpenCiv3: Godot 4.4.1, .NET 8/C#, Lua.
- Existing major directories include `C7/`, `C7Engine/`, `EngineTests/`, `Blast/`, `ConvertCiv3Media/`, `QueryCiv3/`, and `_Console/`.
- Existing C# projects target `net8.0`; the Godot project identifies itself as OpenCiv3 version `0.3 Dutch 2`.
- Existing CI is centered on exporting/testing the Godot project. `.github/workflows/export-c7.yml` is configured for `Development`, pull requests into `Development`, and manual exports.
- Existing submodules are `C7/Assets` (SSH URL) and `C7/addons/markdownlabel`.
- Root `LICENSE` is the OpenCiv3 MIT license. `NOTICE.md` explicitly says media assets are not covered by that MIT declaration and points to separate asset credits/licences.
- No browser app, Node package manifest, pnpm workspace, or TypeScript game engine was found in the audited tree.
- The repository has no open GitHub issues at the time of this audit.

## Risks and constraints

### R1 — Build-system collision
The current CI is a Godot/.NET pipeline. Browser CI must be a separate workflow with path filters and must not replace or weaken `export-c7.yml`.

### R2 — Asset and licence contamination
Do not copy OpenCiv3/Civilization III-derived media, fonts, textures, or assets into the browser game unless the exact asset licence and redistribution rights are verified. Keep the existing `NOTICE.md`, `LICENSE`, asset-credit generator, and submodules untouched. Add browser-specific third-party notices when dependencies/data are introduced.

### R3 — Submodule and checkout assumptions
The current OpenCiv3 workflows explicitly check out submodules and Git LFS. Browser CI should not need the Godot asset submodule or LFS payloads; use checkout settings scoped to the browser job.

### R4 — Map rendering is not map data hosting
MapLibre GL JS renders map styles and vector/raster data; it does not itself provide a production tile service or authoritative administrative boundaries. The browser MVP must identify the tile/style provider, attribution, usage limits, data source, data date, and licence. Do not assume public OpenStreetMap tile servers are an unrestricted production CDN.

### R5 — Geographic accuracy and contested status
Store administrative geography as versioned GeoJSON with source metadata. Model Transnistria and other contested/complex territorial statuses explicitly rather than silently presenting disputed administration as ordinary uncontested state control.

### R6 — Simulation determinism and persistence
The game engine must not depend on React, MapLibre, DOM APIs, or IndexedDB. Use seeded randomness, validated commands, versioned save files, and deterministic simulation ticks. Do not implement offline catch-up in the first MVP.

### R7 — Browser performance and mobile
Test touch interactions, resize, low-memory devices, reduced motion, and WebGL availability. Provide a clear fallback/error state if WebGL or map tiles are unavailable.

## Repository quality score (current repository, not the future browser app)

| Dimension | Score | Finding |
|---|---:|---|
| Workflow | 16/20 | Existing OpenCiv3 CI/build workflow is present; no browser workflow or product backlog yet. |
| Architecture | 13/20 | Existing Godot engine is separated from UI, but it is not reusable as a browser runtime without a separate implementation. |
| Gotchas | 11/15 | Licence and asset notices exist; map/data hosting and browser separation still need explicit rules. |
| Concision | 10/15 | README is upstream OpenCiv3 documentation and does not explain the Moldova browser product. |
| Freshness | 12/15 | Audited branch head is recent; no browser-specific dependencies or tooling exist. |
| Actionability | 8/15 | No issues/milestones or browser build instructions yet. |
| **Total** | **70/100** | Existing upstream codebase is preserved and build-oriented; Moldova browser work has not started. |

## Safe migration boundary

Allowed:
- Add `apps/browser/**`, `packages/game-engine/**`, `packages/game-data/**`, `packages/shared/**`, browser documentation, and a separate browser CI workflow.
- Add root workspace metadata only after checking it does not change existing .NET/Godot commands.
- Add browser-specific licences/notices and data attribution files.

Not allowed without a separate explicit decision:
- Delete, rename, or move any existing OpenCiv3 path.
- Change root `LICENSE`, `NOTICE.md`, `.gitmodules`, Godot project files, C# projects, asset credits, or existing export workflow.
- Reuse OpenCiv3 media in the browser application without asset-by-asset licence verification.
- Force-push, rewrite history, or merge the new browser branch into `Development`.

## Next gate

Build the browser foundation on an isolated feature branch. The first pull request should contain only browser documentation, workspace/app scaffolding, a separate CI workflow, and a visible map-shell prototype. Keep all changes additive and verify the original OpenCiv3 tree remains unchanged.
