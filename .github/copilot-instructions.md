<!-- Guidance for AI coding agents working in the UI-TARS-desktop monorepo -->

# UI-TARS-desktop — Copilot / Agent Guidance

Purpose: help AI coding agents be immediately productive in this monorepo (Electron desktop + multimodal agent stack).

**Big picture**
- **Monorepo layout**: the Electron desktop app lives in `apps/ui-tars/`. Shared libraries, MCP servers and agent cores live under `packages/` and `multimodal/`.
- **Major components**: `apps/ui-tars` (Electron + React + `electron-vite`), `packages/agent-infra` (MCP client/servers), `multimodal/tarko` (agent core and LLM integrations), `packages/ui-tars/*` (SDK, operators, IPC).
- **Why**: separation targets reusability — UI/electron concerns are isolated, operator/SDK logic is packaged for reuse across desktop and other runtimes.

**Quick developer workflows**
- Install & bootstrap: run `pnpm i` at repo root. Repo uses `pnpm` and `turbo` for workspace orchestration. Node >= 20 is required ([package.json](package.json)).
- Run the desktop app (dev): from repo root run `pnpm run dev:ui-tars` (root script runs Turbo). Or `cd apps/ui-tars && pnpm run dev` for direct control.
- Build the desktop app: `cd apps/ui-tars && pnpm run build` (uses `electron-vite` + `electron-forge`). Packaging: `pnpm --filter ui-tars-desktop build` or use the app `publish`/`make` scripts in [apps/ui-tars/package.json](apps/ui-tars/package.json).
- Typecheck: `cd apps/ui-tars && npm run typecheck` (runs both node/web tsc targets).
- Unit tests: `pnpm test` (root) or `cd apps/ui-tars && pnpm run test` (uses `vitest`). E2E: `cd apps/ui-tars && pnpm run test:e2e` (Playwright).
- Debugging: `cd apps/ui-tars && npm run debug` to start dev with `--remote-debugging-port=9222`.

**Project-specific conventions & patterns**
- Packages reference local modules with `workspace:*` in `package.json` (e.g., `@ui-tars/sdk`). Prefer using workspace resolution when editing imports.
- Electron uses `electron-vite` + `electron-forge`. Configuration entry points: [apps/ui-tars/electron.vite.config.ts](apps/ui-tars/electron.vite.config.ts) and [apps/ui-tars/forge.config.ts](apps/ui-tars/forge.config.ts).
- Built outputs: `dist/` and `out/` under `apps/ui-tars`; `asar` is used in packaging (see `asar:analyze`).
- Environment flags: many scripts rely on `cross-env NODE_ENV=production` or `CI=e2e` — set them consistently when invoking build/test CLI.

**Integration points & external dependencies**
- MCP (Modular Connector Protocol): check `packages/agent-infra` and `multimodal/omni-tars/mcp-agent` for server/client patterns and message/event flows.
- Model providers: integrations are configured at runtime (examples in docs/quick-start.md). Typical providers: Volcengine, Anthropic. See top-level `README.md` and `docs/quick-start.md` for runtime flags and examples.
- Native macOS permissions & screen capture: `@computer-use/*` packages and `@computer-use/node-mac-permissions` are used in `apps/ui-tars` for privileged features.

**Where to look for examples**
- App entry / electron hooks: [apps/ui-tars/src/main/](apps/ui-tars/src/main/) and renderer under `apps/ui-tars/src/renderer/`.
- IPC & operators: [packages/ui-tars/electron-ipc/README.md](packages/ui-tars/electron-ipc/README.md) and `packages/ui-tars/operators/*`.
- Tests and CI: see `vitest.config.mts` and [apps/ui-tars/vitest.config.mts](apps/ui-tars/vitest.config.mts) for test patterns.

**Behavior to preserve when changing code**
- Keep the separation between UI (Electron/renderer), main process (native integration, IPC), and operator/SDK logic (packages). Avoid moving operator logic into the renderer.
- Respect workspace package names and exports — changing public API requires updates across multiple `packages/*` consumers.

**Quick examples agents should use**
- Run dev app: `pnpm i && pnpm run dev:ui-tars` (root).
- Build & package mac (ARM): `cd apps/ui-tars && pnpm run build && pnpm run publish:mac-arm64`.
- Run unit tests for app-only changes: `cd apps/ui-tars && pnpm run test`.

If anything here is unclear or you'd like more examples (e.g., common PR/code patterns, test patterns, or a short checklist for making UI/API changes), tell me which area to expand and I will iterate.
