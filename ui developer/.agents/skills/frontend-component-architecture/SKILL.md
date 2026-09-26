---
name: frontend-component-architecture
description: Guides React UI architecture decisions for component breakdown, state ownership, CSS Modules, TanStack Query data access, and remote-call failure states. Use when building, reviewing, or refactoring React UI, page structure, component APIs, server-state caching, or user-facing error handling.
---
# Frontend Component Architecture

Use this skill when you design, build, or review React UI.

## Core rule

Make the UI easy to change, test, and recover from failure. Prefer boring React, small components, explicit data states, and native browser behavior.

## Component breakdown

Break UI into components by responsibility:

- Page or route component: owns routing, data queries, mutations, and page-level decisions.
- Feature component: owns one user task or section.
- Presentational component: renders props and emits events.
- Small leaf component: handles markup, labels, CSS Module classes, and local-only UI details.

Do not split only because a file is long. Split when a part has a clear name, a separate reason to change, or repeated use.

## State ownership

Put state at the highest responsible level:

- Server state belongs in TanStack Query (`useQuery`, `useMutation`, cache invalidation).
- Shared UI state belongs in the nearest common parent.
- Form draft state belongs in the form unless other components must read it.
- Leaf components receive values and callbacks. They should derive display values from props.

Prefer derived state over synchronized duplicate state. Do not copy props into state unless the user can edit a local draft or you must intentionally reset it.

## Data fetching and caching

Use TanStack Query for remote data when it is available in the project:

- Put each remote read behind a query with a stable query key.
- Put each remote write behind a mutation.
- Invalidate or update affected query data after successful writes.
- Use query status to render loading, empty, error, and success states.
- Keep fetch functions small and reusable. They return data or throw a useful error.

Do not add TanStack Query if the project does not already use it unless the user asks or the feature needs shared server-state caching.

## Remote-call failure states

Each HTTP request can fail. For every query or mutation, define what the user sees and can do:

- Loading: clear progress or disabled controls where needed.
- Empty: helpful message when there is no data.
- Error: meaningful failure message, not a raw stack trace.
- Retry: visible action when retry can help.
- Partial success: show what loaded and what failed.
- Mutation failure: preserve user input and explain whether the action was saved.

Never hide remote failures in `console.error` only.

## CSS Modules

Prefer CSS Modules for component styles when the project uses modular CSS:

- Keep styles next to the component they serve.
- Use semantic class names based on role, not visual detail.
- Prefer composition and CSS custom properties over duplicated magic values.
- Keep global CSS for resets, tokens, and true app-wide rules only.

## Review checklist

Before you finish, check:

- Components have clear responsibilities.
- State lives at the highest responsible level, not lower and synchronized.
- Lower components use props and derived values.
- Remote data uses TanStack Query when appropriate.
- Each remote call has loading, empty, error, retry, and mutation-failure behavior where relevant.
- CSS is component-scoped when possible.
- The UI remains accessible: labels, roles, keyboard behavior, focus, and clear error text.
