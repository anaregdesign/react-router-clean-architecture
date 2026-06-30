---
name: react-router-clean-architecture
description: "Own React Router app-code architecture, route boundaries, UI structure, and verification for Vite-based web apps. Use when the request mentions React Router routes, loaders or actions, component boundaries, use cases, domain models, server-side data access through repository ports, component-library UI, responsive UI, charts, Playwright UI verification, or app-structure refactoring."
---

# React Router App Architecture

## Top Priority Rules (Non-Negotiable)

**These five rules are the most important in this skill. They are absolute,
take precedence over convenience, speed, or any other guidance here, and must
never be violated.**

1. **Route with FlatRoute.** Use FlatRoute
   (`@react-router/fs-routes` `flatRoutes()`) as the routing convention: one
   file per route module under `app/routes/`, with dots in the file name
   mapping to URL slashes and `$segment` marking dynamic parameters.
2. **One React component per file.** Each `.tsx` file exports exactly one
   React component, and the file name (`PascalCase.tsx`) matches the exported
   component name exactly.
3. **One CSS Module per component.** Every component that needs custom CSS owns
   exactly one sibling `<ComponentName>.module.css`, imported as
   `import styles from "./<ComponentName>.module.css";` and applied through the
   `styles` object. Never import non-module CSS inside a component.
4. **Obey the canonical layout and dependency direction.** Place every file in
   its canonical owner, and keep dependencies pointing strictly inward:
   `routes`/`components` → client orchestration → `domain`; `server/usecase` →
   `domain` + repository ports; `server/infrastructure` implements those ports.
   Dependencies must never point outward.
5. **Never exceed a layer's responsibility.** Keep every layer strictly within
   its role — components render, use cases orchestrate, domain holds business
   rules, infrastructure talks to the outside world. Never implement anything
   that belongs to another layer.

The detailed rules in the rest of this document expand on these five. Whenever
any guidance appears to conflict, these five always win.

## Overview

Use this skill as the default architecture workflow for a React Router SPA that
uses Vite. Use it from initial bootstrap through ongoing implementation. Keep
FlatRoute modules declarative, view files thin, data access server-only, and
dependency direction explicit before writing code.

This skill owns code structure, dependency direction, UI guardrails, and
verification for the app codebase. Do not use it to define cloud provider
choice, identity provisioning, secret-store topology, IaC layout, or deployment
infrastructure; let a companion hosting skill add those concerns while
preserving these rules.

This skill is also persistence-agnostic. It defines where the data access layer
lives, how it is shaped (repository ports in `domain`, repository
implementations and ORM/SDK code in `server/infrastructure`), and how it must
not leak across boundaries — but it does not pick an ORM, query builder, or
database engine. Plug in the data stack that fits the project (Drizzle, Kysely,
Prisma, TypeORM, raw `pg`, an HTTP backend, etc.) inside
`app/lib/server/infrastructure/` and keep the rest of the rules unchanged.

For new or unstandardized UI work, prefer a single, consistent component
library and a quiet, simple visual presentation. Keep primary
labels and layouts concise, and move only supplemental, non-essential detail
into tooltip or secondary-detail patterns.

When data visualization is required, prefer the simplest accessible chart that
matches the analytical task and keep chart interaction lightweight.

For responsive behavior, prefer guidance that keeps the same feature usable on
both desktop and mobile rather than treating `mobile-first` as a universal
process requirement.

If a companion hosting skill explicitly overrides runtime mode or config
bootstrap, keep these architecture and boundary rules and let the companion
override only the hosting-specific pieces.

## Canonical Layout

This canonical directory layout is the most important structural rule in this
skill. Place every file in its canonical owner and organize directories by
responsibility, not by team preference or historical drift.

```text
app/
  routes/
  components/
    <feature>/
    shared/
  lib/
    client/
      usecase/
        <feature>/
          use-<feature>.ts
          state.ts
          reducer.ts
          selectors.ts
          handlers.ts
      infrastructure/
        api/
        browser/
    server/
      usecase/
      infrastructure/
        config/
        repositories/
        gateways/
    domain/
      entities/
      value-objects/
      policies/
      services/
      repositories/
```

The Placement Guide below maps each need to its location, and
[`references/layout-and-module-placement.md`](references/layout-and-module-placement.md)
covers the placement preflight, naming, and feature-boundary rules.

## Quick Start

