# Architektur

## Systemübersicht

```
                    ┌─────────────────┐
                    │   Storefront    │
                    │   (Astro SSR)   │
                    └────────┬────────┘
                             │ REST API
                             ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│     Admin       │   │   skibidoo-core │   │     Redis       │
│   (Astro SSR)   │──▶│   (Hono API)    │◀──│   (Cache/Queue) │
└─────────────────┘   └────────┬────────┘   └─────────────────┘
        │ tRPC                 │
        └──────────────────────┤
                               ▼
                    ┌─────────────────┐
                    │   PostgreSQL    │
                    └─────────────────┘
```

## Tech Stack

### Backend (skibidoo-core)
- **Runtime**: Node.js 24
- **Framework**: Hono
- **ORM**: Drizzle
- **Database**: PostgreSQL 16
- **Cache/Queue**: Redis + BullMQ
- **Validation**: Zod

### Frontend (AHAT-Stack)
- **Framework**: Astro 5
- **Interactivity**: HTMX 2.0
- **Reactivity**: Alpine.js 3.14
- **Styling**: Tailwind CSS 4.0

## Patterns

### Domain-Driven Design
- Entities, Value Objects, Repositories
- Use Cases (Application Layer)
- Domain Events

### Fragment Rendering
- Server-side rendered fragments
- HTMX für partielle Updates
- @casoon/fragment-renderer

## Verzeichnisstruktur (Core)

```
src/
├── api/          # HTTP routes
├── application/  # Use cases
├── config/       # Configuration
├── db/           # Database schema, migrations
├── domain/       # Entities, repositories
├── infrastructure/
├── jobs/         # Background workers
├── payments/     # Payment adapters
└── trpc/         # tRPC routers
```

