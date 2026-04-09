# TireTutor Wholesale API

REST API for third-party systems that need to search tires and place orders on behalf of a TireTutor wholesale user.

Interactive documentation:
[https://tiretutorinc.github.io/wholesale-api-docs](https://tiretutorinc.github.io/wholesale-api-docs)

## Overview

The API supports a simple wholesale ordering flow:

1. Exchange partner and user credentials for a short-lived bearer token.
2. Search the catalog or fetch tire details.
3. Preview a purchase.
4. Create an order.
5. Fetch order details.

This API does not use an interactive consent flow. The wholesale user obtains their `user_id` and `user_key` from the TireTutor Wholesale Portal and provides them to the third-party system out-of-band.

## Base URL

Sandbox:

```text
https://staging-wholesale-api.tiretutor.dev/v1
```

## Authentication

Authentication is based on a partner + user credential exchange.

1. Call `POST /authorize` with:
   - `credentials.partner.id`
   - `credentials.partner.key`
   - `credentials.user.id`
   - `credentials.user.key`
2. Receive a short-lived TireTutor-issued bearer token.
3. Send that token on subsequent requests:

```http
Authorization: Bearer <token>
```

The bearer token should be treated as opaque.

## Endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/authorize` | Exchange partner + user credentials for an access token |
| `GET` | `/catalog/tires/search` | Search tires by SKU or size |
| `GET` | `/catalog/tires/{product_id}` | Get tire details by `product_id` |
| `POST` | `/purchasing/preview` | Preview pricing, taxes, and availability without creating an order |
| `POST` | `/purchasing/purchase` | Create an order |
| `GET` | `/orders/{order_id}` | Fetch order details |

## Common Flows

### Search tires

`GET /catalog/tires/search` supports:

- `sku`
- `width` + `rim_diameter`
- optional `aspect_ratio`
- optional pagination with `offset` and `limit`

The response returns tire summaries, availability information, pricing, and an optional opaque `next` cursor.

### Get tire details

Use `GET /catalog/tires/{product_id}` when you need detailed catalog information for a tire returned by search, including model, features, UTQG, warranty, and additional specs.

### Preview a purchase

Use `POST /purchasing/preview` to validate a cart and receive:

- line-level pricing
- taxes
- totals
- estimated delivery

This does not create an order.

### Create an order

Use `POST /purchasing/purchase` to place an order for the authenticated wholesale user context.

Request body fields:

- `cart` (required)
- `po_number` (optional)
- `note` (optional)

This endpoint supports an optional `Idempotency-Key` header to make retries safer and reduce duplicate order creation.

### Get order details

Use `GET /orders/{order_id}` to fetch the current status, timestamps, line items, and totals for an order.

## Error Handling

Common error responses include:

- `400 Bad Request` for invalid input or business validation issues
- `401 Unauthorized` for invalid or missing authentication
- `404 Not Found` for missing resources such as an unknown `order_id` or `product_id`

Error payloads use a structured format with:

- `code`
- `message`
- optional `details`

## Example Authorization Request

```json
{
  "credentials": {
    "partner": {
      "id": "pt_partner_123",
      "key": "pt_partner_key_secret"
    },
    "user": {
      "id": "user_987",
      "key": "user_key_secret"
    }
  }
}
```

## Example Authorization Response

```json
{
  "token_type": "Bearer",
  "access_token": "<opaque-token>",
  "expires_in": 3600
}
```