1. Always read
   [`references/layout-and-module-placement.md`](references/layout-and-module-placement.md)
   before deciding where code should live.
2. Run a placement preflight before implementing:
   - list every new file, moved file, and extracted file
   - assign the exact canonical target path for each file before writing code
   - if a file does not fit the canonical layout, stop and revise the placement
     plan before creating a new directory
3. Classify the requested change:
   - route composition
   - presentational UI
   - client interaction flow
   - server use case
   - persistence or external integration
   - domain rule
   - cross-boundary contract or utility
4. Place code in its canonical owner from the Canonical Layout above. If a
   file does not fit, revise the placement plan before creating a new
   directory; the Placement Guide below maps each need to its location.
5. Read the narrowest matching reference for the task. The References section
   at the end of this document lists every reference grouped by topic; load
   only the one or two that match the change instead of the whole catalog.

## Core Rules

These rules expand the five Top Priority Rules above. Each group links to the
reference that owns the full detail; load that reference for a matching change.

### Layout and dependencies

- Lock file placement before coding: name exact target paths first, then
  implement. If a file does not fit the Canonical Layout, revise the plan
  instead of inventing a convenience directory.
- Keep dependency direction inward: `routes`/`components` → `client/usecase` →
  `client/infrastructure` → `domain`; `server/usecase` → `domain` + repository
  ports; `server/infrastructure` implements those ports; `domain` depends only
  on `domain`.
- Keep feature-local components in `app/components/<feature>/` and promote to
  `app/components/shared/` only when feature-agnostic. Do not add generic
  buckets (`app/features/`, `app/hooks/`, `app/utils/`, `app/types/`,
  `app/store/`) or horizontal `state`/`reducers`/`handlers` directories.
- See
  [`references/layout-and-module-placement.md`](references/layout-and-module-placement.md).

### Boundaries, contracts, and dependency injection

- Keep ORM/SDK/query-builder and direct database or network access inside
  `app/lib/server/infrastructure/`; use cases and `domain` talk to repository
  ports, not concrete clients.
- Validate at the owning layer: transport shape in routes or API adapters,
  application rules in use cases, business invariants in `domain`. Keep
  transport `Request`/`Response` DTOs near their boundary and promote into
  `domain` only when they are real business concepts.
- Instantiate repositories and gateways in a composition root or factory, not
  inside `domain` or use cases. Keep authorization, serialization, and side
  effects explicit, and pass request context explicitly instead of through
  module globals or singletons.
- See
  [`references/boundary-and-contract-rules.md`](references/boundary-and-contract-rules.md)
  and
  [`references/dependency-injection-lifetime-and-side-effects.md`](references/dependency-injection-lifetime-and-side-effects.md).

### Routing and APIs

- Use FlatRoute (`@react-router/fs-routes` `flatRoutes()`): one file per route
  module under `app/routes/`, dots mapping to slashes, `$segment` for dynamic
  params. Deviate only when the project, a generator, or a framework
  integration requires it.
- Shape URLs as resources (`/items`, `/items/$itemId`,
  `/items/$itemId/comments`) and reserve rare verb paths for genuine non-CRUD
  operations. Keep route modules limited to HTTP, loader/action wiring, and
  top-level composition.
- See
  [`references/flat-route-rest-api-guidelines.md`](references/flat-route-rest-api-guidelines.md).

### Client state and components

- Keep async state, mutation handlers, and derived view models in
  `app/lib/client/usecase/`, with `state`/`reducer`/`selectors`/`handlers`
  colocated in the owning feature directory. Keep components presentational
  with only ephemeral UI state.
- Default client data access to `api` adapters plus DTO mapping in the owning
  use case; add a client-side repository only for multi-source or local-first
  needs.
- See
  [`references/view-state-and-handler-patterns.md`](references/view-state-and-handler-patterns.md).

### Components, styling, and UI presentation

- One React component per `.tsx` file with the file name matching the exported
  component; default component-owned styling to a sibling
  `<ComponentName>.module.css` and never import non-module CSS inside a
  component. Avoid inline `style` for static styling, and reserve global CSS
  under `app/styles/` for resets, fonts, baseline, and theme-host wiring.
- Prefer one component library for new UI: compose its documented primitives
  before custom controls, use its design tokens and styling solution for
  theme-aware visuals, and do not mix two general-purpose libraries. Keep UI
  visually simple and keep required labels, validation, and critical status
  visible without hover.
