# API Reference

## REST API (Storefront)

Base URL: `http://localhost:3000/api`

### Products

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/products` | GET | List products |
| `/products/:slug` | GET | Get product by slug |
| `/products/search` | GET | Search products |

### Categories

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/categories` | GET | List categories |
| `/categories/:slug` | GET | Get category |

### Cart

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/cart` | GET | Get cart |
| `/cart/add` | POST | Add to cart |
| `/cart/update` | POST | Update quantity |
| `/cart/remove` | POST | Remove item |

### Checkout

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/checkout/intent` | POST | Create checkout intent |
| `/checkout/status` | GET | Get checkout status |

### Auth

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/register` | POST | Register customer |
| `/auth/login` | POST | Login |
| `/auth/refresh` | POST | Refresh token |

## tRPC API (Admin)

Endpoint: `http://localhost:3000/trpc`

### Routers

- `auth` - Authentication
- `products` - Product CRUD
- `categories` - Category CRUD
- `orders` - Order management
- `customers` - Customer management
- `inventory` - Stock management
- `shipping` - Shipping zones/methods

## Webhooks

| Event | Description |
|-------|-------------|
| `order.created` | New order placed |
| `order.paid` | Payment received |
| `order.shipped` | Order shipped |
| `stock.low` | Low stock alert |

