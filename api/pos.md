---
layout: page
title: POS Integration API
permalink: /api/pos/
---

# POS Integration API

The POS Integration API enables you to connect your Point of Sale system with The Neat Profit for real-time sales sync, product mapping, and variance tracking.

## Endpoints

### List POS Connections

Retrieve all configured POS connections.

```
GET /v1/pos/connections
```

**Response:**

```json
{
  "data": [
    {
      "id": "pos_conn_abc123",
      "pos_type": "toast",
      "name": "Main Bar Toast",
      "status": "active",
      "last_sync": "2026-01-15T10:30:00Z",
      "sync_frequency": "realtime"
    }
  ]
}
```

### Create POS Connection

Create a new POS connection.

```
POST /v1/pos/connections
```

**Request Body:**

```json
{
  "pos_type": "toast",
  "name": "Main Bar Toast",
  "credentials": {
    "client_id": "your-client-id",
    "client_secret": "your-client-secret",
    "restaurant_id": "your-restaurant-id"
  },
  "sync_frequency": "realtime"
}
```

**Supported POS Types:**

- `toast` - Toast POS
- `square` - Square POS
- `micros` - Micros 3700/9700
- `clover` - Clover POS
- `shopkeep` - ShopKeep POS
- `lightspeed` - Lightspeed POS
- `upserve` - Upserve POS

**Response:**

```json
{
  "data": {
    "id": "pos_conn_abc123",
    "pos_type": "toast",
    "name": "Main Bar Toast",
    "status": "active",
    "last_sync": "2026-01-15T10:30:00Z",
    "sync_frequency": "realtime",
    "created_at": "2026-01-15T10:30:00Z"
  }
}
```

### Get POS Connection

Retrieve details for a specific POS connection.

```
GET /v1/pos/connections/:id
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
    "sync_frequency": "realtime",
    "sync_stats": {
      "total_syncs": 1523,
      "failed_syncs": 2,
      "last_sync_duration_ms": 450
    },
    "created_at": "2026-01-10T08:00:00Z"
  }
}
```

### Update POS Connection

Update an existing POS connection.

```
PUT /v1/pos/connections/:id
```

**Request Body:**

```json
{
  "sync_frequency": "hourly",
  "name": "Updated Connection Name"
}
```

**Response:**

```json
{
  "data": {
    "id": "pos_conn_abc123",
    "pos_type": "toast",
    "name": "Updated Connection Name",
    "status": "active",
    "last_sync": "2026-01-15T10:30:00Z",
    "sync_frequency": "hourly",
    "updated_at": "2026-01-15T11:00:00Z"
  }
}
```

### Delete POS Connection

Delete a POS connection.

```
DELETE /v1/pos/connections/:id
```

**Response:**

```json
{
  "data": {
    "id": "pos_conn_abc123",
    "deleted": true
  }
}
```

### Trigger POS Sync

Manually trigger a sync for a specific POS connection.

```
POST /v1/pos/connections/:id/sync
```

**Request Body:**

```json
{
  "sync_type": "full"
}
```

**Sync Types:**

- `full` - Complete sync of all data
- `incremental` - Sync only changes since last sync
- `sales_only` - Sync only sales data

**Response:**

```json
{
  "data": {
    "sync_id": "sync_xyz789",
    "status": "processing",
    "sync_type": "full",
    "started_at": "2026-01-15T11:00:00Z",
    "estimated_completion": "2026-01-15T11:05:00Z"
  }
}
```

### Get Sync Status

Check the status of a sync operation.

```
GET /v1/pos/syncs/:sync_id
```

**Response:**

```json
{
  "data": {
    "sync_id": "sync_xyz789",
    "status": "completed",
    "sync_type": "full",
    "started_at": "2026-01-15T11:00:00Z",
    "completed_at": "2026-01-15T11:04:30Z",
    "duration_seconds": 270,
    "results": {
      "sales_synced": 234,
      "products_synced": 156,
      "variance_detected": 3
    }
  }
}
```

### List Product Mappings

Retrieve product mappings between POS and inventory.

```
GET /v1/pos/connections/:id/mappings
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mapped` | boolean | No | Filter by mapping status |
| `category` | string | No | Filter by category |

**Response:**

```json
{
  "data": [
    {
      "pos_item_id": "item_toast_123",
      "inventory_item_id": "item_abc123",
      "pos_name": "Tito's Vodka - Rocks",
      "inventory_name": "Tito's Handmade Vodka",
      "conversion_factor": 1.5,
      "last_synced": "2026-01-15T10:30:00Z"
    }
  ]
}
```

