# API-Dokumentation - Skibidoo E-Commerce

## Übersicht

Skibidoo bietet zwei API-Typen:
- **REST API** - Für Storefront (öffentlich, cacheable)
- **tRPC API** - Für Admin (type-safe, intern)

---

## REST API (Storefront)

**Base URL:** `https://api.example.com/api/v1`

### Authentication

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "customer@example.com",
  "password": "password123"
}

Response: 200 OK
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "customer": { ... }
}
```

### Products

```http
GET /api/v1/products?page=1&limit=20&category=electronics&priceMin=10&priceMax=100&sort=price:asc
GET /api/v1/products/:slug
POST /api/v1/products/search
```

### Checkout

```http
POST /api/v1/checkout
POST /api/v1/checkout/coupon
POST /api/v1/checkout/complete
GET /api/v1/checkout/order/:orderNumber
```

**Vollständige Dokumentation:** https://api.example.com/api/docs (Scalar)

---

## tRPC API (Admin)

**Endpoint:** `https://api.example.com/trpc`

### Beispiel (TypeScript Client)

```typescript
import { createTRPCClient, httpBatchLink } from '@trpc/client';
import type { AppRouter } from '@casoon/skibidoo-core';

const client = createTRPCClient<AppRouter>({
  links: [
    httpBatchLink({
      url: 'https://api.example.com/trpc',
      headers: {
        Authorization: `Bearer ${token}`,
      },
    }),
  ],
});

// List products
const products = await client.product.list.query({
  page: 1,
  limit: 20,
});

// Create product
const newProduct = await client.product.create.mutate({
  sku: 'PROD-001',
  name: 'New Product',
  price: '29.99',
});
```

### Router-Übersicht

- `product` - CRUD, List, Search
- `order` - List, GetById, UpdateStatus
- `customer` - List, GetById
- `category` - Tree-Management
- `shipping` - Zonen, Methoden
- `payment` - Konfiguration
- `coupon` - Rabattcodes
- `tax` - Steuerklassen
- `deliveryTime` - Lieferzeiten
- `adminUser` - Benutzerverwaltung

**Vollständige Type-Definitionen:** `@casoon/skibidoo-shared`

---

## Fehlerbehandlung

```json
{
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "Product with slug 'xyz' not found",
    "statusCode": 404
  }
}
```

## Rate Limiting

- **Storefront:** 100 req/min pro IP
- **Admin:** 1000 req/min pro User

## Webhooks

```http
POST /api/v1/webhooks/stripe
POST /api/v1/webhooks/paypal
```

**Signature-Validation erforderlich!**

---

Siehe auch: `production-setup.md` für Deployment
