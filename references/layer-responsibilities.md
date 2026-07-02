# Layer Responsibilities

What belongs in each layer, from `app/routes` and components down through
`domain` and server infrastructure. Read this alongside
[`layout-and-module-placement.md`](layout-and-module-placement.md) for where
files live and
[`boundary-and-contract-rules.md`](boundary-and-contract-rules.md) for the
dependency direction these responsibilities must respect.

## Client Layers

### `app/routes/`

- Define route modules.
- Read loader and action inputs.
- Map route data into page composition.
- Delegate non-trivial logic to `client/usecase` or `server/usecase`.

Do not call ORM clients, SDK clients, or other server-infrastructure modules
directly; hide business rules in route files; or let route modules become the
main state container for reusable interactions.

### `app/components/<feature>/`

- Render UI from props.
- Hold tiny UI-local state only when it is truly view-local, such as popover
  open state, active tab index, focused element, or an uncontrolled input
  bridge.

Do not fetch data, own mutation orchestration, build DTOs for the server,
contain business rules or persistence rules, or introduce component-local
`state/` or `reducers/` directories.

### `app/components/shared/`

- Hold reusable presentational primitives.
- Keep these components styleable and composable.
- Treat `shared` as an extraction target, not as the default starting location
  for new components.

Promote a component to `shared` only when:

1. it is used or clearly about to be used across multiple features
2. the API can be expressed in generic UI terms rather than product vocabulary
3. it does not need feature-specific state or handlers
4. extraction reduces duplication without turning it into a configurable
   monster

Do not import feature use cases, own feature vocabulary, or hide data fetching
or mutation logic in `shared`.

### `app/lib/client/usecase/`

- Own screen-level state and event handlers.
- Assemble view models for components.
- Coordinate API clients, router calls, optimistic UI, reducers, and derived
  state.
- Hold use-case-local DTO mapping when that mapping is part of the interaction
  flow.
- Expose a stable interface for the view layer.

Prefer a feature directory when more than one file is needed for the same
flow.

Reserve `store.ts` for cases where shared identity and lifecycle actually
matter across multiple sibling views. Do not default to a store when a Hook
plus reducer is enough.

### `app/lib/client/infrastructure/`

- Implement browser-facing and network-facing adapters.
- Wrap `fetch`, `localStorage`, clipboard, browser events, and router
  integration.
- Translate transport details into domain-friendly or usecase-friendly
  interfaces.

Keep framework adapters here, not in a generic common layer.

### `app/lib/client/infrastructure/api/`

- Hold feature or resource API clients.
- Keep transport and serialization details here.
- Return shapes that the owning use case can consume directly.
- Default to JSON DTOs, not domain entity instances.
- Keep client-side mapping shallow here; put use-case-specific reconstruction
  in the owning use case.

Client flow should normally be:

```text
client/usecase
  -> client/infrastructure/api
  -> HTTP JSON DTO
  -> server/usecase
```

### `app/lib/client/infrastructure/browser/`

- Hold browser-only integrations such as `localStorage`, `sessionStorage`,
  clipboard, media query, `BroadcastChannel`, `IntersectionObserver`, or
  document event adapters.
- Keep direct DOM and browser API calls here unless the logic is tiny and
  strictly component-local.

### Optional `client/infrastructure/repositories/`

Do not make this a baseline directory.

Add a client-side repository layer only when the client must hide multiple
data sources behind one abstraction, for example:

- IndexedDB plus remote API
- memory cache plus remote fetch
- offline-first synchronization
- local-first conflict handling

If the client is only calling HTTP endpoints, prefer `api/` instead of a
repository abstraction.

## Server And Domain Layers

The `server/*` layers below execute only when the app runs with the React
Router server runtime (`ssr: true`). In SPA mode (`ssr: false`) they are not
part of the runtime deployment; the client consumes an external backend
through `client/infrastructure/api` instead.

### `app/lib/domain/entities/`

- Hold business concepts that enforce invariants or domain behavior.
- Use `class` when identity or invariants matter.
- Prefer named methods that express business operations over generic setters.
- Keep entities free from React, ORM clients, browser APIs, and HTTP concerns.

Do not place `CreateXRequest`, `UpdateXPayload`, or `ListXResponse` types here
unless they are genuinely domain concepts, which is rare.

### `app/lib/domain/value-objects/`

- Hold small immutable domain concepts with validation and equality semantics.
- Prefer validated factory functions or constructors over raw object literals.
- Keep them free from transport, framework, and persistence details.

Do not use value objects as a disguised home for endpoint DTOs.

### `app/lib/domain/policies/`

- Hold business rules that span multiple entities or require explicit decision
  logic.
- Prefer this directory when the code reads like a rule or policy statement.

### `app/lib/domain/services/`

- Hold domain-level orchestration that is still infrastructure-free and not
  naturally owned by a single entity or value object.
- Use this directory sparingly.
- Prefer `policies/` when the code is fundamentally a rule, and prefer
  `services/` only when it is true domain orchestration.

### `app/lib/domain/repositories/`

- Define repository ports as interfaces or types.
- Describe what the domain or use case needs from persistence.
- Keep these contracts independent from any specific ORM, query builder, or
  storage schema.
- Treat these as domain-facing persistence ports first.

Do not turn repository ports into generic request or response contract
storage.

### `app/lib/server/usecase/`

- Implement server-side application services.
- Orchestrate repositories and domain rules.
- Map between route inputs and domain operations.
- Accept repositories and gateways through constructors or explicit function
  parameters.
- Keep HTTP response formatting and status selection out of this layer.
- Keep authorization checks and side-effect decisions visible in this layer.

Do not leak ORM types into domain or client layers, or instantiate concrete
infrastructure-backed repositories inline with `new`.

### `app/lib/server/infrastructure/`

- Hold ORM clients, query builders, SDK clients, repository implementations,
  external API gateways, filesystem access, and environment-aware wiring.
- Map storage or service details into repository ports.
- Hold explicit job or side-effect adapters when a use case must trigger
  background work.

This is the only layer that should know about the project's chosen ORM, SQL
client, or backend SDKs.

### `app/lib/server/infrastructure/repositories/`

- Hold repository implementations backed by the project's chosen data stack
  (ORM, query builder, raw SQL client, filesystem persistence, or remote
  backend).
- Accept long-lived clients such as a connection pool, ORM client, or SDK
  client through constructors so lifetime stays explicit.
- Keep repositories stateless with respect to request identity, auth state,
  and current transaction.

### `app/lib/server/infrastructure/gateways/`

- Hold integrations with external SDKs and remote services.
- Accept SDK clients or configuration through constructors or explicit factory
  helpers.
- Keep outbound adapter logic here rather than inside use cases.
