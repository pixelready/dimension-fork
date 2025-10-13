# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a full-stack TypeScript application built with TanStack Start (React SSR framework), oRPC (type-safe RPC), and TanStack Query. The stack emphasizes type safety across the entire application from client to server, with automatic code generation for routing.

## Core Architecture

### Tech Stack Integration

- **TanStack Start**: File-based SSR framework powering the application
- **oRPC**: Type-safe RPC layer connecting client and server
- **TanStack Router**: File-based routing with automatic route tree generation
- **TanStack Query**: Data fetching and caching integrated with oRPC
- **Vite**: Build tool with Nitro plugin for server-side rendering
- **Zod**: Runtime validation for both oRPC schemas and environment variables

### Key Architectural Patterns

**Isomorphic oRPC Client** ([src/orpc/client.ts](src/orpc/client.ts))
- Uses `createIsomorphicFn()` to create a client that works both server-side and client-side
- Server-side: Uses `createRouterClient()` to call procedures directly
- Client-side: Uses `RPCLink` to make HTTP requests to `/api/rpc`
- Wrapped with TanStack Query utilities via `createTanstackQueryUtils()`

**Router Integration** ([src/router.tsx](src/router.tsx))
- Router is created with TanStack Query context via `getContext()`
- `setupRouterSsrQueryIntegration()` connects router with query client for SSR hydration
- Router wraps all routes with the TanStack Query provider

**File-Based Routing**
- Routes live in `src/routes/` directory
- Route tree is auto-generated in `src/routeTree.gen.ts` (excluded from Biome linting)
- Root route ([src/routes/__root.tsx](src/routes/__root.tsx)) defines the shell component with head meta tags
- API routes use special file naming: `api.rpc.$.ts` maps to `/api/rpc/*`

**oRPC Router Structure** ([src/orpc/router/](src/orpc/router/))
- Individual procedure files in `src/orpc/router/` (e.g., `todos.ts`)
- Main router in `src/orpc/router/index.ts` combines all procedures
- Procedures use `os` (oRPC server) builder pattern with `.input()` and `.handler()`
- API endpoint handler in `src/routes/api.rpc.$.ts` uses `RPCHandler` to process all HTTP methods

## Development Commands

### Core Development
```bash
pnpm dev          # Start dev server on port 3000
pnpm build        # Build for production
pnpm serve        # Preview production build
pnpm test         # Run tests with Vitest
```

### Code Quality (Biome)
```bash
pnpm format       # Format code (tab indentation, double quotes)
pnpm lint         # Lint code
pnpm check        # Format, lint, and organize imports
```

### Adding UI Components
```bash
pnpx shadcn@latest add <component>  # Install Shadcn component
```

## Important File Locations

- **Environment Variables**: [src/env.ts](src/env.ts) - T3 Env with runtime validation
- **oRPC Router**: [src/orpc/router/index.ts](src/orpc/router/index.ts)
- **oRPC Client**: [src/orpc/client.ts](src/orpc/client.ts)
- **oRPC Schemas**: [src/orpc/schema.ts](src/orpc/schema.ts) - Shared Zod schemas
- **Query Provider**: [src/integrations/tanstack-query/root-provider.tsx](src/integrations/tanstack-query/root-provider.tsx)
- **Vite Config**: [vite.config.ts](vite.config.ts) - Includes Nitro, TanStack Start, and Tailwind plugins

## Code Style

- **Formatter**: Biome with tab indentation and double quotes
- **Linting**: Biome recommended rules enabled
- **Auto-imports**: Organize imports on save
- **Path Aliases**: Configured via `vite-tsconfig-paths` plugin (`@/` maps to `src/`)

## Adding New Features

**Adding an oRPC Procedure:**
1. Create procedure in `src/orpc/router/<feature>.ts` using `os.input().handler()`
2. Export procedure from `src/orpc/router/index.ts`
3. Use in components via `orpc` utils from `@/orpc/client`

**Adding a Route:**
1. Create file in `src/routes/` (e.g., `about.tsx`)
2. TanStack Router auto-generates route tree
3. Use server-side data loading with route loaders or oRPC + TanStack Query

**Adding Environment Variables:**
1. Add validation schema to [src/env.ts](src/env.ts)
2. Client variables must be prefixed with `VITE_`
3. Import and use via `import { env } from '@/env'`

## Node Version

Uses Node.js version specified in `.nvmrc` for version auto-switching with nvm.