### Create Product Mapping

Map a POS item to an inventory item.

```
POST /v1/pos/connections/:id/mappings
```

**Request Body:**

```json
{
  "pos_item_id": "item_toast_123",
  "inventory_item_id": "item_abc123",
  "conversion_factor": 1.5
}
```

**Response:**

```json
{
  "data": {
    "pos_item_id": "item_toast_123",
    "inventory_item_id": "item_abc123",
    "pos_name": "Tito's Vodka - Rocks",
    "inventory_name": "Tito's Handmade Vodka",
    "conversion_factor": 1.5,
    "created_at": "2026-01-15T11:00:00Z"
  }
}
```

### Update Product Mapping

Update an existing product mapping.

```
PUT /v1/pos/connections/:id/mappings/:pos_item_id
```

**Request Body:**

```json
{
  "conversion_factor": 1.75
}
```

**Response:**

```json
{
  "data": {
    "pos_item_id": "item_toast_123",
    "inventory_item_id": "item_abc123",
    "conversion_factor": 1.75,
    "updated_at": "2026-01-15T11:05:00Z"
  }
}
```

### Delete Product Mapping

Delete a product mapping.

```
DELETE /v1/pos/connections/:id/mappings/:pos_item_id
```

**Response:**

```json
{
  "data": {
    "pos_item_id": "item_toast_123",
    "deleted": true
  }
}
```

### Auto-Map Products

Automatically map POS items to inventory items based on name matching.

```
POST /v1/pos/connections/:id/auto-map
```

**Request Body:**

```json
{
  "confidence_threshold": 0.85
}
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

## Data Models

### POS Connection

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique connection identifier |
| `pos_type` | string | POS system type |
| `name` | string | Connection name |
| `status` | string | Connection status (active, inactive, error) |
| `last_sync` | string | ISO 8601 timestamp of last sync |
| `sync_frequency` | string | Sync frequency (realtime, hourly, daily) |
| `sync_stats` | object | Sync statistics |
| `created_at` | string | ISO 8601 timestamp |

### Product Mapping

| Field | Type | Description |
|-------|------|-------------|
| `pos_item_id` | string | POS item identifier |
| `inventory_item_id` | string | Inventory item identifier |
| `pos_name` | string | POS item name |
| `inventory_name` | string | Inventory item name |
| `conversion_factor` | number | Conversion factor between POS and inventory units |
| `last_synced` | string | ISO 8601 timestamp |

## Error Codes

| Code | Description |
|------|-------------|
| `pos_not_supported` | POS type is not supported |
| `invalid_credentials` | Invalid POS credentials |
| `connection_failed` | Failed to connect to POS |
| `sync_in_progress` | Sync already in progress |
| `mapping_not_found` | Product mapping not found |
| `auto_map_failed` | Auto-mapping failed |

## Best Practices

### Use Real-Time Sync for High-Volume Bars

For bars with high sales volume, use real-time sync to detect variance immediately:

```json
{
  "sync_frequency": "realtime"
}
```

### Set Conversion Factors Correctly

Ensure conversion factors accurately reflect the relationship between POS units and inventory units:

```json
{
  "conversion_factor": 1.5
}
```

For example, if a POS "drink" corresponds to 1.5 ounces of inventory, set the conversion factor accordingly.

### Monitor Sync Health

Regularly check sync statistics to ensure data is flowing correctly:

```javascript
const connection = await getPOSConnection(id);
if (connection.sync_stats.failed_syncs > 10) {
  alertTeam('High sync failure rate detected');
}
```

## Examples

### Complete Workflow: Connect and Map

```javascript
// 1. Create POS connection
const connection = await createPOSConnection({
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
const sync = await triggerSync(connection.id, { sync_type: 'full' });

// 3. Wait for sync completion
await waitForSync(sync.sync_id);

// 4. Auto-map products
const mapping = await autoMapProducts(connection.id, {
  confidence_threshold: 0.85
});

console.log(`Created ${mapping.mappings_created} mappings`);
```

## Next Steps

- [Inventory API]({{ site.baseurl }}/api/inventory/) - Manage inventory data
- [Integrations Guide]({{ site.baseurl }}/integrations/pos/) - Step-by-step POS integration
- [Webhooks]({{ site.baseurl }}/api/webhooks/) - Get notified of sync events