- Choose the simplest accessible chart for the task, and build responsive UI
  that stays capable on desktop and mobile with content-driven breakpoints,
  narrow-screen reflow, and WCAG 2.2 touch targets.
- See
  [`references/component-file-and-css-module-rules.md`](references/component-file-and-css-module-rules.md)
  and
  [`references/ui-presentation-guidance.md`](references/ui-presentation-guidance.md).

### Domain modeling and types

- Use `class` only when identity, invariants, or lifecycle matter; prefer
  composition over inheritance and keep DTO or transport shapes as `type` plus
  functions. Use `interface` for stable object ports, `type` for DTOs, unions,
  and view models.
- Treat untrusted data as `unknown` at the boundary and narrow it immediately.
  Keep one concept under one owner, build behavior around `domain` without
  forcing UI, DTO, or infrastructure concerns into it, and keep constants in
  the narrowest owner.
- See
  [`references/domain-modeling-and-type-rules.md`](references/domain-modeling-and-type-rules.md).

### Bootstrap and verification

- Prefer React Router's official Vite-powered bootstrap; pick the data stack
  once and confine its imports to `app/lib/server/infrastructure/`. This skill
  does not choose the ORM, query builder, or database engine, and does not own
  cloud provisioning, identity, secrets, IaC, or release infrastructure — leave
  those to a companion hosting skill.
- For UI-affecting changes, verify the rendered result with Playwright
  (accessible locators, web-first assertions, relevant routes and viewports)
  rather than code inspection alone.
- See
  [`references/project-bootstrap.md`](references/project-bootstrap.md),
  [`references/playwright-ui-verification.md`](references/playwright-ui-verification.md),
  and
  [`references/verification-gates.md`](references/verification-gates.md).

## Implementation Workflow

### Placement Preflight For Every Change

- Read
  [`references/layout-and-module-placement.md`](references/layout-and-module-placement.md)
  first.
- Write the exact target paths for every created or moved file before touching
  code.
- Confirm that each target path matches one canonical owner:
  - route HTTP composition: `app/routes/`
  - feature-local presentational UI: `app/components/<feature>/`
  - shared presentational UI: `app/components/shared/`
  - client orchestration and state: `app/lib/client/usecase/<feature>/`
  - client adapters: `app/lib/client/infrastructure/`
  - server orchestration: `app/lib/server/usecase/`
  - server integrations, ORM/SDK clients, and persistence:
    `app/lib/server/infrastructure/`
  - domain concepts and ports: `app/lib/domain/`
- If any planned file lacks a clear owner, fix the placement plan before
  implementing.

### 0. Bootstrap new projects correctly

- When starting from scratch, follow
  [`references/project-bootstrap.md`](references/project-bootstrap.md) before
  writing features.
- Prefer `create-react-router` for this skill's architecture because it
  already aligns with Vite, route modules, and SPA mode.
- Add `@react-router/fs-routes` and SPA configuration before layering domain or
  use-case code. Use FlatRoute (`flatRoutes()`) as the routing convention from
  the first route file so URL shape, dynamic segments, and index modules stay
  consistent across the app.
- Add the chosen component library early for new UI work so components,
  theming, and accessibility patterns stay consistent from the first screen.
- Pick the data stack (ORM, query builder, or remote backend client) once,
  document it in the project, and keep its imports confined to
  `app/lib/server/infrastructure/`. This skill is intentionally silent on
  which one.
- Before substantial feature work begins, prefer installing the baseline
  dependencies the project already knows it will need so architecture work does
  not drift around missing packages mid-implementation.
- When hosting, identity, or deployment requirements appear, keep this skill on
  code-level architecture and hand the platform-specific decisions to the
  companion hosting skill.

### 1. Model the change around a use case

- Name the user intent first.
- Put invariants in `domain/entities` or `domain/value-objects`, and put
  cross-entity rules in `domain/policies` or `domain/services` when they do not
  naturally belong to one model.
- Put behavior where the state lives: entity behavior on entities, value
  normalization on value objects, cross-entity rules on policies, and
  orchestration on services or use cases.
- Keep literals local until they become stable, named concepts. Extract only
  when the name improves readability or the value is reused inside one clear
  boundary.
- Treat untrusted data as `unknown` first, then narrow it at the boundary with
  a parser, schema, or type guard before promoting it to a DTO or domain shape.
