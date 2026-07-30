---
layout: page
title: Webhooks
permalink: /api/webhooks/
---

# Webhooks

Webhooks enable your application to receive real-time notifications when events occur in The Neat Profit. This guide covers webhook setup, event types, and best practices.

## Overview

Webhooks are HTTP POST callbacks sent to your server when specific events occur. They allow you to:

- React to inventory changes in real-time
- Get notified of variance alerts
- Track order status updates
- Monitor POS sync completions

## Webhook Events

### Inventory Events

#### inventory.updated

Sent when an inventory item's quantity changes.

```json
{
  "event": "inventory.updated",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "item_id": "item_abc123",
    "name": "Tito's Handmade Vodka",
    "previous_quantity": 12,
    "new_quantity": 10,
    "change": -2,
    "location": "main_bar",
    "updated_by": "user_123"
  }
}
```

#### inventory.low_stock

Sent when an item falls below its reorder point.

```json
{
  "event": "inventory.low_stock",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "item_id": "item_abc123",
    "name": "Tito's Handmade Vodka",
    "current_quantity": 2,
    "reorder_point": 6,
    "location": "main_bar",
    "recommended_order": 12
  }
}
```

#### inventory.count_completed

Sent when an inventory count is completed.

```json
{
  "event": "inventory.count_completed",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "count_id": "count_xyz789",
    "location": "main_bar",
    "items_counted": 156,
    "variance_detected": 3,
    "counted_by": "user_123"
  }
}
```

### Variance Events

#### variance.detected

Sent when variance exceeds the configured threshold.

```json
{
  "event": "variance.detected",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "item_id": "item_abc123",
    "name": "Tito's Handmade Vodka",
    "expected_consumption": 24,
    "actual_consumption": 28.5,
    "variance": 4.5,
    "variance_percentage": 18.75,
    "estimated_loss": 112.50,
    "severity": "high",
    "potential_causes": ["over-pouring", "theft"]
  }
}
```

#### variance.threshold_exceeded

Sent when overall variance rate exceeds the threshold.

```json
{
  "event": "variance.threshold_exceeded",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "period": "2026-01-01 to 2026-01-15",
    "variance_rate": 0.12,
    "threshold": 0.08,
    "total_loss": 2470.50,
    "location": "main_bar"
  }
}
```

### Order Events

#### order.created

Sent when a new order is created.

```json
{
  "event": "order.created",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "order_id": "order_abc123",
    "distributor_id": "dist_xyz789",
    "distributor_name": "Southern Glazer's",
    "status": "draft",
    "total_amount": 1247.50,
    "location": "main_bar"
  }
}
```

#### order.submitted

Sent when an order is submitted to the distributor.

```json
{
  "event": "order.submitted",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "order_id": "order_abc123",
    "distributor_name": "Southern Glazer's",
    "confirmation_number": "ORD-2026-001234",
    "estimated_delivery": "2026-01-20T00:00:00Z"
  }
}
```

#### order.shipped

Sent when an order is shipped.

```json
{
  "event": "order.shipped",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "order_id": "order_abc123",
    "tracking_number": "TRK123456789",
    "carrier": "FedEx",
    "estimated_delivery": "2026-01-20T00:00:00Z"
  }
}
```

#### order.delivered

Sent when an order is delivered.

```json
{
  "event": "order.delivered",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "order_id": "order_abc123",
    "delivered_at": "2026-01-20T14:30:00Z",
    "received_by": "user_123",
    "items_received": 12,
    "items_damaged": 0
  }
}
```

### POS Events

#### pos.sync_completed

Sent when a POS sync completes.

```json
{
  "event": "pos.sync_completed",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "connection_id": "pos_conn_abc123",
    "pos_type": "toast",
    "sync_id": "sync_xyz789",
    "sync_type": "full",
    "duration_seconds": 270,
    "sales_synced": 234,
    "products_synced": 156,
    "variance_detected": 3,
    "status": "success"
  }
}
```

#### pos.sync_failed

Sent when a POS sync fails.

```json
{
  "event": "pos.sync_failed",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "connection_id": "pos_conn_abc123",
    "pos_type": "toast",
    "sync_id": "sync_xyz789",
    "error_code": "authentication_failed",
    "error_message": "Invalid POS credentials",
    "retry_after": 3600
  }
}
```

## Managing Webhooks

### List Webhooks

Retrieve all configured webhooks.

```
GET /v1/webhooks
```

**Response:**

```json
{
  "data": [
    {
      "id": "webhook_abc123",
      "url": "https://your-server.com/webhook",
      "events": ["inventory.updated", "variance.detected"],
      "status": "active",
      "created_at": "2026-01-15T10:30:00Z"
    }
  ]
}
```

### Create Webhook

