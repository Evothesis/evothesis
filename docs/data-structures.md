# Data Structures and Payload Documentation

Complete reference for SecurePixel event data structures, payload formats, and delivered data schemas.

## Event Data Structure Overview

All SecurePixel events follow a consistent structure with automatic context enrichment:

```json
{
  "eventType": "string",
  "sessionId": "string",
  "visitorId": "string", 
  "siteId": "string",
  "timestamp": "ISO 8601 datetime",
  "eventData": {},
  "page": {},
  "attribution": {},
  "privacy_level": "string"
}
```

## Automatically Collected Context

SecurePixel automatically adds this context to every event:

### Session Information
```json
{
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012",
  "siteId": "example.com",
  "timestamp": "2024-01-15T10:30:45.123Z",
  "created_at": "2024-01-15T10:30:45.123Z"
}
```

### Page Context
```json
{
  "page": {
    "title": "Product Demo - SecurePixel",
    "url": "https://example.com/demo",
    "path": "/demo",
    "referrer": "https://google.com/search"
  }
}
```

### Attribution Data
```json
{
  "attribution": {
    "utm_source": "google",
    "utm_medium": "cpc", 
    "utm_campaign": "product_launch_q4",
    "utm_content": "hero_cta",
    "utm_term": "analytics_platform",
    "referrer": "https://google.com/search"
  }
}
```

## Event-Specific Data Structures

### Page View Events

**Event Type**: `pageview`

**Template Structure**:
```json
{
  "eventType": "pageview",
  "eventData": {
    "page": {
      "title": "{PAGE_TITLE}",
      "url": "{FULL_URL}",
      "path": "{URL_PATH}"
    }
  }
}
```

**Real Example**:
```json
{
  "eventType": "pageview",
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012",
  "siteId": "acme-corp.com",
  "timestamp": "2024-01-15T10:30:45.123Z",
  "eventData": {
    "page": {
      "title": "ACME Corp - Premium Widgets",
      "url": "https://acme-corp.com/products/widgets",
      "path": "/products/widgets"
    }
  },
  "attribution": {
    "utm_source": "facebook",
    "utm_medium": "social",
    "utm_campaign": "widget_awareness",
    "referrer": "https://facebook.com"
  }
}
```

### Click Events

**Event Type**: `click`

**Template Structure**:
```json
{
  "eventType": "click",
  "eventData": {
    "element": {
      "tag": "{HTML_TAG}",
      "id": "{ELEMENT_ID}",
      "class": "{CSS_CLASSES}",
      "text": "{ELEMENT_TEXT}"
    },
    "position": {
      "x": {CLICK_X_COORDINATE},
      "y": {CLICK_Y_COORDINATE}
    }
  }
}
```

**Real Example**:
```json
{
  "eventType": "click",
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012",
  "siteId": "acme-corp.com",
  "timestamp": "2024-01-15T10:31:22.456Z",
  "eventData": {
    "element": {
      "tag": "button",
      "id": "get-started-btn",
      "class": "btn btn-primary cta-button",
      "text": "Get Started Free"
    },
    "position": {
      "x": 650,
      "y": 340
    }
  }
}
```

### Form Events

**Event Type**: `form_focus`, `form_submit`, `lead_form_submit`

**Template Structure**:
```json
{
  "eventType": "form_focus",
  "eventData": {
    "field": {
      "type": "{INPUT_TYPE}",
      "name": "{FIELD_NAME}",
      "id": "{FIELD_ID}"
    },
    "form": {
      "id": "{FORM_ID}",
      "is_lead_form": {BOOLEAN}
    }
  }
}
```

**Real Example - Form Focus**:
```json
{
  "eventType": "form_focus",
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012",
  "timestamp": "2024-01-15T10:32:15.789Z",
  "eventData": {
    "field": {
      "type": "email",
      "name": "email",
      "id": "contact-email"
    },
    "form": {
      "id": "contact-form",
      "is_lead_form": true
    }
  }
}
```