- When two modules appear to serve the same role, first ask whether they are
  actually the same concept in the same layer. Merge duplicates inside one
  boundary, but keep separate shapes when one is a DTO, one is a view model, or
  one is a domain model.
- Put repository interfaces in `domain/repositories`.
- Keep request or response shapes next to the route, API client, or use case
  that owns them. Promote them only after the boundary is stable and truly
  reused.
- Do not move transport `Request` or `Response` types into `domain` just
  because several modules share them.

### 2. Keep view logic out of components

- Build a custom Hook, controller, or view-model module in
  `app/lib/client/usecase/` for each non-trivial screen or interaction flow.
- When the interaction has enough complexity to need submodules, create a
  feature directory and keep the use case internals there.
- Keep feature-specific presentational components in
  `app/components/<feature>/` by default. Promote a component to
  `app/components/shared/` only after it proves to be truly feature-agnostic.
- Prefer composing views from the component library's primitives before
  introducing custom low-level controls.
- Keep on-screen copy terse. Put optional elaboration behind a tooltip,
  popover, or similar secondary affordance only when the extra detail is not
  required for task completion.
- Keep purely presentational responsive adaptation in CSS, layout primitives,
  and presentational components. Move breakpoint-aware state into `usecase`
  only when it changes interaction flow or data loading behavior.
- For chart-heavy views, keep series transformation, grouping, filtering, and
  drill state in `app/lib/client/usecase/`, and keep chart components focused
  on rendering, theming, and accessibility wiring.
- Let that module own:
  - async calls
  - reducer logic
  - derived screen state
  - event handlers
  - error and pending mapping
- Pass view-ready props into `app/components/<feature>/` or
  `app/components/shared/` when the component is genuinely cross-feature.

### 3. Use React Router primitives before inventing new state containers

- Use loader data for route reads.
- Use action or fetcher state for mutations.
- Use navigation or fetcher pending state instead of duplicating network flags
  in component state.
- Add client state only for interaction state that is not already owned by the
  router.

### 4. Move DTOs through the client boundary explicitly

- Return JSON DTOs from route loaders, actions, or API endpoints.
- Let `client/infrastructure/api` fetch those DTOs.
- Rebuild client-facing objects or value objects inside the owning use case
  only when that adds real value.
- Do not expect server-side `Entity` instance identity to cross the network
  boundary.

### 5. Assemble dependencies at the edge

- Keep `usecase` and `domain` code constructor-injected.
- Build repository, gateway, and service instances in route loaders, actions,
  server entry points, or dedicated dependency factories.
- Prefer manual DI with explicit factory functions before introducing a DI
  container.
- Recreate request-scoped dependencies such as transaction-bound repositories
  from the current request or transaction context.

### 6. Validate and map errors by layer

- Keep request parsing and input shape validation at the HTTP or client
  transport boundary.
- Keep use-case rule validation in `server/usecase` or `client/usecase`.
- Keep invariant enforcement in `domain`.
- Map domain, application, and infrastructure errors to transport responses at
  the edge.

### 7. Keep authorization and serialization explicit

- Resolve authentication at the edge, but keep authorization decisions in use
  cases or domain policies.
- Convert `Date`, ids, money-like values, and value objects at boundaries
  rather than leaking transport or ORM shapes inward.

### 8. Keep mutable state scoped

- Do not store request-specific state in module globals or singleton services.
- Pass auth context, user context, tenant context, and correlation metadata
  explicitly.
- Keep transaction handles scoped to the current transaction only.
- Treat client-side async race conditions as correctness issues and guard
  against stale responses.

### 9. Use explicit compromises for stateful flows

- For chat, streaming, session, playback, or wizard-style flows, allow a
  feature-local controller or store when lifecycle and identity genuinely
  matter.
- Keep that compromise inside `app/lib/client/usecase/<feature>/` rather than
  spreading mutable state across routes or components.
- Split durable state, ephemeral runtime state, and infrastructure handles
  explicitly.

### 10. Push side effects and migrations into explicit workflows

- Keep schema changes, background jobs, external notifications, and indexing
  side effects visible and testable.
- Do not hide them inside random route handlers or repositories.
- Use barrel exports sparingly and only when they do not obscure ownership or
  create cycles.

### 11. Verify before completing the change

