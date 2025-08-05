# Troubleshooting Guide

Common issues, debugging techniques, and solutions for SecurePixel implementation problems.

## Installation Issues

### Script Not Loading

**Problem**: SecurePixel script fails to load or `securepixel` is undefined.

**Template - Check Script Loading**:
```javascript
// Check if SecurePixel loaded correctly
if (typeof window.securepixel === 'undefined') {
    console.error('SecurePixel failed to load');
    console.log('Check: Client ID, domain authorization, network connectivity');
} else {
    console.log('SecurePixel loaded successfully');
    console.log('Client ID:', securepixel.getStats().client_id);
}
```

**Real Example - Script Loading Verification**:
```javascript
// Complete script loading diagnostic
document.addEventListener('DOMContentLoaded', function() {
    setTimeout(function() {
        if (typeof window.securepixel === 'undefined') {
            console.error('❌ SecurePixel failed to load');
            
            // Check common issues
            console.log('🔍 Troubleshooting checklist:');
            console.log('1. Verify client ID in script URL');
            console.log('2. Check domain is authorized in SecurePixel dashboard');
            console.log('3. Disable ad blockers and try again');
            console.log('4. Check network connectivity');
            console.log('5. Verify script URL is accessible:', 
                       'https://pixel-management-275731808857.us-central1.run.app/pixel/YOUR_CLIENT_ID/tracking.js');
            
            // Try to load manually for debugging
            const script = document.createElement('script');
            script.src = 'https://pixel-management-275731808857.us-central1.run.app/pixel/acme-corp-2024/tracking.js';
            script.onload = () => console.log('✅ Manual script load successful');
            script.onerror = () => console.error('❌ Manual script load failed');
            document.head.appendChild(script);
            
        } else {
            console.log('✅ SecurePixel loaded successfully');
            console.log('Stats:', securepixel.getStats());
        }
    }, 2000); // Wait 2 seconds for script to load
});
```

**Common Solutions**:
- Verify client ID is correct in script URL
- Check domain authorization in SecurePixel dashboard
- Disable ad blockers temporarily
- Ensure script loads before other tracking code

### Network Connectivity Issues (NEW)

**Problem**: Intermittent tracking failures due to unstable network (aircraft WiFi, mobile networks).

**Template - Check Network Resilience**:
```javascript
// Monitor network resilience features
securepixel.debug(true);

// Send test event and check retry behavior
securepixel.track('network_resilience_test', {
    test: true,
    timestamp: new Date().toISOString(),
    connection_type: navigator.connection ? navigator.connection.effectiveType : 'unknown'
});

// Check console for retry messages
console.log('Look for retry messages: "Network error for domain", "Retrying in"');
```

**Real Example - Network Resilience Diagnostic**:
```javascript
// Test network resilience and retry logic
function testNetworkResilience() {
    console.log('🔍 Testing Network Resilience...');
    
    // Enable debug to see retry attempts
    securepixel.debug(true);
    
    // Monitor for retry-related log messages
    const originalLog = console.log;
    const networkMessages = [];
    
    console.log = function(...args) {
        const message = args.join(' ');
        if (message.includes('Network error') || 
            message.includes('Retrying in') || 
            message.includes('Failed to get client config') ||
            message.includes('attempt')) {
            networkMessages.push({
                timestamp: new Date().toISOString(),
                message: message
            });
        }
        return originalLog.apply(console, args);
    };
    
    // Send multiple test events to trigger potential retries
    for (let i = 0; i < 5; i++) {
        setTimeout(() => {
            securepixel.track('network_test_' + i, {
                test_sequence: i,
                timestamp: new Date().toISOString(),
                network_info: {
                    online: navigator.onLine,
                    connection: navigator.connection ? {
                        type: navigator.connection.effectiveType,
                        downlink: navigator.connection.downlink
                    } : null
                }
            });
        }, i * 1000);
    }
    
    // Check results after 30 seconds
    setTimeout(() => {
        console.log('📊 Network Resilience Test Results:');
        console.log('Network-related messages:', networkMessages);
        
        if (networkMessages.length === 0) {
            console.log('✅ No network issues detected - good connectivity');
        } else {
            console.log('⚠️ Network issues detected - retry logic should be working');
            console.log('Expected behavior: Up to 3 retry attempts with exponential backoff');
        }
    }, 30000);
}

// Run network resilience test
testNetworkResilience();
```

**Common Solutions**:
- Network resilience is automatic - no configuration needed
- System retries up to 3 times with exponential backoff (1s, 2s, 4s)
- Events may show `error_domain` client IDs during network failures (this is normal)
- Stable connectivity will automatically resolve to correct client IDs