**Real Example - Lead Form Submission**:
```json
{
  "eventType": "lead_form_submit",
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012",
  "timestamp": "2024-01-15T10:33:45.123Z",
  "eventData": {
    "form_id": "contact-form",
    "lead_type": "contact",
    "lead_value": 250,
    "source": "website",
    "campaign": "q4_product_launch",
    "qualified": true
  }  
}
```

### E-commerce Events

**Event Type**: `purchase`

**Template Structure**:
```json
{
  "eventType": "purchase",
  "eventData": {
    "order_id": "{ORDER_ID}",
    "revenue": {TOTAL_AMOUNT},
    "currency": "{CURRENCY_CODE}",
    "products": [
      {
        "sku": "{PRODUCT_SKU}",
        "name": "{PRODUCT_NAME}",
        "price": {UNIT_PRICE},
        "quantity": {QUANTITY},
        "category": "{PRODUCT_CATEGORY}"
      }
    ],
    "shipping": {SHIPPING_COST},
    "tax": {TAX_AMOUNT},
    "customer_type": "{NEW_OR_RETURNING}",
    "coupon_code": "{DISCOUNT_CODE}"
  }
}
```

**Real Example - Purchase**:
```json
{
  "eventType": "purchase", 
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012",
  "timestamp": "2024-01-15T10:45:30.123Z",
  "eventData": {
    "order_id": "ORD-2024-001",
    "revenue": 149.99,
    "currency": "USD",
    "products": [
      {
        "sku": "WIDGET-BLUE-M",
        "name": "Blue Widget Medium",
        "price": 49.99,
        "quantity": 3,
        "category": "Widgets"
      }
    ],
    "shipping": 9.99,
    "tax": 12.50,
    "customer_type": "returning",
    "coupon_code": "SAVE10"
  }
}
```

**Event Type**: `product_view`

**Real Example - Product View**:
```json
{
  "eventType": "product_view",
  "sessionId": "sess_abc123def456", 
  "visitorId": "vis_xyz789uvw012",
  "timestamp": "2024-01-15T10:35:20.456Z",
  "eventData": {
    "sku": "WIDGET-BLUE-M",
    "name": "Blue Widget Medium",
    "price": 49.99,
    "category": "Widgets",
    "brand": "ACME",
    "currency": "USD",
    "in_stock": true
  }
}
```

**Event Type**: `add_to_cart`

**Real Example - Add to Cart**:
```json
{
  "eventType": "add_to_cart",
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012", 
  "timestamp": "2024-01-15T10:36:45.789Z",
  "eventData": {
    "sku": "WIDGET-BLUE-M",
    "name": "Blue Widget Medium",
    "price": 49.99,
    "quantity": 2,
    "currency": "USD",
    "cart_value": 99.98
  }
}
```

### Custom Events

**Event Type**: `{custom_event_name}`

**Template Structure**:
```json
{
  "eventType": "{CUSTOM_EVENT_TYPE}",
  "eventData": {
    "{PROPERTY_NAME}": "{PROPERTY_VALUE}",
    "{NUMERIC_PROPERTY}": {NUMERIC_VALUE},
    "{BOOLEAN_PROPERTY}": {TRUE_OR_FALSE}
  }
}
```

**Real Example - Video Interaction**:
```json
{
  "eventType": "video_play",
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012",
  "timestamp": "2024-01-15T10:37:30.123Z",
  "eventData": {
    "video_title": "Product Demo 2024",
    "video_duration": 120,
    "video_position": "homepage_hero",
    "user_type": "anonymous",
    "autoplay": false
  }
}
```

## Batch Event Structure

SecurePixel automatically batches events for performance. Batch events contain multiple individual events:

**Template Structure**:
```json
{
  "eventType": "batch",
  "sessionId": "{SESSION_ID}",
  "visitorId": "{VISITOR_ID}",
  "siteId": "{SITE_DOMAIN}",
  "timestamp": "{BATCH_TIMESTAMP}",
  "events": [
    {
      "eventType": "{EVENT_TYPE_1}",
      "timestamp": "{EVENT_TIMESTAMP_1}",
      "eventData": {}
    },
    {
      "eventType": "{EVENT_TYPE_2}", 
      "timestamp": "{EVENT_TIMESTAMP_2}",
      "eventData": {}
    }
  ],
  "page": {},
  "attribution": {}
}
```

**Real Example - Event Batch**:
```json
{
  "eventType": "batch",
  "sessionId": "sess_abc123def456",
  "visitorId": "vis_xyz789uvw012",
  "siteId": "acme-corp.com",
  "timestamp": "2024-01-15T10:40:00.000Z",
  "events": [
    {
      "eventType": "click",
      "timestamp": "2024-01-15T10:39:45.123Z",
      "eventData": {
        "element": {
          "tag": "a",
          "id": "nav-products",
          "class": "nav-link",
          "text": "Products"
        }
      }
    },
    {
      "eventType": "pageview", 
      "timestamp": "2024-01-15T10:39:47.456Z",
      "eventData": {
        "page": {
          "title": "Products - ACME Corp",
          "url": "https://acme-corp.com/products",
          "path": "/products"
        }
      }
    }
  ],
  "page": {
    "title": "Products - ACME Corp",
    "url": "https://acme-corp.com/products", 
    "path": "/products",
    "referrer": "https://acme-corp.com"
  }
}
```

## Delivered Data Format

### S3 Export Format

Data is exported to your S3 bucket in JSON Lines format (`.jsonl`):

**File Structure**:
```
your-bucket/
├── raw/
│   └── 2024/01/15/
│       ├── events_2024-01-15_10-00-00.jsonl
│       ├── events_2024-01-15_11-00-00.jsonl
│       └── ...
└── processed/
    └── client-segments/
        ├── acme-corp-2024/
        │   └── 2024/01/15/
        │       ├── acme-corp_events_2024-01-15.jsonl
        │       └── acme-corp_summary_2024-01-15.json
        └── ...
```

**JSONL Content Example**:
```jsonl
{"id":1,"event_id":"550e8400-e29b-41d4-a716-446655440000","event_type":"pageview","session_id":"sess_abc123","visitor_id":"vis_xyz789","site_id":"acme-corp.com","timestamp":"2024-01-15T10:30:45.123Z","url":"https://acme-corp.com/products","path":"/products","user_agent":"Mozilla/5.0...","ip_address":"192.168.1.1","raw_event_data":{"eventType":"pageview","sessionId":"sess_abc123","eventData":{"page":{"title":"Products","url":"https://acme-corp.com/products"}}},"client_id":"acme-corp-2024","created_at":"2024-01-15T10:30:45.123Z"}
{"id":2,"event_id":"550e8400-e29b-41d4-a716-446655440001","event_type":"click","session_id":"sess_abc123","visitor_id":"vis_xyz789","site_id":"acme-corp.com","timestamp":"2024-01-15T10:31:22.456Z","url":"https://acme-corp.com/products","path":"/products","user_agent":"Mozilla/5.0...","ip_address":"192.168.1.1","raw_event_data":{"eventType":"click","sessionId":"sess_abc123","eventData":{"element":{"tag":"button","id":"get-started-btn","text":"Get Started Free"}}},"client_id":"acme-corp-2024","created_at":"2024-01-15T10:31:22.456Z"}
```

### API Response Format

When events are successfully collected, the API returns:

**Success Response**:
```json
{
  "status": "success",
  "events_processed": 5,
  "client_id": "acme-corp-2024", 
  "batch_size": 5
}
```

**Error Response**:
```json
{
  "status": "error",
  "error_code": "VALIDATION_ERROR",
  "message": "Invalid event data format",
  "details": {
    "field": "eventData.revenue",
    "error": "Must be a number"
  }
}
```

