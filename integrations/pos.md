---
layout: page
title: POS Integration Guide
permalink: /integrations/pos/
---

# POS Integration Guide

This guide walks you through integrating your Point of Sale system with The Neat Profit for real-time sales sync, product mapping, and variance tracking.

## Supported POS Systems

The Neat Profit integrates with the following POS systems:

- **Toast** - Full-featured restaurant POS
- **Square** - Popular retail and restaurant POS
- **Micros** - Enterprise hospitality POS (3700/9700)
- **Clover** - Cloud-based POS system
- **ShopKeep** - Retail POS
- **Lightspeed** - Multi-location POS
- **Upserve** - Restaurant management system

## Integration Overview

A POS integration enables:

- **Real-time Sales Sync** - Sales data automatically updates inventory
- **Product Mapping** - Match POS items to inventory products
- **Variance Detection** - Compare expected vs. actual consumption
- **Automated Reconciliation** - Streamline end-of-shift processes

## Step 1: Get POS Credentials

### Toast

1. Log in to your Toast dashboard
2. Navigate to Partners > API Keys
3. Create a new API key with read permissions
4. Note your Client ID, Client Secret, and Restaurant ID

### Square

1. Log in to Square Developer Portal
2. Create a new application
3. Generate access token with inventory permissions
4. Note your Application ID and Access Token

### Micros

1. Contact Micros support for API access
2. Obtain API credentials and endpoint URL
3. Configure firewall to allow API access

## Step 2: Create POS Connection

Use the POS Integration API to create a connection:

```bash
curl -X POST "https://api.theneatprofit.com/v1/pos/connections" \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "pos_type": "toast",
    "name": "Main Bar Toast",
    "credentials": {
      "client_id": "your-client-id",
      "client_secret": "your-client-secret",
      "restaurant_id": "your-restaurant-id"
    },
    "sync_frequency": "realtime"
  }'
```

**Response:**

```json
{
  "data": {
    "id": "pos_conn_abc123",
    "pos_type": "toast",
    "name": "Main Bar Toast",
    "status": "active",
    "last_sync": "2026-01-15T10:30:00Z",
    "sync_frequency": "realtime"
  }
}
```

## Step 3: Trigger Initial Sync

Trigger a full sync to import all POS data:

```bash
curl -X POST "https://api.theneatprofit.com/v1/pos/connections/pos_conn_abc123/sync" \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "sync_type": "full"
  }'
```

Monitor sync status:

```bash
curl -X GET "https://api.theneatprofit.com/v1/pos/syncs/sync_xyz789" \
  -H "X-API-Key: your-api-key"
```

## Step 4: Map Products

### Manual Mapping

Map individual POS items to inventory items:

```bash
curl -X POST "https://api.theneatprofit.com/v1/pos/connections/pos_conn_abc123/mappings" \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "pos_item_id": "item_toast_123",
    "inventory_item_id": "item_abc123",
    "conversion_factor": 1.5
  }'
```

### Auto-Mapping

Automatically map items based on name matching:

```bash
curl -X POST "https://api.theneatprofit.com/v1/pos/connections/pos_conn_abc123/auto-map" \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "confidence_threshold": 0.85
  }'
```

**Response:**

```json
{
  "data": {
    "mappings_created": 45,
    "mappings_review": 12,
    "unmapped_items": 23
  }
}
```

### Review and Adjust

Review auto-mapped items and adjust as needed:

```bash
curl -X GET "https://api.theneatprofit.com/v1/pos/connections/pos_conn_abc123/mappings?mapped=true" \
  -H "X-API-Key: your-api-key"
```

## Step 5: Configure Conversion Factors

Set conversion factors to accurately map POS units to inventory units:

| POS Unit | Inventory Unit | Conversion Factor |
|----------|---------------|-------------------|
| Drink (1.5 oz) | Bottle (750 ml) | 0.02 |
| Shot (1 oz) | Bottle (750 ml) | 0.013 |
| Pitcher (64 oz) | Keg (1984 oz) | 0.032 |

Example:

```json
{
  "pos_item_id": "item_toast_123",
  "inventory_item_id": "item_abc123",
  "conversion_factor": 0.02
}
```

