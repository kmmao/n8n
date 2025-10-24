# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in the n8n repository.

## Project Overview

n8n is a workflow automation platform written in TypeScript, using a monorepo
structure managed by pnpm workspaces. It consists of a Node.js backend, Vue.js
frontend, and extensible node-based workflow engine.

## General Guidelines

- **Always use pnpm** (version 10.18.3 or newer)
- **Node.js requirement**: Version 22.16 or newer
- Use `corepack enable` to manage pnpm versions correctly
- We use Linear as a ticket tracking system
- We use Posthog for feature flags
- When starting to work on a new ticket – create a new branch from fresh
  master with the name specified in Linear ticket
- When creating a new branch for a ticket in Linear - use the branch name
  suggested by linear
- Use mermaid diagrams in MD files when you need to visualise something

## Essential Commands

### Building
Use `pnpm build` to build all packages. ALWAYS redirect the output of the
build command to a file:

```bash
pnpm build > build.log 2>&1
```

You can inspect the last few lines of the build log file to check for errors:
```bash
tail -n 20 build.log
```

### Development
- `pnpm dev` - Start all packages in development mode with hot reload
- `pnpm dev:be` - Backend-only development (excludes frontend packages)
- `pnpm dev:fe` - Frontend-only development (runs backend server + editor-ui dev server)
- `pnpm dev:ai` - AI/LangChain nodes development
- `pnpm start` - Start n8n in production mode
- `N8N_DEV_RELOAD=true pnpm dev` - Enable hot reload for custom nodes/credentials

**Performance tip:** Running all packages in dev mode is resource-intensive. Use
filtered commands (`dev:be`, `dev:fe`, `dev:ai`) to run only relevant packages
for better performance.

### Testing
- `pnpm test` - Run all tests
- `pnpm test:affected` - Runs tests based on what has changed since the last
  commit
- `pnpm dev:e2e` - E2E tests in development mode

Running a particular test file requires going to the directory of that test
and running: `pnpm test <test-file>`.

When changing directories, use `pushd` to navigate into the directory and
`popd` to return to the previous directory. When in doubt, use `pwd` to check
your current directory.

### Code Quality
- `pnpm lint` - Lint code
- `pnpm typecheck` - Run type checks

Always run lint and typecheck before committing code to ensure quality.
Execute these commands from within the specific package directory you're
working on (e.g., `cd packages/cli && pnpm lint`). Run the full repository
check only when preparing the final PR. When your changes affect type
definitions, interfaces in `@n8n/api-types`, or cross-package dependencies,
build the system before running lint and typecheck.

## Architecture Overview

**Monorepo Structure:** pnpm workspaces with Turbo build orchestration

### Package Structure

The monorepo is organized into these key packages:

Backend packages:
- **`packages/@n8n/api-types`**: Shared TypeScript interfaces between frontend and backend
- **`packages/workflow`**: Core workflow interfaces and types
- **`packages/core`**: Workflow execution engine
- **`packages/cli`**: Express server, REST API, and CLI commands
- **`packages/nodes-base`**: Built-in nodes for integrations
- **`packages/@n8n/nodes-langchain`**: AI/LangChain nodes
- **`packages/@n8n/config`**: Centralized configuration management
- **`packages/@n8n/di`**: Dependency injection container
- **`packages/@n8n/errors`**: Error classes and handling

Frontend packages:
- **`packages/frontend/editor-ui`**: Vue 3 frontend application (main editor)
- **`packages/frontend/@n8n/design-system`**: Vue component library for UI consistency
- **`packages/frontend/@n8n/i18n`**: Internationalization for UI text
- **`packages/frontend/@n8n/stores`**: Pinia stores for state management
- **`packages/frontend/@n8n/rest-api-client`**: API client for backend communication
- **`packages/frontend/@n8n/composables`**: Reusable Vue composition functions

## Technology Stack

- **Frontend:** Vue 3 + TypeScript + Vite + Pinia + Storybook UI Library
- **Backend:** Node.js + TypeScript + Express + TypeORM
- **Testing:** Jest (unit) + Playwright (E2E)
- **Database:** TypeORM with SQLite/PostgreSQL/MySQL support
- **Code Quality:** Biome (for formatting) + ESLint + lefthook git hooks

### Key Architectural Patterns

