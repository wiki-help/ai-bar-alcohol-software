---
layout: page
title: Analytics API
permalink: /api/analytics/
---

# Analytics API

The Analytics API provides access to variance reports, demand forecasts, performance metrics, and recipe costing data to help you make data-driven decisions.

## Endpoints

### Get Variance Report

Retrieve variance analysis for a specific time period.

```
GET /v1/analytics/variance
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date_from` | string | No | Start date (ISO 8601, default: 30 days ago) |
| `date_to` | string | No | End date (ISO 8601, default: today) |
| `location` | string | No | Filter by location |
| `category` | string | No | Filter by category |
| `threshold` | number | No | Variance threshold percentage (default: 5) |

**Example Request:**

```bash
curl -X GET "https://api.theneatprofit.com/v1/analytics/variance?date_from=2026-01-01&date_to=2026-01-15" \
  -H "X-API-Key: your-api-key"
```

**Response:**

```json
{
  "data": {
    "period": "2026-01-01 to 2026-01-15",
    "total_variance": 1247.50,
    "variance_percentage": 8.5,
    "items": [
      {
        "inventory_item_id": "item_abc123",
        "name": "Tito's Handmade Vodka",
        "expected_consumption": 24,
        "actual_consumption": 28.5,
        "variance": 4.5,
        "variance_percentage": 18.75,
        "estimated_loss": 112.50,
        "potential_causes": ["over-pouring", "theft"]
      }
    ],
    "summary": {
      "total_items_analyzed": 156,
      "items_above_threshold": 12,
      "high_variance_items": 3
    }
  }
}
```

### Get Demand Forecast

Retrieve AI-powered demand forecast for inventory items.

```
GET /v1/analytics/forecast
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `days_ahead` | integer | No | Forecast horizon in days (default: 7, max: 30) |
| `location` | string | No | Filter by location |
| `category` | string | No | Filter by category |

**Response:**

```json
{
  "data": {
    "forecast_period": "2026-01-15 to 2026-01-22",
    "location": "main_bar",
    "forecasts": [
      {
        "inventory_item_id": "item_abc123",
        "name": "Tito's Handmade Vodka",
        "current_stock": 12,
        "forecasted_demand": 18,
        "confidence": 0.87,
        "factors": {
          "seasonality": "weekend_peak",
          "events": ["local_concert"],
          "trend": "increasing"
        },
        "recommendation": "order_6_cases"
      }
    ],
    "generated_at": "2026-01-15T10:30:00Z"
  }
}
```

### Get Performance Metrics

Retrieve key performance indicators for your venue.

```
GET /v1/analytics/performance
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date_from` | string | No | Start date (ISO 8601, default: 30 days ago) |
| `date_to` | string | No | End date (ISO 8601, default: today) |
| `location` | string | No | Filter by location |

**Response:**

```json
{
  "data": {
    "period": "2026-01-01 to 2026-01-15",
    "location": "main_bar",
    "metrics": {
      "pour_cost": {
        "value": 0.22,
        "target": 0.20,
        "status": "above_target"
      },
      "variance_rate": {
        "value": 0.085,
        "target": 0.05,
        "status": "above_target"
      },
      "inventory_turnover": {
        "value": 4.2,
        "target": 4.0,
        "status": "on_target"
      },
      "gross_margin": {
        "value": 0.78,
        "target": 0.75,
        "status": "above_target"
      }
    },
    "trends": {
      "pour_cost": "stable",
      "variance_rate": "increasing",
      "inventory_turnover": "stable",
      "gross_margin": "increasing"
    }
  }
}
```

### Get Recipe Costing Data

Retrieve detailed recipe costing information.

```
GET /v1/analytics/costing
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `recipe_id` | string | No | Specific recipe ID |
| `category` | string | No | Filter by drink category |

**Response:**

```json
{
  "data": [
    {
      "recipe_id": "recipe_abc123",
      "name": "Vodka Tonic",
      "category": "cocktails",
      "selling_price": 12.00,
      "total_cost": 2.65,
      "pour_cost_percentage": 22.08,
      "gross_margin": 77.92,
      "ingredients": [
        {
          "inventory_item_id": "item_abc123",
          "name": "Tito's Handmade Vodka",
          "quantity": 1.5,
          "unit": "oz",
          "cost": 1.50
        },
        {
          "inventory_item_id": "item_def456",
          "name": "Tonic Water",
          "quantity": 4,
          "unit": "oz",
          "cost": 0.35
        }
      ],
      "optimization_suggestions": [
        "Consider premium vodka for higher margin",
        "Adjust price to target 20% pour cost"
      ]
    }
  ]
}
```

