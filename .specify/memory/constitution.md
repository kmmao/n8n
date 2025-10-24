<!--
Sync Impact Report - Constitution Update
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Version Change: [Initial Template] → 1.0.0
Change Type: MINOR - Initial ratification from project development guidelines
Modified Principles: N/A (initial version)
Added Sections:
  - Core Principles (7 principles)
  - Technical Standards
  - Development Workflow
  - Governance
Templates Impact:
  ✅ spec-template.md - Aligned with TypeScript standards and testing requirements
  ✅ plan-template.md - Constitution Check section references this document
  ✅ tasks-template.md - Test-first and parallel execution patterns aligned
Follow-up TODOs: None
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-->

# n8n Development Constitution

## Core Principles

### I. TypeScript Type Safety (NON-NEGOTIABLE)

**MUST** maintain strict type safety across all TypeScript code:
- **NEVER use `any` type** - use proper types, `unknown`, or type guards instead
- **NEVER use type casting with `as`** - use type guards or type predicates
- Define shared interfaces in `@n8n/api-types` package for frontend/backend communication
- All cross-package type changes MUST trigger full repository typecheck before commit

**Rationale**: Type safety prevents runtime errors, enables confident refactoring, and serves as
living documentation. The `any` type defeats these benefits and creates maintenance debt.

### II. Error Handling Standards (NON-NEGOTIABLE)

**MUST** use appropriate error classes from `@n8n/errors` package:
- Use `UserError` for user-facing validation and input errors
- Use `OperationalError` for expected runtime conditions (network failures, resource limits)
- Use `UnexpectedError` for programming errors and assertions
- **NEVER use deprecated `ApplicationError`** in CLI and nodes packages
- All service layer methods MUST declare error types they throw

**Rationale**: Consistent error taxonomy enables proper error handling at boundaries
(API responses, UI, logs) and distinguishes recoverable from non-recoverable failures.

### III. Code Quality Gates (NON-NEGOTIABLE)

**MUST** pass quality checks before committing:
- Run `pnpm lint` from within the specific package directory being modified
- Run `pnpm typecheck` from within the specific package directory
- Build system (`pnpm build`) MUST succeed if type definitions or cross-package dependencies changed
- Run full repository checks only when preparing final PR
- All build output MUST be redirected to file: `pnpm build > build.log 2>&1`

**Rationale**: Package-local checks provide fast feedback. Full repository validation occurs
at PR time via CI. Build output redirection prevents console spam during development.

### IV. Frontend I18N and Design System (NON-NEGOTIABLE)

**MUST** follow frontend consistency standards:
- **All UI text MUST use i18n** - add translations to `@n8n/i18n` package
- **NEVER hardcode spacing as px values** - use CSS variables from design system
- **NEVER use multiple values in `data-test-id`** - single kebab-case identifier only
- Reusable pure Vue components MUST be placed in `@n8n/design-system` package
- CSS variables from design system (spacing, colors, typography) MUST be used directly

**Rationale**: Consistent design tokens and internationalization enable global deployments,
accessibility, and visual consistency. Centralized component library prevents duplication.

### V. Testing Discipline

**MUST** follow testing requirements:
- **Always work from within the package directory** when running tests
- **Mock all external dependencies** in unit tests (use `nock` for HTTP mocking)
- **Confirm test cases with user** before writing unit tests
- Backend: Use Jest for unit tests (examples in `packages/nodes-base/nodes/**/*test*`)
- Frontend: Use Vitest for component/unit tests
- E2E: Use Playwright only (Cypress tests being migrated, do not add new Cypress tests)
- Node workflow tests: Use JSON-based workflow definitions for integration testing

**Rationale**: Test confirmation prevents wasted effort. Isolated tests enable confident
refactoring. Playwright standardization reduces maintenance burden of multiple frameworks.

### VI. Build Performance and Selective Development

**MUST** optimize build and development performance:
- Use filtered development commands: `pnpm dev:be` (backend only), `pnpm dev:fe` (frontend only),
  `pnpm dev:ai` (AI/LangChain nodes only)