### Domain Authorization Errors

**Problem**: 403 Forbidden errors or "Domain not authorized" messages.

**Template - Debug Domain Authorization**:
```javascript
// Check current domain and authorization
console.log('Current domain:', window.location.hostname);
console.log('Current URL:', window.location.href);

// Test with manual event
securepixel.track('domain_test', {
    domain: window.location.hostname,
    test_timestamp: new Date().toISOString()
});
```

**Real Example - Domain Authorization Debugging**:
```javascript
// Complete domain authorization diagnostic
function debugDomainAuth() {
    const currentDomain = window.location.hostname;
    const currentURL = window.location.href;
    
    console.log('🔍 Domain Authorization Debug Info:');
    console.log('Current domain:', currentDomain);
    console.log('Current URL:', currentURL);
    console.log('Client ID:', securepixel.getStats().client_id);
    
    // Test different domain variations
    const domainVariations = [
        currentDomain,
        currentDomain.replace('www.', ''),
        'www.' + currentDomain.replace('www.', ''),
        currentDomain.split('.').slice(-2).join('.') // Root domain
    ];
    
    console.log('Domain variations to check in dashboard:', domainVariations);
    
    // Send test event to check authorization
    securepixel.track('authorization_test', {
        domain: currentDomain,
        variations_tested: domainVariations,
        test_time: new Date().toISOString(),
        user_agent: navigator.userAgent.substring(0, 100)
    });
    
    console.log('✅ Test event sent - check network tab for 403 errors');
}

// Run diagnostic if SecurePixel is loaded
if (typeof securepixel !== 'undefined') {
    debugDomainAuth();
}
```

**Common Solutions**:
- Add exact domain (including www/non-www) to SecurePixel dashboard
- Check for typos in domain configuration
- Add both www and non-www versions of domain
- For subdomains, ensure parent domain is authorized

## Event Tracking Issues

### Events Not Firing

**Problem**: Events appear to work but don't show up in data collection.

**Template - Debug Event Firing**:
```javascript
// Enable debug mode and test events
securepixel.debug(true);

// Test basic event
securepixel.track('{TEST_EVENT_TYPE}', {
    test: true,
    timestamp: new Date().toISOString(),
    debug_info: '{DEBUG_CONTEXT}'
});

// Check event queue
console.log('Events queued:', securepixel.getStats().events_queued);
```

**Real Example - Event Debugging Workflow**:
```javascript
// Complete event debugging workflow
function debugEventTracking() {
    console.log('🔍 Starting Event Tracking Debug...');
    
    // 1. Enable debug mode
    securepixel.debug(true);
    
    // 2. Check SecurePixel status
    const stats = securepixel.getStats();
    console.log('SecurePixel Stats:', stats);
    
    // 3. Test simple event
    console.log('📤 Sending test event...');
    securepixel.track('debug_test_event', {
        test: true,
        timestamp: new Date().toISOString(),
        page_url: window.location.href,
        user_agent: navigator.userAgent.substring(0, 50)
    });
    
    // 4. Test e-commerce event
    console.log('📤 Sending test purchase event...');
    securepixel.trackPurchase({
        order_id: 'DEBUG-TEST-' + Date.now(),
        revenue: 99.99,
        currency: 'USD',
        products: [{
            sku: 'DEBUG-SKU',
            name: 'Debug Test Product',
            price: 99.99,
            quantity: 1
        }]
    });
    
    // 5. Test form event
    console.log('📤 Sending test lead event...');
    securepixel.trackLead({
        form_id: 'debug-test-form',
        lead_type: 'debug',
        lead_value: 100,
        source: 'troubleshooting'
    });
    
    // 6. Check network requests
    setTimeout(() => {
        console.log('✅ Debug events sent. Check:');
        console.log('1. Network tab for /collect requests');
        console.log('2. Request payloads contain expected data');
        console.log('3. Response status codes (should be 200)');
        console.log('4. Events queued:', securepixel.getStats().events_queued);
    }, 1000);
}

// Run debug if SecurePixel is available
if (typeof securepixel !== 'undefined') {
    debugEventTracking();
} else {
    console.error('❌ SecurePixel not loaded - fix loading issues first');
}
```

**Common Solutions**:
- Check browser network tab for failed requests
- Verify event data doesn't exceed size limits
- Ensure required fields are provided
- Check for JavaScript errors preventing execution

### Form Tracking Not Working

**Problem**: Lead forms not automatically detected or tracked.