### Get Staff Performance

Retrieve performance metrics by staff member.

```
GET /v1/analytics/staff
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date_from` | string | No | Start date (ISO 8601) |
| `date_to` | string | No | End date (ISO 8601) |
| `staff_id` | string | No | Specific staff member ID |

**Response:**

```json
{
  "data": [
    {
      "staff_id": "staff_abc123",
      "name": "John Smith",
      "role": "bartender",
      "metrics": {
        "variance_rate": 0.03,
        "drinks_served": 1247,
        "average_ticket": 45.50,
        "customer_satisfaction": 4.7
      },
      "comparison": {
        "variance_rate_vs_avg": -0.055,
        "drinks_served_vs_avg": 234,
        "performance_rating": "excellent"
      }
    }
  ]
}
```

### Get Category Analysis

Retrieve analysis by product category.

```
GET /v1/analytics/categories
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date_from` | string | No | Start date (ISO 8601) |
| `date_to` | string | No | End date (ISO 8601) |

**Response:**

```json
{
  "data": [
    {
      "category": "spirits",
      "total_sales": 15470.00,
      "total_cost": 3403.40,
      "pour_cost": 0.22,
      "variance": 450.00,
      "variance_percentage": 0.13,
      "top_performers": [
        {
          "name": "Tito's Handmade Vodka",
          "sales": 3240.00,
          "margin": 0.78
        }
      ],
      "underperformers": [
        {
          "name": "Premium Gin",
          "sales": 890.00,
          "variance": 0.25
        }
      ]
    }
  ]
}
```

## Data Models

### Variance Report

| Field | Type | Description |
|-------|------|-------------|
| `period` | string | Analysis period |
| `total_variance` | number | Total variance in dollars |
| `variance_percentage` | number | Overall variance percentage |
| `items` | array | Individual item variance data |
| `summary` | object | Summary statistics |

### Demand Forecast

| Field | Type | Description |
|-------|------|-------------|
| `forecast_period` | string | Forecast period |
| `location` | string | Location identifier |
| `forecasts` | array | Individual item forecasts |
| `generated_at` | string | ISO 8601 timestamp |

### Performance Metrics

| Field | Type | Description |
|-------|------|-------------|
| `period` | string | Analysis period |
| `location` | string | Location identifier |
| `metrics` | object | KPI values and targets |
| `trends` | object | Trend directions |

## Error Codes

| Code | Description |
|------|-------------|
| `invalid_date_range` | Invalid date range specified |
| `insufficient_data` | Insufficient data for analysis |
| `forecast_unavailable` | Forecast not available for specified parameters |
| `staff_not_found` | Staff member not found |

## Best Practices

### Monitor Variance Regularly

Check variance reports weekly to identify issues early:

```javascript
const variance = await getVarianceReport({
  date_from: getStartDate(7),
  date_to: new Date().toISOString()
});

if (variance.data.variance_percentage > 0.10) {
  alertTeam('High variance detected');
}
```

### Use Forecasts for Ordering

Integrate demand forecasts into your ordering workflow:

```javascript
const forecast = await getDemandForecast({ days_ahead: 7 });
const recommendations = forecast.data.forecasts.filter(f => 
  f.forecasted_demand > f.current_stock
);

// Create orders based on forecast
await createOrdersFromRecommendations(recommendations);
```

### Track Performance Trends

Monitor KPI trends over time to identify improvement areas:

```javascript
const current = await getPerformanceMetrics();
const previous = await getPerformanceMetrics({
  date_from: getStartDate(30),
  date_to: getStartDate(7)
});

compareTrends(current.data.metrics, previous.data.metrics);
```

## Examples

### Complete Workflow: Variance Analysis

```javascript
// 1. Get variance report
const variance = await getVarianceReport({
  date_from: '2026-01-01',
  date_to: '2026-01-15'
});

// 2. Identify high variance items
const highVariance = variance.data.items.filter(item => 
  item.variance_percentage > 15
);

// 3. Get staff performance for investigation
const staff = await getStaffPerformance({
  date_from: '2026-01-01',
  date_to: '2026-01-15'
});

// 4. Correlate variance with staff
const analysis = correlateVarianceWithStaff(highVariance, staff);

// 5. Generate recommendations
const recommendations = generateVarianceRecommendations(analysis);
```

## Next Steps

- [Inventory API]({{ site.baseurl }}/api/inventory/) - Manage inventory data
- [Ordering API]({{ site.baseurl }}/api/ordering/) - Automate reordering based on forecasts
- [Webhooks]({{ site.baseurl }}/api/webhooks/) - Get notified of variance alerts