## Step 6: Set Up Webhooks

Configure webhooks to receive real-time sync notifications:

```bash
curl -X POST "https://api.theneatprofit.com/v1/webhooks" \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/webhook",
    "events": ["pos.sync_completed", "pos.sync_failed"],
    "secret": "your-webhook-secret"
  }'
```

## Step 7: Monitor and Maintain

### Check Sync Health

Regularly monitor sync statistics:

```bash
curl -X GET "https://api.theneatprofit.com/v1/pos/connections/pos_conn_abc123" \
  -H "X-API-Key: your-api-key"
```

Review `sync_stats` in the response:
- `total_syncs` - Total number of syncs
- `failed_syncs` - Number of failed syncs
- `last_sync_duration_ms` - Duration of last sync

### Handle Sync Failures

If sync fails, check the error details:

```json
{
  "error": {
    "code": "authentication_failed",
    "message": "Invalid POS credentials"
  }
}
```

Common issues:
- **Authentication Failed** - Verify POS credentials are correct
- **Rate Limit Exceeded** - Reduce sync frequency
- **Network Error** - Check network connectivity

### Update Mappings

As your menu changes, update product mappings:

```bash
curl -X PUT "https://api.theneatprofit.com/v1/pos/connections/pos_conn_abc123/mappings/item_toast_123" \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "conversion_factor": 0.025
  }'
```

## Best Practices

### Choose the Right Sync Frequency

- **Real-time** - High-volume bars, immediate variance detection
- **Hourly** - Moderate volume, balance between timeliness and performance
- **Daily** - Low volume, end-of-day reconciliation

### Set Appropriate Conversion Factors

Accurate conversion factors are critical for variance detection:

- Test conversion factors with manual counts
- Adjust based on actual pour sizes
- Account for spillage and waste

### Monitor Variance Patterns

Use the variance data to identify issues:

```javascript
const variance = await getVarianceReport();
const highVariance = variance.data.items.filter(item => 
  item.variance_percentage > 15
);

if (highVariance.length > 0) {
  alertTeam('High variance items detected');
}
```

## Troubleshooting

### Sync Not Running

1. Check connection status is `active`
2. Verify POS credentials are valid
3. Ensure network connectivity to POS API
4. Review sync logs for specific errors

### Incorrect Variance

1. Verify conversion factors are accurate
2. Check product mappings are correct
3. Ensure POS data is complete
4. Review inventory count accuracy

### Webhook Not Received

1. Verify webhook URL is publicly accessible
2. Check webhook status is `active`
3. Review delivery statistics
4. Test webhook endpoint manually

## Example Integration

Complete example using the JavaScript SDK:

```javascript
import { NeatProfitAPI } from '@neatprofit/sdk';

const api = new NeatProfitAPI({
  apiKey: process.env.API_KEY
});

// 1. Create POS connection
const connection = await api.pos.createConnection({
  pos_type: 'toast',
  name: 'Main Bar Toast',
  credentials: {
    client_id: process.env.TOAST_CLIENT_ID,
    client_secret: process.env.TOAST_CLIENT_SECRET,
    restaurant_id: process.env.TOAST_RESTAURANT_ID
  },
  sync_frequency: 'realtime'
});

// 2. Trigger initial sync
const sync = await api.pos.triggerSync(connection.id, {
  sync_type: 'full'
});

// 3. Wait for sync completion
await api.pos.waitForSync(sync.sync_id);

// 4. Auto-map products
const mapping = await api.pos.autoMapProducts(connection.id, {
  confidence_threshold: 0.85
});

console.log(`Created ${mapping.mappings_created} mappings`);

// 5. Set up webhook
await api.webhooks.create({
  url: 'https://your-server.com/webhook',
  events: ['pos.sync_completed', 'variance.detected'],
  secret: process.env.WEBHOOK_SECRET
});
```

## Next Steps

- [POS API Reference]({{ site.baseurl }}/api/pos/) - Complete API documentation
- [Webhooks Guide]({{ site.baseurl }}/api/webhooks/) - Set up real-time notifications
- [Analytics API]({{ site.baseurl }}/api/analytics/) - Access variance reports
