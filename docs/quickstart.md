# SecurePixel Quickstart Guide

Get SecurePixel tracking running on your website in under 5 minutes.

## Step 1: Add the Tracking Script

### Template
Add this script tag to your website's `<head>` section:

```html
<script src="https://pixel-management-275731808857.us-central1.run.app/pixel/{YOUR_CLIENT_ID}/tracking.js" async></script>
```

### Real Example
```html
<script src="https://pixel-management-275731808857.us-central1.run.app/pixel/acme-corp-2024/tracking.js" async></script>
```

> **Note**: Replace `{YOUR_CLIENT_ID}` with your actual client ID from the SecurePixel dashboard.

## Step 2: Verify Installation

Open your browser's developer console and look for these messages:

```
[SecurePixel] Analytics pixel loaded for client: your-client-id
[SecurePixel] DataLayer integration active
```

## Step 3: Track Your First Event

### Template - Manual Event Tracking
```javascript
// Wait for SecurePixel to load, then track custom events
securepixel.track('{EVENT_TYPE}', {
    {PROPERTY_NAME}: '{PROPERTY_VALUE}',
    {NUMERIC_PROPERTY}: {NUMERIC_VALUE}
});
```

### Real Example - Button Click Tracking
```javascript
// Track newsletter signup button clicks
document.getElementById('newsletter-signup').addEventListener('click', function() {
    securepixel.track('newsletter_signup_click', {
        button_location: 'header',
        page_url: window.location.href,
        user_logged_in: false
    });
});
```

## Step 4: E-commerce Tracking (Optional)

### Template - Purchase Tracking
```javascript
securepixel.trackPurchase({
    order_id: '{ORDER_ID}',
    revenue: {TOTAL_AMOUNT},
    currency: '{CURRENCY_CODE}',
    products: [
        {
            sku: '{PRODUCT_SKU}',
            name: '{PRODUCT_NAME}',
            price: {UNIT_PRICE},
            quantity: {QUANTITY}
        }
    ]
});
```

### Real Example - E-commerce Purchase
```javascript
securepixel.trackPurchase({
    order_id: 'ORD-2024-001',
    revenue: 149.99,
    currency: 'USD',
    products: [
        {
            sku: 'WIDGET-BLUE-M',
            name: 'Blue Widget Medium',
            price: 49.99,
            quantity: 3
        }
    ]
});
```

## Step 5: Lead Form Tracking (Optional)

### Template - Lead Form Setup
```html
<form id="{FORM_ID}" data-securepixel-lead="true" data-lead-type="{LEAD_TYPE}">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <button type="submit">Submit</button>
</form>
```

### Real Example - Contact Form
```html
<form id="contact-form" data-securepixel-lead="true" data-lead-type="contact">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <textarea name="message" required></textarea>
    <button type="submit">Send Message</button>
</form>
```

## What Happens Next?

1. **Automatic Tracking**: SecurePixel immediately starts collecting:
   - Page views
   - Click events
   - Scroll depth
   - Session data

2. **Data Processing**: Your data flows through:
   - Real-time collection
   - Privacy filtering (GDPR/HIPAA compliant)
   - S3 export pipeline
   - Your business intelligence tools

3. **Data Access**: Access your data via:
   - Direct S3 bucket access
   - API endpoints
   - Scheduled exports
   - Real-time webhooks

## Need More Advanced Setup?

- **E-commerce Sites**: See [E-commerce Tracking Guide](ecommerce-tracking.md)
- **Lead Generation**: See [Form Tracking Guide](form-tracking.md)
- **Platform Integration**: See [Platform Integration Guide](platform-integrations.md)
- **Custom Events**: See [Engagement Tracking Guide](engagement-tracking.md)

## Troubleshooting

### Script Not Loading?
- Verify your client ID is correct
- Check that your domain is authorized in SecurePixel dashboard
- Ensure no ad blockers are interfering

### Events Not Appearing?
- Check browser console for error messages
- Enable debug mode: `securepixel.debug(true)`
- Verify network requests in DevTools

### Questions?
Contact our support team with your client ID and specific error messages for fastest resolution.