Create a new webhook subscription.

```
POST /v1/webhooks
```

**Request Body:**

```json
{
  "url": "https://your-server.com/webhook",
  "events": ["inventory.updated", "variance.detected", "order.shipped"],
  "secret": "your-webhook-secret"
}
```

**Response:**

```json
{
  "data": {
    "id": "webhook_abc123",
    "url": "https://your-server.com/webhook",
    "events": ["inventory.updated", "variance.detected", "order.shipped"],
    "secret": "whsec_abc123",
    "status": "active",
    "created_at": "2026-01-15T10:30:00Z"
  }
}
```

### Get Webhook

Retrieve details for a specific webhook.

```
GET /v1/webhooks/:id
```

**Response:**

```json
{
  "data": {
    "id": "webhook_abc123",
    "url": "https://your-server.com/webhook",
    "events": ["inventory.updated", "variance.detected"],
    "status": "active",
    "delivery_stats": {
      "total_delivered": 1523,
      "total_failed": 2,
      "last_delivery": "2026-01-15T10:30:00Z"
    },
    "created_at": "2026-01-10T08:00:00Z"
  }
}
```

### Update Webhook

Update an existing webhook.

```
PUT /v1/webhooks/:id
```

**Request Body:**

```json
{
  "events": ["inventory.updated", "variance.detected", "order.shipped"],
  "status": "active"
}
```

**Response:**

```json
{
  "data": {
    "id": "webhook_abc123",
    "events": ["inventory.updated", "variance.detected", "order.shipped"],
    "status": "active",
    "updated_at": "2026-01-15T11:00:00Z"
  }
}
```

### Delete Webhook

Delete a webhook subscription.

```
DELETE /v1/webhooks/:id
```

**Response:**

```json
{
  "data": {
    "id": "webhook_abc123",
    "deleted": true
  }
}
```

## Signature Verification

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

### Handling Webhooks

```javascript
app.post('/webhook', (req, res) => {
  const signature = req.headers['x-webhook-signature'];
  const payload = JSON.stringify(req.body);
  
  // Verify signature
  if (!verifyWebhookSignature(payload, signature, process.env.WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }
  
  const event = req.body;
  
  // Handle the event
  switch (event.event) {
    case 'inventory.updated':
      handleInventoryUpdate(event.data);
      break;
    case 'variance.detected':
      handleVarianceAlert(event.data);
      break;
    case 'order.shipped':
      handleOrderShipped(event.data);
      break;
  }
  
  res.status(200).send('OK');
});
```

## Best Practices

### Respond Quickly

Acknowledge webhook deliveries within 5 seconds:

```javascript
app.post('/webhook', (req, res) => {
  // Acknowledge immediately
  res.status(200).send('OK');
  
  // Process asynchronously
  processWebhook(req.body);
});
```

### Implement Retry Logic

Handle temporary failures gracefully:

```javascript
async function processWebhook(event) {
  const maxRetries = 3;
  let attempt = 0;
  
  while (attempt < maxRetries) {
    try {
      await handleEvent(event);
      return;
    } catch (error) {
      attempt++;
      if (attempt === maxRetries) {
        logError('Failed to process webhook', error);
        return;
      }
      await delay(1000 * attempt);
    }
  }
}
```

### Use Idempotency

Design your handlers to be idempotent:

```javascript
async function handleInventoryUpdate(data) {
  const { item_id, new_quantity } = data;
  
  // Use upsert to prevent duplicates
  await db.inventory.update(
    { item_id },
    { quantity: new_quantity },
    { upsert: true }
  );
}
```

### Monitor Delivery

Track webhook delivery statistics:

```javascript
const deliveryStats = {
  total: 0,
  success: 0,
  failed: 0
};

app.post('/webhook', (req, res) => {
  deliveryStats.total++;
  
  try {
    processWebhook(req.body);
    deliveryStats.success++;
  } catch (error) {
    deliveryStats.failed++;
  }
  
  res.status(200).send('OK');
});
```

## Troubleshooting

### Webhook Not Received

- Check webhook status is `active`
- Verify URL is publicly accessible
- Check firewall allows incoming requests
- Review delivery stats in webhook details

### Signature Verification Fails

- Ensure secret matches webhook configuration
- Check timestamp is within 5 minutes
- Verify payload is not modified

### Delivery Failures

- Check server response time (must be < 5 seconds)
- Verify server returns 200 status
- Review error logs for specific failure reasons

## Next Steps

- [API Overview]({{ site.baseurl }}/api/overview/) - Learn about API architecture
- [Authentication]({{ site.baseurl }}/api/authentication/) - Secure your webhooks
- [Integrations Guide]({{ site.baseurl }}/integrations/webhooks/) - Webhook integration examples