## Database Schema

Events are stored in PostgreSQL with this schema:

```sql
CREATE TABLE events_log (
    id SERIAL PRIMARY KEY,
    event_id UUID NOT NULL DEFAULT gen_random_uuid(),
    event_type VARCHAR(50) NOT NULL,
    session_id VARCHAR(100),
    visitor_id VARCHAR(100), 
    site_id VARCHAR(100),
    timestamp TIMESTAMPTZ NOT NULL,
    url TEXT,
    path VARCHAR(500),
    user_agent TEXT,
    ip_address INET,
    raw_event_data JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    processed_at TIMESTAMPTZ,
    raw_exported_at TIMESTAMPTZ,
    client_id VARCHAR(255),
    batch_id UUID,
    export_status VARCHAR(20) DEFAULT 'pending'
);
```

**Indexes for Performance**:
```sql
CREATE INDEX idx_events_timestamp ON events_log(timestamp);
CREATE INDEX idx_events_client_id ON events_log(client_id);
CREATE INDEX idx_events_session_id ON events_log(session_id);
CREATE INDEX idx_events_event_type ON events_log(event_type);
CREATE INDEX idx_events_export_status ON events_log(export_status);
```

## Privacy and Compliance

### GDPR/HIPAA Data Filtering

For clients with enhanced privacy levels, data is automatically filtered:

**Original Event**:
```json
{
  "eventType": "form_submit",
  "eventData": {
    "email": "user@example.com",
    "phone": "555-123-4567", 
    "message": "I'm interested in your product"
  }
}
```

**Filtered Event (GDPR/HIPAA)**:
```json
{
  "eventType": "form_submit",
  "eventData": {
    "message": "I'm interested in your product"
  }
}
```

### IP Address Handling

- **Standard**: Full IP address stored
- **GDPR**: IP address hashed with client-specific salt
- **HIPAA**: IP address hashed with client-specific salt

## Data Validation Rules

### Field Limits
- `eventType`: Maximum 100 characters
- `sessionId`: Maximum 200 characters
- `url`: Maximum 2000 characters
- `eventData`: Maximum 10KB JSON serialized
- Batch size: Maximum 100 events per request

### Required Fields
- `eventType`: Always required
- `timestamp`: Auto-generated if not provided
- `sessionId`: Auto-generated if not provided

### Data Types
- Numeric fields: Must be valid numbers (revenue, prices, quantities)
- Currency codes: Must be 3-letter ISO codes (USD, EUR, GBP)
- Timestamps: Must be valid ISO 8601 format
- Boolean fields: Must be `true` or `false`

## Example Client Integration

### Complete Event Flow
```javascript
// 1. Page load triggers automatic pageview
// (Handled automatically by SecurePixel)

// 2. User clicks product
securepixel.trackProduct({
    sku: 'WIDGET-BLUE-M',
    name: 'Blue Widget Medium', 
    price: 49.99,
    category: 'Widgets'
});

// 3. User adds to cart
securepixel.trackAddToCart({
    sku: 'WIDGET-BLUE-M',
    price: 49.99,
    quantity: 2,
    cart_value: 99.98
});

// 4. User begins checkout
securepixel.trackBeginCheckout({
    cart_value: 99.98,
    currency: 'USD',
    item_count: 2
});

// 5. User completes purchase
securepixel.trackPurchase({
    order_id: 'ORD-2024-001',
    revenue: 109.97, // Including shipping
    currency: 'USD',
    products: [{
        sku: 'WIDGET-BLUE-M',
        name: 'Blue Widget Medium',
        price: 49.99,
        quantity: 2,
        category: 'Widgets'
    }],
    shipping: 9.99
});
```

This complete flow generates trackable revenue attribution from initial page view through purchase completion, with all data structured for analysis and business intelligence integration.