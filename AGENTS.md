# AGENTS.md

This file provides guidance for agentic coding agents working in the UI-TARS-desktop monorepo.

## Project Overview

UI-TARS-desktop is a monorepo containing an Electron desktop application with multimodal agent capabilities. The project uses a modern TypeScript stack with pnpm workspaces and Turbo for orchestration.

**Architecture:**
- `apps/ui-tars/` - Main Electron desktop app (React + electron-vite)
- `packages/` - Shared libraries, MCP servers, and agent infrastructure
- `multimodal/` - Agent cores and LLM integrations

## Build & Development Commands

### Root Level Commands
```bash
# Install dependencies and bootstrap
pnpm i

# Run desktop app in development
pnpm run dev:ui-tars

# Format code
pnpm run format

# Lint and fix code
pnpm run lint

# Run all tests
pnpm test

# Run tests with coverage
pnpm run coverage

# Run benchmark tests
pnpm run test:bench
```

### App-Specific Commands (apps/ui-tars/)
```bash
# Development
pnpm run dev              # Start dev server
pnpm run debug            # Start with remote debugging on port 9222
pnpm run dev:w            # Start with watch mode

# Type checking
pnpm run typecheck        # Run both node and web typecheck
pnpm run typecheck:node   # Typecheck main process only
pnpm run typecheck:web    # Typecheck renderer only

# Building
pnpm run build            # Full build with typecheck
pnpm run build:dist       # Build without typecheck
pnpm run clean            # Clean build outputs

# Testing
pnpm run test             # Run unit tests
pnpm run test:e2e         # Run Playwright E2E tests
pnpm run coverage         # Run tests with coverage

# Packaging
pnpm run package          # Package the app
pnpm run make             # Make distributables
pnpm run publish:mac-arm64 # Build and publish for macOS ARM64
pnpm run publish:mac-x64   # Build and publish for macOS x64
pnpm run publish:win32    # Build and publish for Windows
```

### Running Single Tests
```bash
# Run a specific test file
pnpm vitest run path/to/test.test.ts

# Run tests in watch mode for a specific file
pnpm vitest path/to/test.test.ts

# Run tests with coverage for a specific file
pnpm vitest run --coverage path/to/test.test.ts
```

## Code Style Guidelines

### Import Organization
- Use workspace resolution for local packages (`@ui-tars/*`)
- Import order: Node modules → workspace packages → relative imports
- Use `@ui-tars/*` prefixes for workspace packages
- Import React explicitly in JSX files when needed

### TypeScript Configuration
- Strict mode enabled with `noUncheckedIndexedAccess`
- Target ES2022 with ESNext modules
- Use `composite: true` for project references
- Consistent casing enforced (`forceConsistentCasingInFileNames`)

### Formatting (Prettier)
- Single quotes for strings
- Trailing commas for all
- 2-space indentation
- Semicolons required
- Arrow function parentheses always

### ESLint Rules
- TypeScript rules are relaxed (no explicit return types, unused vars allowed)
- React rules are lenient (no display-name required, prop-types off)
- Import rules are disabled for workspace flexibility
- No shadow restrictions

### File Naming Conventions
- Use kebab-case for files and directories
- TypeScript files: `.ts` for types, `.tsx` for React components
- Test files: `.test.ts` or `.spec.ts` suffix
- Config files: `.config.ts` or `.config.mjs`

### Component Patterns
- Use functional components with hooks
- Prefer custom hooks for complex logic
- Use Zustand for state management
- Follow React hooks rules strictly

### Error Handling
- Use try-catch blocks for async operations
- Implement proper error boundaries in React
- Log errors with appropriate context
- Return consistent error types

### IPC Patterns
- Use `@ui-tars/electron-ipc` for main/renderer communication
- Define routes in `ipcRoutes/` directory
- Type-safe IPC with generated router types
- Separate concerns between main and renderer processes

## Testing Guidelines

### Unit Tests
- Use Vitest as the test runner
- Test files should be co-located with source files
- Use `describe` blocks for test organization
- Mock external dependencies appropriately

### E2E Tests
- Use Playwright for end-to-end testing
- Tests located in `e2e/` directory
- Use CI environment variable for E2E builds
- Test critical user workflows

### Coverage
- Aim for >80% coverage on new code
- Use Istanbul coverage provider
- Coverage reports include text, JSON, HTML, and LCOV formats
- Exclude test files and certain packages from coverage

## Development Workflow

### Before Making Changes
1. Run `pnpm i` to ensure dependencies are current
2. Check `pnpm run typecheck` passes
3. Run `pnpm run lint` to check code style
4. Run `pnpm test` to ensure tests pass

### After Making Changes
1. Run `pnpm run format` to format code
2. Run `pnpm run lint --fix` to fix linting issues
3. Run `pnpm run typecheck` to verify types
4. Run relevant tests: `pnpm test` or specific test files
5. For app changes, test with `pnpm run dev`

### Package Management
- Use `pnpm` for all package operations
- Workspace packages use `workspace:*` versioning
- Prefer workspace resolution over npm registry
- Update dependencies carefully due to Electron constraints

## Environment Configuration

### Required Environment Variables
- `NODE_ENV=production` for production builds
- `CI=e2e` for E2E test builds
- Model provider configurations (see docs/quick-start.md)

### Build Targets
- Node.js >= 20 required
- Electron 34.1.1 (fixed version)
- TypeScript 5.7.2+
- Vite 6.1.0+ for bundling

## Key Integration Points

### MCP (Modular Connector Protocol)
- Client/server patterns in `packages/agent-infra/`
- Message/event flows documented in package READMEs
- Use shared types from `@ui-tars/shared`

### Model Providers
- Configured at runtime
- Support for Volcengine, Anthropic, and others
- See `docs/quick-start.md` for configuration examples

### Native Integration
- macOS permissions via `@computer-use/node-mac-permissions`
- Screen capture capabilities
- Native automation through operators

## Common Patterns

### Creating New Operators
1. Extend base operator class in `packages/ui-tars/operators/`
2. Implement required interface methods
3. Add tests in co-located test files
4. Update exports in package.json

### Adding IPC Routes
1. Create route file in `apps/ui-tars/src/main/ipcRoutes/`
2. Define typed handlers using `@ui-tars/electron-ipc`
3. Export route and add to main router
4. Add tests for route handlers

### Package Development
1. Use `workspace:*` for local dependencies
2. Export public API via package.json exports
3. Include TypeScript definitions
4. Add comprehensive README with usage examples

## Debugging Tips

### Main Process Debugging
- Use `pnpm run debug` for Chrome DevTools
- Remote debugging on port 9222
- Check `electron-log` for runtime logs

### Renderer Process Debugging
- Use browser DevTools in development
- React DevTools for component inspection
- Network tab for IPC communication

### Test Debugging
- Use Vitest's watch mode for iterative testing
- Add `console.log` statements in test files
- Use VS Code debugger for test files

## Security Considerations

- Never commit secrets or API keys
- Use environment variables for sensitive data
- Validate all IPC inputs
- Follow Electron security best practices
- Use context isolation in preload scripts