**Template - Debug Form Detection**:
```html
<form id="{FORM_ID}" data-securepixel-lead="true">
    <!-- form fields -->
</form>

<script>
// Check if form is detected
const form = document.getElementById('{FORM_ID}');
console.log('Form detected:', form);
console.log('Has lead attribute:', form.dataset.securepixelLead);
console.log('Form ID contains lead keywords:', /lead|contact|demo|trial/.test(form.id));
</script>
```

**Real Example - Form Detection Debugging**:
```html
<!-- Test form with debugging -->
<form id="contact-form" data-securepixel-lead="true" data-lead-type="contact">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <textarea name="message" required></textarea>
    <button type="submit">Send Message</button>
</form>

<script>
function debugFormDetection() {
    console.log('🔍 Form Detection Debug...');
    
    // Check all forms on page
    const allForms = document.querySelectorAll('form');
    console.log('Total forms found:', allForms.length);
    
    allForms.forEach((form, index) => {
        console.log(`Form ${index + 1}:`, {
            id: form.id,
            classes: form.className,
            hasLeadAttribute: form.dataset.securepixelLead === 'true',
            leadType: form.dataset.leadType,
            isLeadForm: isLeadForm(form)
        });
    });
    
    // Test specific form
    const testForm = document.getElementById('contact-form');
    if (testForm) {
        console.log('✅ Test form found:', testForm);
        
        // Add manual event listener to test
        testForm.addEventListener('submit', function(e) {
            e.preventDefault();
            console.log('📤 Form submitted - manual tracking');
            
            securepixel.trackLead({
                form_id: this.id,
                lead_type: this.dataset.leadType || 'contact',
                lead_value: 250,
                source: 'manual_debug',
                form_fields: Array.from(this.elements).map(el => el.name).filter(Boolean)
            });
        });
        
        console.log('✅ Manual form listener added');
    } else {
        console.error('❌ Test form not found');
    }
}

function isLeadForm(form) {
    const indicators = ['lead', 'contact', 'quote', 'demo', 'trial', 'signup', 'subscribe'];
    const formId = (form.id || '').toLowerCase();
    const formClass = (form.className || '').toLowerCase();
    const formAction = (form.action || '').toLowerCase();
    
    return form.dataset.securepixelLead === 'true' || 
           indicators.some(indicator => 
               formId.includes(indicator) || 
               formClass.includes(indicator) || 
               formAction.includes(indicator)
           );
}

// Run when DOM is ready
document.addEventListener('DOMContentLoaded', debugFormDetection);
</script>
```

**Common Solutions**:
- Add `data-securepixel-lead="true"` attribute to forms
- Ensure form has proper ID with lead keywords
- Check form is in DOM when SecurePixel loads
- Add manual tracking if automatic detection fails

### E-commerce Tracking Issues

**Problem**: Purchase or product events not tracking correctly.

**Template - Debug E-commerce Events**:
```javascript
// Test purchase tracking
securepixel.debug(true);

securepixel.trackPurchase({
    order_id: '{TEST_ORDER_ID}',
    revenue: {TEST_AMOUNT},
    currency: '{CURRENCY}',
    products: [{
        sku: '{TEST_SKU}',
        name: '{TEST_PRODUCT}',
        price: {TEST_PRICE},
        quantity: 1
    }]
});

console.log('Purchase event sent - check network tab');
```

**Real Example - E-commerce Debug Suite**:
```javascript
// Complete e-commerce debugging suite
function debugEcommerceTracking() {
    console.log('🔍 E-commerce Tracking Debug Suite...');
    
    securepixel.debug(true);
    
    // 1. Test product view
    console.log('📤 Testing product view...');
    securepixel.trackProduct({
        sku: 'DEBUG-WIDGET-001',
        name: 'Debug Widget Test',
        price: 49.99,
        category: 'Debug Products',
        brand: 'Debug Brand',
        currency: 'USD',
        in_stock: true
    });
    
    // 2. Test add to cart
    console.log('📤 Testing add to cart...');
    securepixel.trackAddToCart({
        sku: 'DEBUG-WIDGET-001',
        name: 'Debug Widget Test',
        price: 49.99,
        quantity: 2,
        currency: 'USD',
        cart_value: 99.98
    });
    
    // 3. Test begin checkout
    console.log('📤 Testing begin checkout...');
    securepixel.trackBeginCheckout({
        cart_value: 99.98,
        currency: 'USD',
        item_count: 2,
        products: [{
            sku: 'DEBUG-WIDGET-001',
            quantity: 2
        }]
    });
    
    // 4. Test purchase
    console.log('📤 Testing purchase...');
    securepixel.trackPurchase({
        order_id: 'DEBUG-ORDER-' + Date.now(),
        revenue: 109.97,
        currency: 'USD',
        shipping: 9.99,
        tax: 8.75,
        customer_type: 'new',
        products: [{
            sku: 'DEBUG-WIDGET-001',
            name: 'Debug Widget Test',
            price: 49.99,
            quantity: 2,
            category: 'Debug Products'
        }]
    });
    
    // 5. Test dataLayer integration
    console.log('📤 Testing dataLayer integration...');
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
        'event': 'purchase',
        'ecommerce': {
            'transaction_id': 'DEBUG-DL-' + Date.now(),
            'value': 199.99,
            'currency': 'USD',
            'items': [{
                'item_id': 'DEBUG-DL-WIDGET',
                'item_name': 'DataLayer Debug Widget',
                'price': 199.99,
                'quantity': 1
            }]
        }
    });
    
    setTimeout(() => {
        console.log('✅ E-commerce debug complete. Check:');
        console.log('1. Network tab for multiple /collect requests');
        console.log('2. Each request has correct event_type');
        console.log('3. Product data is properly formatted');
        console.log('4. Revenue values are numbers, not strings');
        console.log('5. DataLayer events are processed automatically');
    }, 2000);
}

// Test e-commerce tracking
if (typeof securepixel !== 'undefined') {
    debugEcommerceTracking();
}
```

