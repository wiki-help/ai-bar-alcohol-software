---
layout: page
title: Inventory API
permalink: /api/inventory/
---

# Inventory API

The Inventory API provides endpoints for managing your bar's inventory, including products, stock levels, and inventory counts.

## Endpoints

### List Inventory Items

Retrieve a paginated list of inventory items.

```
GET /v1/inventory/items
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `location` | string | No | Filter by location (e.g., "main_bar", "restaurant") |
| `category` | string | No | Filter by category (e.g., "spirits", "beer", "wine") |
| `low_stock` | boolean | No | Only show items below reorder point |
| `page` | integer | No | Page number (default: 1) |
| `per_page` | integer | No | Items per page (default: 50, max: 100) |
| `sort` | string | No | Sort field (e.g., "name", "quantity", "last_updated") |
| `order` | string | No | Sort order ("asc" or "desc", default: "asc") |

**Example Request:**

```bash
curl -X GET "https://api.theneatprofit.com/v1/inventory/items?location=main_bar&low_stock=true" \
  -H "X-API-Key: your-api-key"
```

**Response:**

```json
{
  "data": [
    {
      "id": "item_abc123",
      "name": "Tito's Handmade Vodka",
      "sku": "SKU-001",
      "category": "spirits",
      "quantity": 2.5,
      "unit": "bottle",
      "size_ml": 750,
      "location": "main_bar",
      "reorder_point": 6,
      "cost_per_unit": 24.99,
      "last_updated": "2026-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 15,
    "page": 1,
    "per_page": 50,
    "total_pages": 1
  }
}
```

### Create Inventory Item

Create a new inventory item.

```
POST /v1/inventory/items
```

**Request Body:**

```json
{
  "name": "Tito's Handmade Vodka",
  "sku": "SKU-001",
  "category": "spirits",
  "quantity": 12,
  "unit": "bottle",
  "size_ml": 750,
  "location": "main_bar",
  "reorder_point": 6,
  "cost_per_unit": 24.99
}
```

**Response:**

```json
{
  "data": {
    "id": "item_abc123",
    "name": "Tito's Handmade Vodka",
    "sku": "SKU-001",
    "category": "spirits",
    "quantity": 12,
    "unit": "bottle",
    "size_ml": 750,
    "location": "main_bar",
    "reorder_point": 6,
    "cost_per_unit": 24.99,
    "created_at": "2026-01-15T10:30:00Z",
    "last_updated": "2026-01-15T10:30:00Z"
  }
}
```

### Get Inventory Item

Retrieve details for a specific inventory item.

```
GET /v1/inventory/items/:id
```

**Response:**

```json
{
  "data": {
    "id": "item_abc123",
    "name": "Tito's Handmade Vodka",
    "sku": "SKU-001",
    "category": "spirits",
    "quantity": 12,
    "unit": "bottle",
    "size_ml": 750,
    "location": "main_bar",
    "reorder_point": 6,
    "cost_per_unit": 24.99,
    "created_at": "2026-01-15T10:30:00Z",
    "last_updated": "2026-01-15T10:30:00Z"
  }
}
```

### Update Inventory Item

Update an existing inventory item.

```
PUT /v1/inventory/items/:id
```

**Request Body:**

```json
{
  "quantity": 10,
  "reorder_point": 8,
  "cost_per_unit": 26.99
}
```

**Response:**

```json
{
  "data": {
    "id": "item_abc123",
    "name": "Tito's Handmade Vodka",
    "sku": "SKU-001",
    "category": "spirits",
    "quantity": 10,
    "unit": "bottle",
    "size_ml": 750,
    "location": "main_bar",
    "reorder_point": 8,
    "cost_per_unit": 26.99,
    "created_at": "2026-01-15T10:30:00Z",
    "last_updated": "2026-01-15T11:00:00Z"
  }
}
```

### Delete Inventory Item

Delete an inventory item.

```
DELETE /v1/inventory/items/:id
```

**Response:**

```json
{
  "data": {
    "id": "item_abc123",
    "deleted": true
  }
}
```

### Submit Inventory Count

Submit an inventory count to update stock levels.

```
POST /v1/inventory/count
```

**Request Body:**

```json
{
  "location": "main_bar",
  "counted_by": "user_123",
  "items": [
    {
      "item_id": "item_abc123",
      "quantity": 10.5,
      "notes": "Partial bottle"
    },
    {
      "item_id": "item_def456",
      "quantity": 24,
      "notes": null
    }
  ]
}
```

**Response:**

```json
{
  "data": {
    "count_id": "count_xyz789",
    "location": "main_bar",
    "counted_by": "user_123",
    "items_updated": 2,
    "variance_detected": [
      {
        "item_id": "item_abc123",
        "expected": 12,
        "actual": 10.5,
        "variance": -1.5
      }
    ],
    "created_at": "2026-01-15T11:00:00Z"
  }
}
```

### Batch Update Inventory

Update multiple inventory items in a single request.

```
POST /v1/inventory/batch
```

**Request Body:**

```json
{
  "updates": [
    {
      "item_id": "item_abc123",
      "quantity": 10
    },
    {
      "item_id": "item_def456",
      "quantity": 24
    }
  ]
}
```

**Response:**

```json
{
  "data": {
    "batch_id": "batch_xyz789",
    "items_updated": 2,
    "failed": []
  }
}
```

## Data Models

### Inventory Item

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique item identifier |
| `name` | string | Product name |
| `sku` | string | Stock keeping unit |
| `category` | string | Product category |
| `quantity` | number | Current quantity |
| `unit` | string | Unit of measurement (bottle, case, liter) |
| `size_ml` | integer | Size in milliliters |
| `location` | string | Storage location |
| `reorder_point` | number | Quantity threshold for reordering |
| `cost_per_unit` | number | Cost per unit |
| `created_at` | string | ISO 8601 timestamp |
| `last_updated` | string | ISO 8601 timestamp |

## Error Codes

| Code | Description |
|------|-------------|
| `item_not_found` | Inventory item not found |
| `invalid_quantity` | Quantity must be a positive number |
| `invalid_unit` | Invalid unit of measurement |
| `duplicate_sku` | SKU already exists |
| `location_not_found` | Specified location does not exist |

## Best Practices

### Use Batch Operations

When updating multiple items, use the batch endpoint instead of individual requests:

```javascript
// Inefficient
for (const item of items) {
  await updateItem(item.id, item.quantity);
}

// Efficient
await batchUpdate(items);
```

### Handle Variance

When submitting inventory counts, check for variance in the response:

```javascript
const response = await submitCount(countData);
if (response.data.variance_detected.length > 0) {
  // Alert staff to investigate variance
  alertVariance(response.data.variance_detected);
}
```

### Set Reorder Points

Configure reorder points to enable low stock alerts:

```json
{
  "reorder_point": 6
}
```

## Examples

### Complete Workflow: Create and Count

```javascript
// 1. Create item
const item = await createItem({
  name: "Tito's Handmade Vodka",
  sku: "SKU-001",
  category: "spirits",
  quantity: 12,
  unit: "bottle",
  location: "main_bar",
  reorder_point: 6
});

// 2. Submit count
const count = await submitCount({
  location: "main_bar",
  items: [
    {
      item_id: item.id,
      quantity: 10.5
    }
  ]
});

// 3. Check for variance
if (count.variance_detected.length > 0) {
  console.log("Variance detected:", count.variance_detected);
}
```

## Next Steps

- [POS Integration API]({{ site.baseurl }}/api/pos/) - Sync inventory with POS data
- [Ordering API]({{ site.baseurl }}/api/ordering/) - Automate reordering
- [Webhooks]({{ site.baseurl }}/api/webhooks/) - Get notified of inventory changes
