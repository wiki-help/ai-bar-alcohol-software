---
layout: page
title: Getting Started with The Neat Profit API
permalink: /getting-started/
---

# Getting Started with The Neat Profit API

Welcome to your comprehensive guide for integrating with The Neat Profit API. This guide will help you set up your integration, authenticate, and make your first API call.

## Prerequisites

Before you begin, make sure you have:

- A The Neat Profit account ([sign up here](https://theneatprofit.com))
- Basic knowledge of REST APIs
- A code editor or API testing tool (like Postman or cURL)
- Your preferred programming language's HTTP client library

## Getting Your API Key

### 1. Access the Developer Portal

Log in to your The Neat Profit account and navigate to Settings > Developer Portal.

### 2. Create an API Key

Click "Generate New API Key" to create your credentials. You'll receive:

- **API Key** - Your unique identifier for authentication
- **API Secret** - Used for signing requests (keep this secure!)

### 3. Set Permissions

Choose the permissions your integration needs:

- `inventory:read` - Read inventory data
- `inventory:write` - Modify inventory
- `pos:read` - Access POS data
- `pos:write` - Modify POS mappings
- `ordering:read` - View orders
- `ordering:write` - Create and manage orders
- `analytics:read` - Access analytics and reports
- `webhooks:manage` - Manage webhook subscriptions

⚠️ **Security Tip**: Only request the minimum permissions your integration needs.

## Authentication

The Neat Profit API uses API key authentication. Include your API key in the `X-API-Key` header:

```bash
curl -X GET "https://api.theneatprofit.com/v1/inventory" \
  -H "X-API-Key: your-api-key-here"
```

### Environment

The API has two environments:

- **Sandbox** - `https://sandbox-api.theneatprofit.com` - For testing
- **Production** - `https://api.theneatprofit.com` - For live data

Use the sandbox environment during development to avoid affecting real inventory data.

## Making Your First API Call

Let's retrieve your inventory items as a test:

```bash
curl -X GET "https://sandbox-api.theneatprofit.com/v1/inventory/items" \
  -H "X-API-Key: your-sandbox-api-key"
```

**Response:**

```json
{
  "data": [
    {
      "id": "item_abc123",
      "name": "Tito's Handmade Vodka",
      "sku": "SKU-001",
      "quantity": 12.5,
      "unit": "bottle",
      "location": "main_bar",
      "last_updated": "2026-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 150,
    "page": 1,
    "per_page": 50
  }
}
```

## Setting Up Webhooks

Webhooks allow your application to receive real-time notifications when events occur in The Neat Profit.

### 1. Create a Webhook Endpoint

Create an endpoint on your server that can receive POST requests:

```javascript
app.post('/webhook', (req, res) => {
  const event = req.body;
  
  // Verify the webhook signature
  const signature = req.headers['x-webhook-signature'];
  if (!verifySignature(signature, event)) {
    return res.status(401).send('Invalid signature');
  }
  
  // Handle the event
  switch (event.type) {
    case 'inventory.updated':
      handleInventoryUpdate(event.data);
      break;
    case 'variance.detected':
      handleVarianceAlert(event.data);
      break;
  }
  
  res.status(200).send('OK');
});
```

### 2. Register the Webhook

```bash
curl -X POST "https://sandbox-api.theneatprofit.com/v1/webhooks" \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/webhook",
    "events": ["inventory.updated", "variance.detected"],
    "secret": "your-webhook-secret"
  }'
```

## Rate Limits

The API has the following rate limits:

- **Sandbox**: 100 requests per minute
- **Production**: 1,000 requests per minute

Rate limit headers are included in every response:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1705334400
```

## Error Handling

The API uses standard HTTP status codes:

- `200 OK` - Request succeeded
- `201 Created` - Resource created successfully
- `400 Bad Request` - Invalid request parameters
- `401 Unauthorized` - Invalid or missing API key
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `429 Too Many Requests` - Rate limit exceeded
- `500 Server Error` - Internal server error

Error response format:

```json
{
  "error": {
    "code": "invalid_parameter",
    "message": "The 'quantity' parameter must be a positive number",
    "details": {
      "parameter": "quantity",
      "value": "-5"
    }
  }
}
```

## Best Practices

### Security

- Never commit API keys to version control
- Use environment variables to store credentials
- Rotate API keys regularly
- Implement webhook signature verification
- Use HTTPS for all API calls

### Performance

- Use pagination for large datasets
- Cache frequently accessed data
- Implement exponential backoff for retries
- Use batch endpoints when available

### Reliability

- Implement proper error handling
- Set up monitoring for API failures
- Use idempotent operations where possible
- Test in sandbox before production

## Next Steps

Now that you're set up, explore our other documentation:

- [API Overview]({{ site.baseurl }}/api/overview/) - Complete API architecture guide
- [Authentication]({{ site.baseurl }}/api/authentication/) - Advanced authentication methods
- [Inventory API]({{ site.baseurl }}/api/inventory/) - Inventory management endpoints
- [POS Integration]({{ site.baseurl }}/integrations/pos/) - Connect your POS system
- [Webhooks]({{ site.baseurl }}/api/webhooks/) - Real-time event notifications

## Need Help?

Our developer support team is available:

- **Email**: api@theneatprofit.com
- **GitHub Issues**: [Report bugs and request features](https://github.com/wiki-help/ai-bar-alcohol-software/issues)
- **Documentation**: Browse our complete API reference

---

Ready to dive deeper? Explore our [API reference documentation]({{ site.baseurl }}/api/overview/) to learn about all available endpoints.