- Run targeted tests for the touched area.
- Run typecheck and lint or the project quality gate.
- Audit for boundary drift and forbidden imports.
- For UI-affecting changes, run the touched route in Playwright and confirm the
  rendered result, interaction states, and responsive layout.
- Fix architecture violations before considering the change done even if tests
  pass.

### 12. Refactor overloaded files in phases

- When a `ts` or `tsx` file has too many responsibilities, do not rewrite it in
  one jump.
- Follow the hotspot workflow in
  [`references/hotspot-refactor-workflow.md`](references/hotspot-refactor-workflow.md).
- When several hotspots exist, start with the one that combines correctness
  risk, change frequency, and boundary damage rather than the one that is
  merely largest.
- Separate analysis, planning, extraction sequencing, execution, and
  verification.
- Move one stable seam at a time and keep the file working after each batch.

## Placement Guide

- Need feature-local pure rendering and markup: `app/components/<feature>/`
- Need reusable pure UI primitives or shared component-library composition
  wrappers: `app/components/shared/`
- Need route composition or loader/action bridging: `app/routes/`
- Need client-side state, handlers, reducers, selectors, or orchestration:
  `app/lib/client/usecase/<feature>/`
- Need browser APIs, API clients, or router adapters:
  `app/lib/client/infrastructure/`
- Need endpoint-specific API clients: `app/lib/client/infrastructure/api/`
- Need browser-only adapters such as storage, clipboard, media queries, or
  channel APIs: `app/lib/client/infrastructure/browser/`
- Need a business invariant or behavior-rich model: `app/lib/domain/entities/`
- Need a small immutable business concept with validation:
  `app/lib/domain/value-objects/`
- Need cross-entity business rules: prefer `app/lib/domain/policies/`
- Need domain-level orchestration that is still infrastructure-free:
  `app/lib/domain/services/`
- Need a repository port or domain-facing persistence contract:
  `app/lib/domain/repositories/`
- Need server orchestration: `app/lib/server/usecase/`
- Need ORM clients, SDK clients, file system access, or external service code:
  `app/lib/server/infrastructure/`
- Need platform-specific server config bootstrap or config readers:
  `app/lib/server/infrastructure/config/`
- Need persistence implementations: `app/lib/server/infrastructure/repositories/`
- Need external SDK or HTTP adapters: `app/lib/server/infrastructure/gateways/`
- Need a reusable type or utility: place it with the owning route, use case, or
  domain module first; extract only after repeated reuse proves the boundary

## References

Read [`references/layout-and-module-placement.md`](references/layout-and-module-placement.md)
first, then load only the one or two references that match the change.

- project bootstrap and baseline dependency install:
  [`references/project-bootstrap.md`](references/project-bootstrap.md)
- layout and module placement, always read first:
  [`references/layout-and-module-placement.md`](references/layout-and-module-placement.md)
- client, server, and domain layer responsibilities:
  [`references/layer-responsibilities.md`](references/layer-responsibilities.md)
- boundary and contract rules:
  [`references/boundary-and-contract-rules.md`](references/boundary-and-contract-rules.md)
- domain modeling and type rules:
  [`references/domain-modeling-and-type-rules.md`](references/domain-modeling-and-type-rules.md)
- dependency injection, lifetime, and side-effect rules:
  [`references/dependency-injection-lifetime-and-side-effects.md`](references/dependency-injection-lifetime-and-side-effects.md)
- FlatRoute REST API guidance:
  [`references/flat-route-rest-api-guidelines.md`](references/flat-route-rest-api-guidelines.md)
- view-state and handler composition:
  [`references/view-state-and-handler-patterns.md`](references/view-state-and-handler-patterns.md)
- component file, component-library, and CSS Module rules:
  [`references/component-file-and-css-module-rules.md`](references/component-file-and-css-module-rules.md)
- chart and responsive/mobile UI guidance:
  [`references/ui-presentation-guidance.md`](references/ui-presentation-guidance.md)
- Playwright UI verification workflow:
  [`references/playwright-ui-verification.md`](references/playwright-ui-verification.md)
- stateful flow compromise rules:
  [`references/stateful-flow-compromises.md`](references/stateful-flow-compromises.md)
- hotspot refactor workflow:
  [`references/hotspot-refactor-workflow.md`](references/hotspot-refactor-workflow.md)
- verification gates:
  [`references/verification-gates.md`](references/verification-gates.md)
