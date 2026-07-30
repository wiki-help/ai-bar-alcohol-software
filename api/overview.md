---
layout: page
title: API Overview
permalink: /api/overview/
---

# API Overview

The Neat Profit API provides programmatic access to bar inventory management, POS integration, demand forecasting, and analytics. This guide covers the API architecture, design principles, and how to get the most out of our endpoints.

## Base URLs

- **Sandbox**: `https://sandbox-api.theneatprofit.com/v1`
- **Production**: `https://api.theneatprofit.com/v1`

All API paths are relative to the base URL. For example, the full URL for the inventory items endpoint in production would be:

```
https://api.theneatprofit.com/v1/inventory/items
```

## API Architecture

### RESTful Design

The API follows REST principles:

- **Resource-based URLs** - Each resource has a clear, hierarchical URL structure
- **HTTP methods** - Use appropriate methods (GET, POST, PUT, DELETE) for operations
- **Standard status codes** - HTTP status codes indicate success or failure
- **JSON format** - All requests and responses use JSON

### Versioning

The API is versioned using the URL path (`/v1/`). We will maintain backward compatibility within each version. When breaking changes are necessary, we will release a new version (`/v2/`) and maintain the old version for at least 12 months.

## Request Format

### Headers

All API requests must include:

```
X-API-Key: your-api-key
Content-Type: application/json
```

### Query Parameters

Use query parameters for filtering, sorting, and pagination:

```
GET /v1/inventory/items?location=main_bar&sort=name&order=asc
```

### Request Body

POST and PUT requests should include a JSON body:

```json
{
  "name": "Tito's Handmade Vodka",
  "sku": "SKU-001",
  "quantity": 12.5,
  "unit": "bottle"
}
```

## Response Format

### Success Response

```json
{
  "data": {
    "id": "item_abc123",
    "name": "Tito's Handmade Vodka",
    "sku": "SKU-001",
    "quantity": 12.5
  },
  "meta": {
    "request_id": "req_xyz789",
    "timestamp": "2026-01-15T10:30:00Z"
  }
}
```

### Error Response

```json
{
  "error": {
    "code": "invalid_parameter",
    "message": "The 'quantity' parameter must be a positive number",
    "details": {
      "parameter": "quantity",
      "value": "-5"
    }
  },
  "meta": {
    "request_id": "req_xyz789",
    "timestamp": "2026-01-15T10:30:00Z"
  }
}
```

### Pagination

List endpoints support pagination:

```json
{
  "data": [...],
  "pagination": {
    "total": 150,
    "page": 1,
    "per_page": 50,
    "total_pages": 3,
    "next_page_url": "/v1/inventory/items?page=2",
    "prev_page_url": null
  }
}
```

Pagination parameters:

- `page` - Page number (default: 1)
- `per_page` - Items per page (default: 50, max: 100)

## Rate Limiting

### Limits

- **Sandbox**: 100 requests per minute
- **Production**: 1,000 requests per minute

### Headers

Rate limit information is included in response headers:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1705334400
X-RateLimit-Reset-After: 59
```

### Handling Limits

When you exceed the rate limit, you'll receive a `429 Too Many Requests` response:

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Try again in 60 seconds."
  }
}
```

Implement exponential backoff when handling rate limits:

```javascript
async function makeRequest(url, options, retries = 3) {
  try {
    const response = await fetch(url, options);
    
    if (response.status === 429 && retries > 0) {
      const retryAfter = parseInt(response.headers.get('Retry-After') || '5');
      await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      return makeRequest(url, options, retries - 1);
    }
    
    return response;
  } catch (error) {
    if (retries > 0) {
      await new Promise(resolve => setTimeout(resolve, 1000 * (4 - retries)));
      return makeRequest(url, options, retries - 1);
    }
    throw error;
  }
}
```

## Idempotency

To prevent duplicate operations, use idempotency keys for POST requests:

```
X-Idempotency-Key: your-unique-key-here
```

Idempotency keys are valid for 24 hours. If you retry a request with the same key within that window, the API will return the original response.

## Webhooks

Webhooks provide real-time notifications when events occur:

- `inventory.updated` - Inventory quantity changed
- `inventory.low_stock` - Item below reorder point
- `variance.detected` - Variance threshold exceeded
- `order.created` - New order created
- `order.shipped` - Order shipped
- `pos.sync_completed` - POS sync completed

See the [Webhooks documentation]({{ site.baseurl }}/api/webhooks/) for details.

## SDKs

Official SDKs are available:

- [JavaScript/TypeScript]({{ site.baseurl }}/sdks/javascript/)
- [Python]({{ site.baseurl }}/sdks/python/)

SDKs handle authentication, pagination, error handling, and webhooks automatically.

## Best Practices

### Security

- Never expose API keys in client-side code
- Use environment variables for credentials
- Implement webhook signature verification
- Rotate API keys regularly
- Use HTTPS for all requests

### Performance

- Use pagination for large datasets
- Cache frequently accessed data
- Use batch endpoints when available
- Implement request batching

### Reliability

- Implement proper error handling
- Use idempotency keys for critical operations
- Set up monitoring and alerting
- Test in sandbox before production

## Available Endpoints

### Inventory
- `GET /v1/inventory/items` - List inventory items
- `POST /v1/inventory/items` - Create inventory item
- `GET /v1/inventory/items/:id` - Get item details
- `PUT /v1/inventory/items/:id` - Update item
- `DELETE /v1/inventory/items/:id` - Delete item
- `POST /v1/inventory/count` - Submit inventory count

### POS Integration
- `GET /v1/pos/connections` - List POS connections
- `POST /v1/pos/connections` - Create POS connection
- `GET /v1/pos/connections/:id` - Get connection details
- `PUT /v1/pos/connections/:id` - Update connection
- `DELETE /v1/pos/connections/:id` - Delete connection
- `POST /v1/pos/sync` - Trigger POS sync
- `GET /v1/pos/mappings` - List product mappings

### Ordering
- `GET /v1/orders` - List orders
- `POST /v1/orders` - Create order
- `GET /v1/orders/:id` - Get order details
- `PUT /v1/orders/:id` - Update order
- `DELETE /v1/orders/:id` - Cancel order
- `GET /v1/distributors` - List distributors
- `POST /v1/distributors` - Add distributor

### Analytics
- `GET /v1/analytics/variance` - Get variance report
- `GET /v1/analytics/forecast` - Get demand forecast
- `GET /v1/analytics/performance` - Get performance metrics
- `GET /v1/analytics/costing` - Get recipe costing data

### Webhooks
- `GET /v1/webhooks` - List webhooks
- `POST /v1/webhooks` - Create webhook
- `GET /v1/webhooks/:id` - Get webhook details
- `PUT /v1/webhooks/:id` - Update webhook
- `DELETE /v1/webhooks/:id` - Delete webhook

## Next Steps

- [Authentication]({{ site.baseurl }}/api/authentication/) - Learn about authentication methods
- [Inventory API]({{ site.baseurl }}/api/inventory/) - Inventory management endpoints
- [POS Integration]({{ site.baseurl }}/api/pos/) - POS sync and mapping
- [Webhooks]({{ site.baseurl }}/api/webhooks/) - Real-time notifications