1. **Dependency Injection**: Uses `@n8n/di` for IoC container
2. **Controller-Service-Repository**: Backend follows MVC-like pattern
3. **Event-Driven**: Internal event bus for decoupled communication
4. **Context-Based Execution**: Different contexts for different node types
5. **State Management**: Frontend uses Pinia stores
6. **Design System**: Reusable components and design tokens are centralized in
   `@n8n/design-system`, where all pure Vue components should be placed to
   ensure consistency and reusability

## Key Development Patterns

- **Turbo Build Orchestration**: Uses Turbo to manage build dependencies and caching
  - Build tasks automatically handle dependencies (e.g., `^build` means "build dependencies first")
  - Turbo caches build outputs for faster rebuilds
  - Most commands support `--filter` to target specific packages
- **Package Isolation**: Each package has isolated build configuration and can be developed independently
- **Hot Reload**: Works across the full stack during development
  - Use `N8N_DEV_RELOAD=true` for automatic node/credential reloading
  - File watching can be resource-intensive on slower machines
- **Node Development**: Uses dedicated `node-dev` CLI tool for creating custom nodes
- **JSON-Based Workflow Tests**: Integration tests for nodes use JSON workflow definitions
- **Selective Development**: Use filtered dev commands for better performance on resource-constrained machines

### TypeScript Best Practices
- **NEVER use `any` type** - use proper types or `unknown`
- **Avoid type casting with `as`** - use type guards or type predicates instead
- **Define shared interfaces in `@n8n/api-types`** package for FE/BE communication

### Error Handling
- Don't use `ApplicationError` class in CLI and nodes for throwing errors,
  because it's deprecated. Use `UnexpectedError`, `OperationalError` or
  `UserError` instead.
- Import from appropriate error classes in each package

### Frontend Development
- **All UI text must use i18n** - add translations to `@n8n/i18n` package
- **Use CSS variables directly** - never hardcode spacing as px values
- **data-test-id must be a single value** (no spaces or multiple values)

When implementing CSS, refer to @packages/frontend/CLAUDE.md for guidelines on
CSS variables and styling conventions.

### Testing Guidelines
- **Always work from within the package directory** when running tests
- **Mock all external dependencies** in unit tests
- **Confirm test cases with user** before writing unit tests
- **Typecheck is critical before committing** - always run `pnpm typecheck`
- **When modifying pinia stores**, check for unused computed properties

What we use for testing and writing tests:
- For testing nodes and other backend components, we use Jest for unit tests. Examples can be found in `packages/nodes-base/nodes/**/*test*`.
- We use `nock` for server mocking
- For frontend we use `vitest`
- For e2e tests we use `Playwright` and `pnpm dev:e2e`. The old Cypress tests
  are being migrated to Playwright, so please use Playwright for new tests.

### Common Development Tasks

When implementing full-stack features:
1. Define API types in `packages/@n8n/api-types`
2. Implement backend logic in `packages/cli`:
   - Use dependency injection (`@n8n/di`) for services
   - Follow Controller-Service-Repository pattern
   - Use appropriate error classes from `@n8n/errors`
3. Add API endpoints via controllers (REST API follows Express conventions)
4. Update frontend in `packages/frontend/editor-ui`:
   - Add UI text to `@n8n/i18n` package (never hardcode text)
   - Use components from `@n8n/design-system` when possible
   - Update Pinia stores in `@n8n/stores` for state management
   - Use CSS variables from design system (see `packages/frontend/CLAUDE.md`)
5. Write tests with proper mocks:
   - Backend: Jest with `nock` for HTTP mocking
   - Frontend: Vitest
   - E2E: Playwright (Cypress is being phased out)
6. Build and run typechecks:
   - `pnpm build` (if you changed types/interfaces)
   - `pnpm typecheck` to verify types

When implementing new nodes:
1. Use `pnpm --filter @n8n/node-cli create` to scaffold
2. Implement node logic in `packages/nodes-base/nodes/`
3. Add workflow tests as JSON files
4. Enable hot reload with `N8N_DEV_RELOAD=true pnpm dev`

## Github Guidelines
- When creating a PR, use the conventions in
  `.github/pull_request_template.md` and
  `.github/pull_request_title_conventions.md`.
- Use `gh pr create --draft` to create draft PRs.
- Always reference the Linear ticket in the PR description,
  use `https://linear.app/n8n/issue/[TICKET-ID]`
- always link to the github issue if mentioned in the linear ticket.
