---
layout: page
title: Authentication
permalink: /api/authentication/
---

# Authentication

The Neat Profit API uses API key authentication to secure access to your data. This guide covers authentication methods, best practices, and security considerations.

## API Key Authentication

### Getting Your API Key

1. Log in to your The Neat Profit account
2. Navigate to Settings > Developer Portal
3. Click "Generate New API Key"
4. Copy your API key and secret

### Making Authenticated Requests

Include your API key in the `X-API-Key` header:

```bash
curl -X GET "https://api.theneatprofit.com/v1/inventory/items" \
  -H "X-API-Key: your-api-key-here"
```

### Example in JavaScript

```javascript
const response = await fetch('https://api.theneatprofit.com/v1/inventory/items', {
  headers: {
    'X-API-Key': 'your-api-key-here',
    'Content-Type': 'application/json'
  }
});
```

### Example in Python

```python
import requests

headers = {
    'X-API-Key': 'your-api-key-here',
    'Content-Type': 'application/json'
}

response = requests.get(
    'https://api.theneatprofit.com/v1/inventory/items',
    headers=headers
)
```

## API Key Permissions

API keys can be scoped with specific permissions. When creating an API key, choose the minimum permissions needed:

| Permission | Description |
|------------|-------------|
| `inventory:read` | Read inventory data |
| `inventory:write` | Create, update, delete inventory items |
| `pos:read` | Access POS data and mappings |
| `pos:write` | Modify POS connections and mappings |
| `ordering:read` | View orders and distributor data |
| `ordering:write` | Create and manage orders |
| `analytics:read` | Access analytics and reports |
| `webhooks:manage` | Create and manage webhooks |

### Example: Scoped API Key

When creating an API key for a POS integration that only needs to read data:

```json
{
  "name": "POS Integration - Read Only",
  "permissions": ["pos:read", "inventory:read"]
}
```

## Security Best Practices

### Never Expose API Keys

❌ **Don't do this:**

```javascript
// Client-side code (never expose API keys)
const apiKey = "sk_live_abc123"; // This is visible to users!
```

✅ **Do this instead:**

```javascript
// Server-side code
const apiKey = process.env.API_KEY;
```

### Use Environment Variables

Store API keys in environment variables:

```bash
# .env file
NEAT_PROFIT_API_KEY=sk_live_abc123
```

```javascript
// Access in code
const apiKey = process.env.NEAT_PROFIT_API_KEY;
```

### Rotate API Keys Regularly

- Rotate API keys every 90 days
- Immediately rotate if a key is compromised
- Use different keys for different environments

### Use Separate Keys for Different Environments

- **Sandbox Key** - For development and testing
- **Production Key** - For live data

Never use production keys in development environments.

## Webhook Signature Verification

Webhooks include a signature header for security:

```
X-Webhook-Signature: t=1234567890,v1=abc123...
```

### Verifying Signatures

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const [timestamp, signatureValue] = signature.split(',');
  const t = timestamp.split('=')[1];
  const v1 = signatureValue.split('=')[1];
  
  // Check timestamp (reject if older than 5 minutes)
  if (Math.floor(Date.now() / 1000) - t > 300) {
    return false;
  }
  
  // Compute expected signature
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(`${t}.${payload}`)
    .digest('hex');
  
  return v1 === expectedSignature;
}
```

## Error Handling

### Invalid API Key

```json
{
  "error": {
    "code": "invalid_api_key",
    "message": "The provided API key is invalid or expired"
  }
}
```

### Insufficient Permissions

```json
{
  "error": {
    "code": "insufficient_permissions",
    "message": "API key does not have permission to perform this action",
    "details": {
      "required": "inventory:write",
      "provided": ["inventory:read"]
    }
  }
}
```

## Testing Authentication

### Test Your API Key

```bash
curl -X GET "https://api.theneatprofit.com/v1/auth/test" \
  -H "X-API-Key: your-api-key"
```

Response:

```json
{
  "data": {
    "valid": true,
    "permissions": ["inventory:read", "inventory:write"],
    "rate_limit": {
      "limit": 1000,
      "remaining": 999
    }
  }
}
```

## Troubleshooting

### Common Issues

**Issue**: 401 Unauthorized
- **Cause**: Invalid or missing API key
- **Solution**: Verify your API key is correct and included in headers

**Issue**: 403 Forbidden
- **Cause**: Insufficient permissions
- **Solution**: Check that your API key has the required permissions

**Issue**: 429 Too Many Requests
- **Cause**: Rate limit exceeded
- **Solution**: Implement rate limiting in your application

## Next Steps

- [API Overview]({{ site.baseurl }}/api/overview/) - Learn about API architecture
- [Getting Started]({{ site.baseurl }}/getting-started/) - Set up your first integration
- [Webhooks]({{ site.baseurl }}/api/webhooks/) - Set up real-time notifications