- Running full `pnpm dev` for all packages is resource-intensive - use only when necessary
- Use `N8N_DEV_RELOAD=true pnpm dev` to enable hot reload for custom nodes/credentials
- Leverage Turbo build caching - avoid unnecessary rebuilds
- Use `--filter` flag to target specific packages during build operations

**Rationale**: Monorepo development can consume significant resources. Selective mode
enables faster iteration on slower machines and reduces cognitive load.

### VII. Package Management Consistency (NON-NEGOTIABLE)

**MUST** use pnpm for all package operations:
- Use `pnpm` (version 10.18.3 or newer) for all install/add/remove operations
- **NEVER use npm or yarn** - these will corrupt the pnpm workspace
- Use `corepack enable` to manage pnpm versions correctly
- Node.js version 22.16 or newer required
- Respect pnpm workspace structure defined in `pnpm-workspace.yaml`

**Rationale**: Mixed package managers cause lockfile conflicts, phantom dependencies,
and disk space waste. pnpm workspaces enable efficient monorepo dependency management.

## Technical Standards

### Dependency Injection

Use `@n8n/di` container for all service dependencies:
- Controllers and services MUST use constructor injection
- Follow Controller-Service-Repository pattern in backend
- Avoid service locator anti-pattern (directly importing container in business logic)

### Full-Stack Feature Implementation Order

When implementing features requiring frontend and backend changes:

1. Define API types in `packages/@n8n/api-types`
2. Implement backend logic in `packages/cli` (dependency injection, appropriate error classes)
3. Add REST API endpoints via Express controllers
4. Update frontend in `packages/frontend/editor-ui`:
   - Add UI text to `@n8n/i18n` package
   - Use components from `@n8n/design-system`
   - Update Pinia stores in `@n8n/stores` for state management
   - Use CSS variables from design system
5. Write tests with proper mocks (Jest backend, Vitest frontend, Playwright E2E)
6. Build and typecheck if types/interfaces changed

### Node Development

When creating custom nodes:
- Use `pnpm --filter @n8n/node-cli create` to scaffold new nodes
- Implement node logic in `packages/nodes-base/nodes/`
- Add JSON-based workflow tests
- Enable hot reload during development: `N8N_DEV_RELOAD=true pnpm dev`

## Development Workflow

### Branching and Commit Strategy

- Use Linear for ticket tracking
- Use Posthog for feature flags
- Create new branch from fresh master with Linear ticket name suggestion
- Follow conventions in `.github/pull_request_template.md` and
  `.github/pull_request_title_conventions.md`
- Always reference Linear ticket: `https://linear.app/n8n/issue/[TICKET-ID]`
- Link to GitHub issue if mentioned in Linear ticket
- Use `gh pr create --draft` for draft PRs
- Run `pnpm lint` and `pnpm typecheck` from package directory before committing
- Commit after each logical task completion

### Working with Pinia Stores

When modifying frontend state in `@n8n/stores`:
- Check for unused computed properties after modifications
- Ensure store actions properly handle error states
- Follow reactive patterns - avoid direct state mutation outside actions

### Git Hooks

- Lefthook git hooks are configured for pre-commit quality checks
- Do not bypass hooks without justification
- If hooks fail, fix issues rather than using `--no-verify`

## Governance

This constitution supersedes all other development practices. Amendments require:
1. Documentation of the change rationale
2. Approval from project maintainers
3. Migration plan for existing code if breaking change
4. Update of this constitution file with version bump

All PRs and code reviews MUST verify compliance with these principles.

Complexity and deviations MUST be explicitly justified in implementation plans
(see `Complexity Tracking` section in plan-template.md).

For runtime development guidance, refer to:
- `/Users/sangreal/Documents/GitHub/n8n/CLAUDE.md` (primary development guidelines)
- `/Users/sangreal/Documents/GitHub/n8n/packages/frontend/CLAUDE.md` (frontend-specific guidance)

**Version**: 1.0.0 | **Ratified**: 2025-10-25 | **Last Amended**: 2025-10-25
