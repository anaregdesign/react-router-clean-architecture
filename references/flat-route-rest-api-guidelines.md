# FlatRoute REST API Guidelines

## What is FlatRoute

FlatRoute is the file routing convention enabled by
`@react-router/fs-routes`'s `flatRoutes()` helper.

Use this convention:

- one file per route module under `app/routes/`
- dots in the file name map to URL slashes
- `$segment` marks a dynamic URL parameter
- the file `_index.tsx` represents the index route for its parent

Examples:

- `app/routes/_index.tsx` -> `GET /`
- `app/routes/items.tsx` -> any HTTP method to `/items`
- `app/routes/items.$itemId.tsx` -> any HTTP method to `/items/:itemId`

## Default Routing Choice

Use FlatRoute by default for new SPA projects.

Prefer the alternative folder-based or convention-based routing only when:

- the project already uses it
- a third-party generator imposes it
- a specific framework integration requires it

## Runtime Requirement

`loader` and `action` handlers — including every `/api/*` route in this guide
— run only when the app is deployed with the React Router server runtime
(`ssr: true`). In SPA mode (`ssr: false`) they are unavailable at runtime:
use `clientLoader`/`clientAction` inside route modules and reach the external
backend through `app/lib/client/infrastructure/api/` instead. See
[`project-bootstrap.md`](project-bootstrap.md) for choosing the runtime mode.

## REST URL Mapping

Prefer URLs that read like resources, not actions.

Use this URL pattern by default:

- collection: `/items`
- single resource: `/items/$itemId`
- sub-collection: `/items/$itemId/comments`
- single sub-resource: `/items/$itemId/comments/$commentId`

Reserve verb-shaped paths for genuine operations that do not fit CRUD, such as
`/items/$itemId/publish` or `/imports/$importId/run`. Keep them rare.

## Route Module Responsibility

A route module should be responsible for:

- accepting HTTP input
- selecting the correct use case
- mapping use-case results into a response
- returning typed loader data when applicable
- rendering route-level error UI through an exported `ErrorBoundary` when the
  route owns error presentation

A route module should not contain:

- business invariants
- direct data-client queries
- multi-step orchestration that belongs to a use case
- shared transport DTOs for other features

## Route Type Safety

Type loaders, actions, and route components with the generated route types
instead of hand-written argument or data types:

```ts
import type { Route } from "./+types/items.$itemId";

export async function loader({ params }: Route.LoaderArgs) {
  // ...
}

export default function ItemDetail({ loaderData }: Route.ComponentProps) {
  // ...
}
```

Type generation runs automatically with `react-router dev`; keep the
project's `typecheck` script running `react-router typegen && tsc` so CI and
editors see the same types.

## Method And Handler Mapping

Use this default mapping inside a single route file:

- `GET`: `loader` returns the read model
- `POST`: `action` handles creation
- `PUT` or `PATCH`: `action` handles update
- `DELETE`: `action` handles deletion
- read methods other than `GET` are usually a smell; prefer `GET` for reads

For `action`, branch on `request.method` to keep mutation logic together for
the same resource.

## Single-Resource Route Rule

Co-locate single-resource and sub-resource routes by URL shape, not by HTTP
method.

Prefer:

- `app/routes/items.tsx` for the collection
- `app/routes/items.$itemId.tsx` for the single item
- `app/routes/items.$itemId.comments.tsx` for the comment sub-collection
- `app/routes/items.$itemId.comments.$commentId.tsx` for a single comment

Avoid separate files such as `items.get.tsx`, `items.post.tsx`, or
`items.delete.tsx`. Method dispatch belongs inside the route module.

## Nesting And Index Caveat

Dots create layout nesting as well as URL segments:

- `items.tsx` becomes the parent layout of `items.$itemId.tsx`, so the child
  renders inside the parent's `<Outlet />`.
- When `items.tsx` acts as a section layout, put the collection page in
  `items._index.tsx`.
- Use a trailing underscore (`items_.$itemId.tsx`) when the detail page must
  not nest inside the collection layout.

Resource routes without a component are unaffected: a request to
`/api/items/123` invokes only the matching leaf module's loader or action.

## Internal API Path Rule

When the same resource needs an internal JSON API for the SPA client, prefer
prefixing it with `api`.

Use this convention:

- UI page: `app/routes/items.tsx` for `/items`
- internal JSON: `app/routes/api.items.ts` for `/api/items`

A JSON-only route is a resource route: it exports a loader or action but no
component, so it stays a `.ts` file.

Keep both routes thin and let them share the same use case from
`app/lib/server/usecase/`.

## Action And Loader Body Rule

Prefer narrow handlers.

A route handler should:

- parse and validate input
- call a use case
- map the result into a `Response` or loader data

It should not:

- contain business decisions
- compose multiple unrelated use cases
- share local helpers across unrelated routes

Promote shared logic into a use case rather than into a sibling route module.

## Status Code Defaults

Use these defaults:

- `200 OK` for successful reads and updates returning data
- `201 Created` for resource creation, with the new resource id or location
- `204 No Content` for delete success without a body
- `400 Bad Request` for malformed transport input
- `401 Unauthorized` for missing or invalid authentication
- `403 Forbidden` for authenticated but disallowed actions
- `404 Not Found` for unknown resource ids
- `409 Conflict` for state conflicts such as version mismatches

Map domain or use-case errors to these codes at the route boundary, not inside
the use case itself. In UI routes, `throw` a `Response` (for example a 404)
from the loader and render it in the nearest exported `ErrorBoundary`; return
status responses with JSON bodies from `/api/*` resource routes.