**Common Solutions**:
- Ensure numeric values (prices, quantities) are numbers, not strings
- Verify required fields like `order_id` and `sku` are provided
- Check currency codes are valid 3-letter ISO codes
- Test with simple data first, then add complexity

## Data Quality Issues

### Missing Event Data

**Problem**: Events fire but miss important data fields.

**Template - Validate Event Data**:
```javascript
function validateEventData(eventType, eventData) {
    console.log(`Validating ${eventType}:`, eventData);
    
    // Check required fields based on event type
    const requiredFields = {
        'purchase': ['order_id', 'revenue'],
        'product_view': ['sku'],
        'lead_form_submit': ['form_id', 'lead_type']
    };
    
    const required = requiredFields[eventType] || [];
    const missing = required.filter(field => !eventData[field]);
    
    if (missing.length > 0) {
        console.error(`❌ Missing required fields: ${missing.join(', ')}`);
        return false;
    }
    
    console.log('✅ Event data validation passed');
    return true;
}
```

**Real Example - Data Quality Validator**:
```javascript
// Comprehensive data quality validator
class SecurePixelValidator {
    constructor() {
        this.validationRules = {
            purchase: {
                required: ['order_id', 'revenue'],
                types: {
                    order_id: 'string',
                    revenue: 'number',
                    currency: 'string',
                    shipping: 'number',
                    tax: 'number'
                },
                formats: {
                    currency: /^[A-Z]{3}$/,
                    order_id: /^.{1,100}$/
                }
            },
            product_view: {
                required: ['sku'],
                types: {
                    sku: 'string',
                    name: 'string',
                    price: 'number',
                    in_stock: 'boolean'
                }
            },
            lead_form_submit: {
                required: ['form_id', 'lead_type'],
                types: {
                    form_id: 'string',
                    lead_type: 'string',
                    lead_value: 'number'
                }
            }
        };
    }
    
    validate(eventType, eventData) {
        console.log(`🔍 Validating ${eventType}...`);
        
        const rules = this.validationRules[eventType];
        if (!rules) {
            console.warn(`⚠️ No validation rules for ${eventType}`);
            return true;
        }
        
        const errors = [];
        
        // Check required fields
        rules.required.forEach(field => {
            if (eventData[field] === undefined || eventData[field] === null || eventData[field] === '') {
                errors.push(`Missing required field: ${field}`);
            }
        });
        
        // Check data types
        Object.entries(rules.types || {}).forEach(([field, expectedType]) => {
            if (eventData[field] !== undefined) {
                const actualType = typeof eventData[field];
                if (actualType !== expectedType) {
                    errors.push(`Field ${field} should be ${expectedType}, got ${actualType}`);
                }
            }
        });
        
        // Check formats
        Object.entries(rules.formats || {}).forEach(([field, regex]) => {
            if (eventData[field] !== undefined && !regex.test(eventData[field])) {
                errors.push(`Field ${field} format invalid: ${eventData[field]}`);
            }
        });
        
        if (errors.length > 0) {
            console.error(`❌ Validation failed for ${eventType}:`, errors);
            return false;
        }
        
        console.log(`✅ ${eventType} validation passed`);
        return true;
    }
}

// Usage: Validate before sending events
const validator = new SecurePixelValidator();

// Wrap SecurePixel methods with validation
const originalTrackPurchase = securepixel.trackPurchase;
securepixel.trackPurchase = function(data) {
    if (validator.validate('purchase', data)) {
        return originalTrackPurchase.call(this, data);
    } else {
        console.error('❌ Purchase tracking blocked due to validation errors');
    }
};

// Test with invalid data
securepixel.trackPurchase({
    order_id: '',  // Invalid: empty
    revenue: '99.99',  // Invalid: string instead of number
    currency: 'INVALID'  // Invalid: not 3 letters
});

// Test with valid data
securepixel.trackPurchase({
    order_id: 'ORD-2024-001',
    revenue: 99.99,
    currency: 'USD'
});
```

