# TeamFlow

TeamFlow is a pnpm workspace for the TeamFlow application. The repository contains a NestJS API, a Next.js web application, and shared TypeScript packages.

## Requirements

- Node.js 22 or newer
- pnpm

## Setup

```bash
pnpm install
```

## Development

Run the API and web application together:

```bash
pnpm dev
```

Run an individual application:

```bash
pnpm --filter @teamflow/api start:dev
pnpm --filter @teamflow/ui dev
```

## Validation

```bash
pnpm build
pnpm lint
pnpm typecheck
pnpm format:check
```

Run API tests directly:

```bash
pnpm --filter @teamflow/api test
pnpm --filter @teamflow/api test:e2e
```

## Repository layout

- `apps/api` - NestJS backend and API tests
- `apps/ui` - Next.js frontend
- `packages/database` - Database package
- `packages/domain` - Domain models and logic
- `packages/shared` - Shared utilities and types
- `Docs` - Project requirements and technical documentation

## Documentation

Project requirements and technical notes are available in the [`Docs`](Docs) directory.
