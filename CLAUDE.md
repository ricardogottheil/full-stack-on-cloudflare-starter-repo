# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Full-stack Cloudflare application - a platform for exploring and interacting with educational courses. Monorepo managed with **pnpm**.

## Commands

### Development
```bash
pnpm install                    # Install all dependencies
pnpm build-package              # Build @repo/data-ops (required before running apps)
pnpm dev-frontend               # Run user-application on port 3000
pnpm dev-data-service           # Run data-service worker locally
```

### Testing
```bash
pnpm --filter data-service run test         # Backend tests (Vitest + Workers pool)
pnpm --filter user-application run test     # Frontend tests (Vitest + JSDOM)
```

### Database (from packages/data-ops)
```bash
pnpm --filter @repo/data-ops run pull       # Pull schema from remote D1
pnpm --filter @repo/data-ops run generate   # Generate migrations
pnpm --filter @repo/data-ops run migrate    # Apply migrations
pnpm --filter @repo/data-ops run studio     # Open Drizzle Studio
pnpm --filter @repo/data-ops run better-auth-generate  # Regenerate auth schema
```

### Deployment
```bash
pnpm stage:deploy-frontend      # Deploy frontend to staging
pnpm production:deploy-frontend # Deploy frontend to production
```

### Type Generation
```bash
pnpm --filter data-service run stage:cf-typegen       # Generate Cloudflare types for data-service
pnpm --filter user-application run stage:cf-typegen   # Generate Cloudflare types for frontend
```

## Architecture

### Monorepo Structure
- **apps/user-application** - React 19 + Vite frontend deployed as Cloudflare Workers Assets
- **apps/data-service** - Hono.js backend on Cloudflare Workers
- **packages/data-ops** - Shared data layer (Drizzle ORM, queries, auth, types)

### Frontend Stack (user-application)
- TanStack Router with file-based routing (`src/routes/`)
- tRPC for type-safe API calls (`src/worker/trpc/`)
- Tailwind CSS 4 + Radix UI components
- Zustand for state management
- Better Auth with Google OAuth

### Backend Stack (data-service)
- Hono.js HTTP routing (`src/hono/app.ts`)
- Cloudflare D1 database via Drizzle ORM
- Durable Objects for real-time state (`src/durable-objects/`)
- Workflows for async jobs (`src/workflows/`)
- Queues for event processing (`src/queue-handlers/`)

### Data Flow
1. Frontend tRPC routes call backend via HTTP
2. Backend routes handle auth via Better Auth, data via Drizzle queries from @repo/data-ops
3. Durable Objects manage real-time click tracking with SQL storage
4. Workflows handle async destination evaluations
5. Queues process link click events asynchronously

### Key Bindings (wrangler.jsonc)
- D1 Database (DB)
- KV Cache (CACHE)
- R2 Storage (BUCKET)
- Durable Objects (LINK_CLICK_TRACKER, EVALUATION_SCHEDULER)
- Workflows (DESTINATION_EVALUATION_WORKFLOW)
- Browser API for Puppeteer rendering
- AI API for Workers AI

## Code Conventions

### TypeScript
- Strict mode enabled across all packages
- Path aliases: `@/*` maps to `src/`, `@/worker/*` maps to `worker/`
- Zod for runtime validation schemas

### Prettier
- Tabs for indentation
- Single quotes
- 140 character print width
- Semicolons required

### Database
- Use Drizzle Kit for all schema changes, not manual SQL
- Schema defined in `packages/data-ops/src/drizzle-out/`
- Queries in `packages/data-ops/src/queries/`

### Authentication
- Better Auth handles all auth flows
- Auth schema auto-generated via `better-auth-generate` command
- Protected routes use auth layout in TanStack Router

## Environment Configuration

Two environments configured in wrangler.jsonc:
- **stage**: Uses stage database, `stage.ricardogottheil.dev` domain
- **production**: Uses production database, `ricardogottheil.dev` domain

Cloudflare env vars required for database operations:
- `CLOUDFLARE_ACCOUNT_ID`
- `CLOUDFLARE_DATABASE_ID`
- `CLOUDFLARE_D1_TOKEN`