### Privacy Filtering Issues

**Problem**: Expected data is being filtered out by privacy settings.

**Template - Check Privacy Filtering**:
```javascript
// Test what data gets filtered
const testData = {
    email: 'test@example.com',
    phone: '555-123-4567',
    name: 'John Doe',
    company: 'ACME Corp',
    message: 'Interested in product'
};

securepixel.track('privacy_filter_test', testData);
console.log('Check network request to see what data was filtered');
```

**Real Example - Privacy Filter Analysis**:
```javascript
// Test privacy filtering behavior
function testPrivacyFiltering() {
    console.log('🔍 Testing Privacy Filtering...');
    
    const sensitiveData = {
        // These should be filtered for GDPR/HIPAA clients
        email: 'user@example.com',
        phone: '555-123-4567',
        ssn: '123-45-6789',
        credit_card: '4111-1111-1111-1111',
        name: 'John Doe',
        address: '123 Main St',
        
        // These should be kept
        company: 'ACME Corp',
        industry: 'Technology',
        budget: '50000',
        source: 'website',
        interest: 'Product demo'
    };
    
    console.log('📤 Sending test data with PII...');
    securepixel.track('privacy_filter_test', sensitiveData);
    
    console.log('Original data:', sensitiveData);
    console.log('Check network tab to see filtered result');
    
    // Test nested data filtering
    const nestedData = {
        user_info: {
            email: 'nested@example.com',
            company: 'Nested Corp'
        },
        form_data: {
            contact_email: 'form@example.com',
            business_type: 'SaaS'
        }
    };
    
    console.log('📤 Sending nested data test...');
    securepixel.track('nested_privacy_test', nestedData);
    
    setTimeout(() => {
        console.log('✅ Privacy filtering tests sent');
        console.log('Expected behavior:');
        console.log('- Standard privacy: All data kept');
        console.log('- GDPR privacy: PII fields filtered out');
        console.log('- HIPAA privacy: Aggressive PII filtering');
        console.log('Check /collect requests in network tab for actual filtering');
    }, 1000);
}

// Run privacy filtering test
testPrivacyFiltering();
```

## Performance Issues

### Slow Page Load

**Problem**: SecurePixel script affects page load performance.

**Template - Performance Monitoring**:
```javascript
// Monitor SecurePixel loading performance
const startTime = performance.now();

window.addEventListener('load', function() {
    const loadTime = performance.now() - startTime;
    console.log('Page load time with SecurePixel:', loadTime + 'ms');
    
    securepixel.track('performance_monitoring', {
        page_load_time: loadTime,
        script_load_method: 'async',
        performance_timing: performance.timing
    });
});
```

