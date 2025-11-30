# Skibidoo Documentation

Zentrale Dokumentation für das Skibidoo E-Commerce System.

## Projekte

| Repository | Beschreibung | Docs |
|------------|--------------|------|
| [skibidoo-core](https://github.com/casoon/skibidoo-core) | Backend API | [API Docs](./api/) |
| [skibidoo-admin](https://github.com/casoon/skibidoo-admin) | Admin Dashboard | [Admin Guide](./admin/) |
| [skibidoo-storefront](https://github.com/casoon/skibidoo-storefront) | Kundenportal | [Storefront Guide](./storefront/) |
| [skibidoo-ui](https://github.com/casoon/skibidoo-ui) | UI Components | [Component Docs](./ui/) |
| [fragment-renderer](https://github.com/casoon/fragment-renderer) | Astro Fragment Renderer | [Fragment Docs](./fragment-renderer/) |

## Schnellstart

```bash
# Alle Repositories klonen
git clone https://github.com/casoon/skibidoo-core.git
git clone https://github.com/casoon/skibidoo-admin.git
git clone https://github.com/casoon/skibidoo-storefront.git
git clone https://github.com/casoon/skibidoo-infra.git

# Infrastruktur starten
cd skibidoo-infra
docker-compose up -d

# Backend starten
cd ../skibidoo-core
pnpm install
pnpm dev

# Admin starten
cd ../skibidoo-admin
pnpm install
pnpm dev

# Storefront starten
cd ../skibidoo-storefront
pnpm install
pnpm dev
```

## Architektur

- **Backend**: Node.js 24, Hono, Drizzle ORM, PostgreSQL, Redis
- **Frontend**: Astro 5, HTMX, Alpine.js, Tailwind CSS (AHAT-Stack)
- **API**: tRPC (Admin), REST (Storefront)
- **Payments**: Stripe, PayPal
- **Storage**: S3/MinIO

## Dokumentation

- [API Reference](./api/README.md)
- [Architektur](./architecture/README.md)
- [Deployment](./deployment/README.md)
- [Development Guide](./development/README.md)

## Lizenz

LGPL-3.0

