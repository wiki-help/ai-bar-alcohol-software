---
layout: page
title: Ordering API
permalink: /api/ordering/
---

# Ordering API

The Ordering API enables you to automate distributor orders, manage supplier relationships, and track shipments through The Neat Profit platform.

## Endpoints

### List Orders

Retrieve a paginated list of orders.

```
GET /v1/orders
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | Filter by status (draft, submitted, processing, shipped, delivered, cancelled) |
| `distributor_id` | string | No | Filter by distributor |
| `date_from` | string | No | Filter orders from date (ISO 8601) |
| `date_to` | string | No | Filter orders to date (ISO 8601) |
| `page` | integer | No | Page number (default: 1) |
| `per_page` | integer | No | Items per page (default: 50, max: 100) |

**Example Request:**

```bash
curl -X GET "https://api.theneatprofit.com/v1/orders?status=processing" \
  -H "X-API-Key: your-api-key"
```

**Response:**

```json
{
  "data": [
    {
      "id": "order_abc123",
      "distributor_id": "dist_xyz789",
      "distributor_name": "Southern Glazer's",
      "status": "processing",
      "total_amount": 1247.50,
      "estimated_delivery": "2026-01-20T00:00:00Z",
      "created_at": "2026-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 45,
    "page": 1,
    "per_page": 50,
    "total_pages": 1
  }
}
```

### Create Order

Create a new distributor order.

```
POST /v1/orders
```

**Request Body:**

```json
{
  "distributor_id": "dist_xyz789",
  "location": "main_bar",
  "items": [
    {
      "inventory_item_id": "item_abc123",
      "quantity": 12,
      "unit": "case"
    },
    {
      "inventory_item_id": "item_def456",
      "quantity": 6,
      "unit": "case"
    }
  ],
  "notes": "Weekly restock"
}
```

**Response:**

```json
{
  "data": {
    "id": "order_abc123",
    "distributor_id": "dist_xyz789",
    "distributor_name": "Southern Glazer's",
    "status": "draft",
    "location": "main_bar",
    "total_amount": 1247.50,
    "items": [
      {
        "inventory_item_id": "item_abc123",
        "name": "Tito's Handmade Vodka",
        "quantity": 12,
        "unit": "case",
        "unit_price": 89.99,
        "total": 1079.88
      }
    ],
    "notes": "Weekly restock",
    "created_at": "2026-01-15T10:30:00Z"
  }
}
```

### Get Order

Retrieve details for a specific order.

```
GET /v1/orders/:id
```

**Response:**

```json
{
  "data": {
    "id": "order_abc123",
    "distributor_id": "dist_xyz789",
    "distributor_name": "Southern Glazer's",
    "status": "shipped",
    "location": "main_bar",
    "total_amount": 1247.50,
    "estimated_delivery": "2026-01-20T00:00:00Z",
    "tracking_number": "TRK123456789",
    "items": [
      {
        "inventory_item_id": "item_abc123",
        "name": "Tito's Handmade Vodka",
        "quantity": 12,
        "unit": "case",
        "unit_price": 89.99,
        "total": 1079.88
      }
    ],
    "notes": "Weekly restock",
    "created_at": "2026-01-15T10:30:00Z",
    "updated_at": "2026-01-16T14:00:00Z"
  }
}
```

### Update Order

Update an existing order.

```
PUT /v1/orders/:id
```

**Request Body:**

```json
{
  "status": "submitted",
  "items": [
    {
      "inventory_item_id": "item_abc123",
      "quantity": 15,
      "unit": "case"
    }
  ]
}
```

**Response:**

```json
{
  "data": {
    "id": "order_abc123",
    "status": "submitted",
    "items": [
      {
        "inventory_item_id": "item_abc123",
        "quantity": 15,
        "unit": "case"
      }
    ],
    "updated_at": "2026-01-15T11:00:00Z"
  }
}
```

### Delete Order

Cancel or delete an order.

```
DELETE /v1/orders/:id
```

**Response:**

```json
{
  "data": {
    "id": "order_abc123",
    "deleted": true
  }
}
```

### Submit Order

Submit a draft order to the distributor.

```
POST /v1/orders/:id/submit
```

**Response:**

```json
{
  "data": {
    "id": "order_abc123",
    "status": "submitted",
    "submitted_at": "2026-01-15T11:00:00Z",
    "confirmation_number": "ORD-2026-001234"
  }
}
```

### Get Smart Order Recommendations

Get AI-powered order recommendations based on demand forecasting.

```
GET /v1/orders/recommendations
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `distributor_id` | string | No | Filter by distributor |
| `location` | string | No | Filter by location |
| `days_ahead` | integer | No | Forecast horizon in days (default: 7) |

**Response:**

```json
{
  "data": {
    "location": "main_bar",
    "forecast_period": "2026-01-15 to 2026-01-22",
    "recommendations": [
      {
        "inventory_item_id": "item_abc123",
        "name": "Tito's Handmade Vodka",
        "current_stock": 12,
        "forecasted_demand": 18,
        "recommended_order": 12,
        "unit": "case",
        "reason": "High demand forecast for weekend",
        "confidence": 0.87
      }
    ],
    "total_recommended_amount": 1247.50
  }
}
```

### List Distributors

Retrieve available distributors.

```
GET /v1/distributors
```

**Response:**

```json
{
  "data": [
    {
      "id": "dist_xyz789",
      "name": "Southern Glazer's",
      "region": "southeast",
      "supported_categories": ["spirits", "wine", "beer"],
      "min_order_amount": 100.00,
      "lead_time_days": 3
    }
  ]
}
```

### Add Distributor

Add a new distributor to your account.

```
POST /v1/distributors
```

**Request Body:**

```json
{
  "name": "Southern Glazer's",
  "distributor_code": "SG",
  "region": "southeast",
  "contact_email": "orders@southernglazers.com",
  "min_order_amount": 100.00,
  "lead_time_days": 3
}
```

**Response:**

```json
{
  "data": {
    "id": "dist_xyz789",
    "name": "Southern Glazer's",
    "distributor_code": "SG",
    "region": "southeast",
    "contact_email": "orders@southernglazers.com",
    "min_order_amount": 100.00,
    "lead_time_days": 3,
    "created_at": "2026-01-15T10:30:00Z"
  }
}
```

## Data Models

### Order

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique order identifier |
| `distributor_id` | string | Distributor identifier |
| `distributor_name` | string | Distributor name |
| `status` | string | Order status |
| `location` | string | Delivery location |
| `total_amount` | number | Total order amount |
| `estimated_delivery` | string | ISO 8601 timestamp |
| `tracking_number` | string | Shipment tracking number |
| `items` | array | Order items |
| `notes` | string | Order notes |
| `created_at` | string | ISO 8601 timestamp |
| `updated_at` | string | ISO 8601 timestamp |

### Order Item

| Field | Type | Description |
|-------|------|-------------|
| `inventory_item_id` | string | Inventory item identifier |
| `name` | string | Item name |
| `quantity` | number | Quantity ordered |
| `unit` | string | Unit of measurement |
| `unit_price` | number | Price per unit |
| `total` | number | Line item total |

### Distributor

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique distributor identifier |
| `name` | string | Distributor name |
| `distributor_code` | string | Distributor code |
| `region` | string | Geographic region |
| `contact_email` | string | Contact email |
| `min_order_amount` | number | Minimum order amount |
| `lead_time_days` | integer | Standard lead time in days |

## Error Codes

| Code | Description |
|------|-------------|
| `order_not_found` | Order not found |
| `invalid_status` | Invalid order status transition |
| `distributor_not_found` | Distributor not found |
| `below_min_order` | Order below minimum amount |
| `item_not_available` | Item not available from distributor |
| `insufficient_stock` | Insufficient stock for order |

## Best Practices

### Use Smart Order Recommendations

Leverage AI-powered recommendations to optimize ordering:

```javascript
const recommendations = await getOrderRecommendations({
  location: 'main_bar',
  days_ahead: 7
});

// Create order from recommendations
const order = await createOrder({
  distributor_id: recommendations.distributor_id,
  items: recommendations.recommendations.map(r => ({
    inventory_item_id: r.inventory_item_id,
    quantity: r.recommended_order,
    unit: r.unit
  }))
});
```

### Check Minimum Order Requirements

Verify orders meet distributor minimums before submitting:

```javascript
const order = await createOrder(orderData);
const distributor = await getDistributor(order.distributor_id);

if (order.total_amount < distributor.min_order_amount) {
  // Add items to meet minimum
  await addFillerItems(order.id);
}
```

### Track Order Status

Monitor order status to anticipate deliveries:

```javascript
const order = await getOrder(orderId);
if (order.status === 'shipped') {
  scheduleInventoryUpdate(order.estimated_delivery);
}
```

## Examples

### Complete Workflow: Smart Ordering

```javascript
// 1. Get recommendations
const recommendations = await getOrderRecommendations({
  location: 'main_bar',
  days_ahead: 7
});

// 2. Create order from recommendations
const order = await createOrder({
  distributor_id: 'dist_xyz789',
  location: 'main_bar',
  items: recommendations.recommendations.map(r => ({
    inventory_item_id: r.inventory_item_id,
    quantity: r.recommended_order,
    unit: r.unit
  }))
});

// 3. Review and submit
await submitOrder(order.id);

console.log(`Order ${order.id} submitted successfully`);
```

## Next Steps

- [Inventory API]({{ site.baseurl }}/api/inventory/) - Manage inventory data
- [Analytics API]({{ site.baseurl }}/api/analytics/) - Access demand forecasting
- [Integrations Guide]({{ site.baseurl }}/integrations/distributors/) - Distributor integration