**Real Example - Performance Diagnostics**:
```javascript
// Complete performance diagnostic suite
class SecurePixelPerformanceMonitor {
    constructor() {
        this.startTime = performance.now();
        this.metrics = {};
        this.setupMonitoring();
    }
    
    setupMonitoring() {
        // Monitor script loading
        this.monitorScriptLoading();
        
        // Monitor event batching performance
        this.monitorEventBatching();
        
        // Monitor network requests
        this.monitorNetworkRequests();
        
        // Monitor page load impact
        this.monitorPageLoadImpact();
    }
    
    monitorScriptLoading() {
        const scripts = document.querySelectorAll('script[src*="securepixel"], script[src*="tracking.js"]');
        scripts.forEach(script => {
            script.onload = () => {
                this.metrics.script_load_time = performance.now() - this.startTime;
                console.log('✅ SecurePixel script loaded in:', this.metrics.script_load_time + 'ms');
            };
            
            script.onerror = () => {
                console.error('❌ SecurePixel script failed to load');
            };
        });
    }
    
    monitorEventBatching() {
        let eventCount = 0;
        let batchCount = 0;
        
        // Override console.log to catch SecurePixel debug messages
        const originalLog = console.log;
        console.log = function(...args) {
            if (args[0] && args[0].includes && args[0].includes('[SecurePixel]')) {
                if (args[0].includes('Bulk inserted')) {
                    batchCount++;
                    const events = args[0].match(/\d+/);
                    if (events) eventCount += parseInt(events[0]);
                }
            }
            return originalLog.apply(console, args);
        };
        
        setTimeout(() => {
            this.metrics.events_sent = eventCount;
            this.metrics.batches_sent = batchCount;
            this.metrics.avg_events_per_batch = eventCount / batchCount || 0;
            
            console.log('📊 Event Batching Performance:', {
                events_sent: eventCount,
                batches_sent: batchCount,
                avg_events_per_batch: this.metrics.avg_events_per_batch.toFixed(2)
            });
        }, 10000); // Check after 10 seconds
    }
    
    monitorNetworkRequests() {
        const observer = new PerformanceObserver((list) => {
            list.getEntries().forEach((entry) => {
                if (entry.name.includes('/collect')) {
                    this.metrics.network_requests = this.metrics.network_requests || [];
                    this.metrics.network_requests.push({
                        duration: entry.duration,
                        response_start: entry.responseStart,
                        response_end: entry.responseEnd,
                        transfer_size: entry.transferSize
                    });
                    
                    console.log('📡 Network request to /collect:', {
                        duration: entry.duration.toFixed(2) + 'ms',
                        size: entry.transferSize + ' bytes'
                    });
                }
            });
        });
        
        observer.observe({ entryTypes: ['resource'] });
    }
    
    monitorPageLoadImpact() {
        window.addEventListener('load', () => {
            const timing = performance.timing;
            const pageLoadTime = timing.loadEventEnd - timing.navigationStart;
            
            this.metrics.page_load_time = pageLoadTime;
            this.metrics.dom_ready_time = timing.domContentLoadedEventEnd - timing.navigationStart;
            
            console.log('📊 Page Performance Impact:', {
                total_page_load: pageLoadTime + 'ms',
                dom_ready: this.metrics.dom_ready_time + 'ms',
                script_overhead: this.metrics.script_load_time + 'ms'
            });
            
            // Send performance data
            if (typeof securepixel !== 'undefined') {
                securepixel.track('performance_metrics', this.metrics);
            }
        });
    }
    
    getReport() {
        return this.metrics;
    }
}

// Initialize performance monitoring
const perfMonitor = new SecurePixelPerformanceMonitor();

// Get performance report after 15 seconds
setTimeout(() => {
    console.log('📊 Final Performance Report:', perfMonitor.getReport());
}, 15000);
```

**Common Solutions**:
- Ensure script has `async` attribute
- Load script after critical page content
- Use event batching (enabled by default)
- Monitor network requests for efficiency

## Browser Compatibility Issues

### Old Browser Support

**Problem**: SecurePixel not working in older browsers.

**Template - Browser Compatibility Check**:
```javascript
function checkBrowserCompatibility() {
    const features = {
        fetch: typeof fetch !== 'undefined',
        promise: typeof Promise !== 'undefined',
        arrow_functions: true, // Can't test directly
        const_let: true, // Can't test directly
        json: typeof JSON !== 'undefined'
    };
    
    console.log('Browser compatibility:', features);
    
    const unsupported = Object.entries(features)
        .filter(([key, supported]) => !supported)
        .map(([key]) => key);
        
    if (unsupported.length > 0) {
        console.warn('Unsupported features:', unsupported);
    }
}
```

