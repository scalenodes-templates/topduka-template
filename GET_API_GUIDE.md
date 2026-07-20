# TopDuka GET API Guide

This guide documents every public GET endpoint and all query parameters currently implemented by TopDuka. Paths are relative to `/pb/v1` and require a store API key:

```http
Authorization: Bearer ak_your_store_api_key
```

`x-api-key: ak_your_store_api_key` is also supported. Keep the key in server-side code.

## Products

### `GET /products`

The endpoint always returns a JSON array. Every query parameter is optional.

| Parameter | Meaning |
| --- | --- |
| `id` | Exact product UUID. This forces `skip=0`. |
| `sku` | Exact SKU. |
| `barcode` | Exact barcode. |
| `search_term` | Case-insensitive partial match against name, SKU, description, short description, category names, and tag names. |
| `status` | Exact `draft`, `active`, `inactive`, or `archived`; `all` includes every status. Omitted status excludes archived products. |
| `skip` | Number of matching rows to skip. Default `0`. |
| `limit` | Maximum rows returned. Default `10` when omitted, zero, or negative. Prefer `1`–`100`. |
| `category_id` | Exact category UUID assigned to the product. |
| `category` | Category UUID, slug, or normalized name; also matches the assigned category's parent. Aliases: `collection`, `category_key`. |
| `brand` | Value from a `Brand: value` tag. Alias: `brands`. |
| `series` | Value from a `Series: value` tag. |
| `product_type` | Category UUID/slug/name or a `Product Type:`, `Type:`, or `Category:` tag. Aliases: `productType`, `product_types`. |
| `color` | Value from a `Color:` or `Colour:` tag. Alias: `colors`. |
| `availability` | In stock: `in_stock`, `in-stock`, `available`. Out of stock: `out_of_stock`, `out-of-stock`, `unavailable`. |
| `sale` | `true`, `sale`, `on_sale`, or `on-sale` selects products whose positive `sales_price` is lower than `price`. |
| `source` | Applies a catalog collection/filter and ordering described below. |

Repeat a multi-value parameter or comma-separate it:

```text
?brand=Nike&brand=Adidas
?brand=Nike,Adidas
```

Values in one group are ORed; different groups are ANDed. Therefore:

```text
?brand=Nike&brand=Adidas&color=Black&availability=in_stock
```

means `(Nike OR Adidas) AND Black AND in stock`.

Product `slug` is not currently an effective lookup. Use `id`, `sku`, or `barcode`. Category slugs work through `category` and `/categories?slug=...`.

### Collection sources

| Collection | `source` values | Result |
| --- | --- | --- |
| All/newest default | `all_products`, omitted, or unrecognized | Filtered products ordered newest first. |
| New arrivals | `new_arrivals`, `new-arrivals`, `newest` | Newest products first. |
| On sale | `discounted`, `on_sale`, `on-sale`, `sale` | Valid discounted products, newest first. |
| Popular | `popular`, `top_rated`, `top-rated` | Rating descending, then newest. |
| Best selling | `most_sold`, `most-sold`, `best_selling`, `best-selling`, `best_sellers`, `best-sellers` | Non-cancelled/non-refunded quantity sold descending, then newest. |
| In stock | `in_stock`, `in-stock`, `available` | Available or unlimited-stock products, newest first. |

Common examples:

```text
# Active catalog
/pb/v1/products?status=active&limit=24

# Newest products
/pb/v1/products?source=newest&status=active&limit=12

# Products on sale in a category
/pb/v1/products?source=on-sale&category=shoes&limit=12

# Best sellers
/pb/v1/products?source=best-selling&status=active&limit=8

# Search
/pb/v1/products?search_term=waterproof&status=active

# Combined facets
/pb/v1/products?brand=Nike,Adidas&color=Black&availability=in_stock

# Exact product lookup (still returns an array)
/pb/v1/products?id=PRODUCT_UUID
```

### Dedicated collection routes

| Route | Ordering/filter |
| --- | --- |
| `/products/popular` | Highest rated first. |
| `/products/discounted` | Only valid discounted products. |
| `/products/new-arrivals` | Newest first. |
| `/products/most-sold` | Quantity sold first. |
| `/products/best-selling` | Alias of most sold. |

These routes force `status=active`. They accept:

- `skip`, `limit`
- `category`, `collection`, `category_key`
- `brand`, `brands`
- `series`
- `product_type`, `productType`, `product_types`
- `color`, `colors`
- `availability`, `sale`

They do not read `id`, `sku`, `barcode`, `search_term`, `status`, or `category_id`. Use `/products` for those parameters.

## Categories

### `GET /categories`

| Parameter | Meaning |
| --- | --- |
| `slug` | Exact category slug. |
| `is_active` | Exact `true` or `false`. |

Without filters, categories are ordered by `sort_order`, then name.

### `GET /categories/:id/products`

`:id` is a category UUID. The route returns products directly assigned to that category. It currently has no supported query filters or pagination.

## Availability

### `GET /products/:id/availability`

`:id` is a product UUID. There are no query parameters. It returns slots from the current time through the next 30 days with `starts_at`, `ends_at`, and `remaining`.

## Cart

### `GET /cart`

| Parameter | Required | Meaning |
| --- | --- | --- |
| `session_id` | Yes | Cart session UUID returned by `POST /cart`. |

Example: `/pb/v1/cart?session_id=CART_UUID`.

## Shipping

### `GET /shipping/zones`

There are no query parameters. The endpoint returns active public zones with their configured active delivery methods. Calculate methods eligible for a specific checkout using `POST /shipping/rates`.

## Orders

### `GET /orders`

| Parameter | Meaning |
| --- | --- |
| `skip` | Number of orders to skip. Default `0`. |

The endpoint returns 10 orders per request, newest first. It has no public `limit` parameter.

### `GET /orders/:order_number`

Use the positive numeric TopDuka order number, not the order UUID. Returns full order information.

### `GET /orders/track`

| Parameter | Required | Meaning |
| --- | --- | --- |
| `order_number` | Yes | Positive numeric TopDuka order number. |

## Payments and store configuration

These endpoints have no path or query parameters:

| Route | Result |
| --- | --- |
| `/payments/config` | Public payment availability/configuration. |
| `/config` | Currency, VAT, logo, cash availability, Paystack public key, and provider configuration flags. |
| `/store-info` | Public store identity and contact information. |

Use `/config` instead of hard-coding currency, VAT, branding, or enabled payment options.

## MCP discovery

When connected to `https://YOUR_TOPDUKA_API/mcp`, read `topduka://api/reference` or call `topduka_api_reference` for the same live contract. The `list_products` MCP tool exposes exact lookup, search, status, category, brand, series, product-type, color, availability, sale, source, and pagination inputs.