**Real Example - Browser Compatibility Suite**:
```javascript
// Comprehensive browser compatibility checker
function runBrowserCompatibilityTest() {
    console.log('🔍 Browser Compatibility Test...');
    
    const browserInfo = {
        user_agent: navigator.userAgent,
        platform: navigator.platform,
        language: navigator.language,
        cookie_enabled: navigator.cookieEnabled,
        online: navigator.onLine
    };
    
    console.log('Browser Info:', browserInfo);
    
    // Test core JavaScript features
    const jsFeatures = {
        fetch: typeof fetch !== 'undefined',
        promise: typeof Promise !== 'undefined',
        json: typeof JSON !== 'undefined',
        local_storage: typeof localStorage !== 'undefined',
        session_storage: typeof sessionStorage !== 'undefined',
        query_selector: typeof document.querySelector !== 'undefined',
        add_event_listener: typeof document.addEventListener !== 'undefined',
        request_animation_frame: typeof requestAnimationFrame !== 'undefined'
    };
    
    console.log('JavaScript Features:', jsFeatures);
    
    // Test SecurePixel specific requirements
    const securepixelRequirements = {
        securepixel_loaded: typeof securepixel !== 'undefined',
        console_available: typeof console !== 'undefined',
        performance_api: typeof performance !== 'undefined',
        url_api: typeof URL !== 'undefined'
    };
    
    console.log('SecurePixel Requirements:', securepixelRequirements);
    
    // Identify potential issues
    const issues = [];
    
    if (!jsFeatures.fetch) {
        issues.push('fetch API not supported - may need polyfill');
    }
    
    if (!jsFeatures.promise) {
        issues.push('Promise not supported - may need polyfill');
    }
    
    if (!jsFeatures.local_storage) {
        issues.push('localStorage not available - session tracking may be affected');
    }
    
    if (!browserInfo.cookie_enabled) {
        issues.push('Cookies disabled - tracking may be limited');
    }
    
    // Test specific browser versions
    const isIE = /MSIE|Trident/.test(navigator.userAgent);
    const isOldChrome = /Chrome\/([0-9]+)/.test(navigator.userAgent) && 
                       parseInt(navigator.userAgent.match(/Chrome\/([0-9]+)/)[1]) < 60;
    const isOldSafari = /Safari\/([0-9]+)/.test(navigator.userAgent) && 
                       parseInt(navigator.userAgent.match(/Safari\/([0-9]+)/)[1]) < 537;
    
    if (isIE) {
        issues.push('Internet Explorer detected - limited support');
    }
    
    if (isOldChrome) {
        issues.push('Old Chrome version - may have compatibility issues');
    }
    
    if (isOldSafari) {
        issues.push('Old Safari version - may have compatibility issues');
    }
    
    // Report results
    if (issues.length > 0) {
        console.warn('❌ Compatibility Issues Found:', issues);
        
        // Try to send compatibility report
        if (typeof securepixel !== 'undefined') {
            securepixel.track('browser_compatibility_issues', {
                issues: issues,
                browser_info: browserInfo,
                js_features: jsFeatures,
                timestamp: new Date().toISOString()
            });
        }
    } else {
        console.log('✅ Browser compatibility looks good');
        
        if (typeof securepixel !== 'undefined') {
            securepixel.track('browser_compatibility_success', {
                browser_info: browserInfo,
                js_features: jsFeatures
            });
        }
    }
    
    return {
        compatible: issues.length === 0,
        issues: issues,
        browserInfo: browserInfo,
        jsFeatures: jsFeatures
    };
}

// Run compatibility test
const compatResult = runBrowserCompatibilityTest();

// Provide fallback for incompatible browsers
if (!compatResult.compatible) {
    console.log('🔧 Applying compatibility fixes...');
    
    // Basic fetch polyfill for older browsers
    if (!window.fetch) {
        console.log('Adding basic fetch polyfill...');
        window.fetch = function(url, options) {
            return new Promise(function(resolve, reject) {
                const xhr = new XMLHttpRequest();
                xhr.open(options.method || 'GET', url);
                xhr.onload = function() {
                    resolve({
                        ok: xhr.status >= 200 && xhr.status < 300,
                        status: xhr.status,
                        json: function() {
                            return Promise.resolve(JSON.parse(xhr.responseText));
                        }
                    });
                };
                xhr.onerror = function() {
                    reject(new Error('Network error'));
                };
                if (options.body) {
                    xhr.setRequestHeader('Content-Type', 'application/json');
                    xhr.send(options.body);
                } else {
                    xhr.send();
                }
            });
        };
    }
}
```

## Getting Help

### Debug Information to Collect

When contacting support, collect this debug information:

**Template - Support Debug Info**:
```javascript
function collectDebugInfo() {
    const debugInfo = {
        // Browser information
        user_agent: navigator.userAgent,
        url: window.location.href,
        timestamp: new Date().toISOString(),
        
        // SecurePixel status
        securepixel_loaded: typeof securepixel !== 'undefined',
        client_stats: typeof securepixel !== 'undefined' ? securepixel.getStats() : null,
        
        // Console errors (if any)
        console_errors: [], // Manually note any console errors
        
        // Network requests
        network_status: 'Check network tab for /collect requests'
    };
    
    console.log('Debug Info for Support:', JSON.stringify(debugInfo, null, 2));
    return debugInfo;
}
```

**Real Example - Complete Support Package**:
```javascript
// Generate comprehensive support debug package
function generateSupportPackage() {
    console.log('📦 Generating Support Debug Package...');
    
    const supportPackage = {
        timestamp: new Date().toISOString(),
        
        // Environment info
        environment: {
            url: window.location.href,
            domain: window.location.hostname,
            user_agent: navigator.userAgent,
            screen_resolution: `${screen.width}x${screen.height}`,
            viewport: `${window.innerWidth}x${window.innerHeight}`,
            language: navigator.language,
            timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
            cookie_enabled: navigator.cookieEnabled,
            local_storage: typeof localStorage !== 'undefined',
            connection: navigator.connection ? {
                effective_type: navigator.connection.effectiveType,
                downlink: navigator.connection.downlink
            } : 'unknown'
        },
        
        // SecurePixel status
        securepixel: {
            loaded: typeof securepixel !== 'undefined',
            stats: typeof securepixel !== 'undefined' ? securepixel.getStats() : null,
            methods_available: typeof securepixel !== 'undefined' ? {
                track: typeof securepixel.track === 'function',
                trackPurchase: typeof securepixel.trackPurchase === 'function',
                trackLead: typeof securepixel.trackLead === 'function',
                debug: typeof securepixel.debug === 'function'
            } : null
        },
        
        // Page info
        page: {
            title: document.title,
            forms_count: document.querySelectorAll('form').length,
            lead_forms_count: document.querySelectorAll('form[data-securepixel-lead="true"]').length,
            scripts_count: document.querySelectorAll('script').length,
            securepixel_scripts: Array.from(document.querySelectorAll('script[src*="tracking.js"], script[src*="securepixel"]')).map(s => s.src)
        },
        
        // Console errors
        console_errors: window.consoleErrors || [],
        
        // Performance info
        performance: {
            page_load_time: performance.timing ? performance.timing.loadEventEnd - performance.timing.navigationStart : null,
            dom_ready_time: performance.timing ? performance.timing.domContentLoadedEventEnd - performance.timing.navigationStart : null
        },
        
        // Test results
        test_results: {}
    };
    
    // Run quick tests
    console.log('🧪 Running diagnostic tests...');
    
    // Test 1: Basic tracking
    if (typeof securepixel !== 'undefined') {
        supportPackage.test_results.basic_tracking = 'available';
        
        try {
            securepixel.track('support_debug_test', {
                test: true,
                timestamp: new Date().toISOString()
            });
            supportPackage.test_results.basic_tracking = 'success';
        } catch (error) {
            supportPackage.test_results.basic_tracking = 'error: ' + error.message;
        }
    } else {
        supportPackage.test_results.basic_tracking = 'securepixel_not_loaded';
    }
    
    // Test 2: Network connectivity
    supportPackage.test_results.network_test = 'testing';
    fetch('https://pixel-management-275731808857.us-central1.run.app/health')
        .then(response => response.ok ? 'success' : 'failed')
        .catch(error => 'error: ' + error.message)
        .then(result => {
            supportPackage.test_results.network_test = result;
            console.log('Network test result:', result);
        });
    
    // Output support package
    console.log('📦 Support Debug Package Generated:');
    console.log(JSON.stringify(supportPackage, null, 2));
    
    // Copy to clipboard if available
    if (navigator.clipboard) {
        navigator.clipboard.writeText(JSON.stringify(supportPackage, null, 2))
            .then(() => console.log('✅ Debug package copied to clipboard'))
            .catch(() => console.log('❌ Could not copy to clipboard'));
    }
    
    return supportPackage;
}

// Capture console errors
window.consoleErrors = [];
const originalError = console.error;
console.error = function(...args) {
    window.consoleErrors.push({
        timestamp: new Date().toISOString(),
        message: args.join(' ')
    });
    return originalError.apply(console, args);
};

// Generate support package when needed
// generateSupportPackage();
```

### Contact Support Checklist

Before contacting support:

- [ ] Run `securepixel.debug(true)` and check console output
- [ ] Generate support debug package with `generateSupportPackage()`
- [ ] Check browser network tab for failed requests
- [ ] Note specific error messages and when they occur
- [ ] Test in different browsers if possible
- [ ] Verify domain authorization in SecurePixel dashboard
- [ ] Include client ID and affected domain in support request

### Common Support Scenarios

**1. Events Not Showing Up**
- Check network requests are successful (200 status)
- Verify client ID and domain authorization
- Enable debug mode and check event formatting

**2. Privacy Compliance Issues**
- Confirm privacy level setting in dashboard
- Test data filtering behavior
- Review consent management integration

**3. Performance Problems**
- Monitor page load times with/without SecurePixel
- Check event batching frequency
- Review script loading method (async recommended)

**4. Integration Specific Issues**
- Provide platform version (Shopify, WooCommerce, etc.)
- Include integration code used
- Test manual tracking vs automatic detection

This troubleshooting guide should resolve most common SecurePixel implementation issues. For persistent problems, contact support with the debug information collected using the tools provided above.