# Platform Integration Guide

Step-by-step integration guides for popular e-commerce and website platforms with SecurePixel tracking.

## Shopify Integration

### Step 1: Add SecurePixel Script

**Template - Add to theme.liquid**:
```liquid
<!-- Add to <head> section in theme.liquid -->
<script src="https://pixel-management-275731808857.us-central1.run.app/pixel/{YOUR_CLIENT_ID}/tracking.js" async></script>
```

**Real Example**:
```liquid
<!-- Add to <head> section in theme.liquid -->
<script src="https://pixel-management-275731808857.us-central1.run.app/pixel/acme-store-2024/tracking.js" async></script>
```

### Step 2: Order Tracking

**Template - Add to checkout.liquid or order-status.liquid**:
```liquid
{% if first_time_accessed and order %}
<script>
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
    'event': 'purchase',
    'ecommerce': {
        'transaction_id': '{{ order.order_number }}',
        'value': {{ order.total_price | divided_by: 100.0 }},
        'currency': '{{ shop.currency }}',
        'shipping': {{ order.shipping_price | divided_by: 100.0 }},
        'tax': {{ order.tax_price | divided_by: 100.0 }},
        'items': [
            {% for line_item in order.line_items %}
            {
                'item_id': '{{ line_item.sku | default: line_item.variant_id }}',
                'item_name': '{{ line_item.title | escape }}',
                'price': {{ line_item.price | divided_by: 100.0 }},
                'quantity': {{ line_item.quantity }},
                'item_category': '{{ line_item.product.type | escape }}'
            }{% unless forloop.last %},{% endunless %}
            {% endfor %}
        ]
    }
});
</script>
{% endif %}
```

**Real Example - Working Shopify Order Tracking**:
```liquid
{% if first_time_accessed and order %}
<script>
// Track successful order completion
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
    'event': 'purchase',
    'ecommerce': {
        'transaction_id': '{{ order.order_number }}',
        'value': {{ order.total_price | divided_by: 100.0 }},
        'currency': '{{ shop.currency }}',
        'shipping': {{ order.shipping_price | divided_by: 100.0 }},
        'tax': {{ order.tax_price | divided_by: 100.0 }},
        'coupon': {% if order.discount_applications.size > 0 %}'{{ order.discount_applications[0].title }}'{% else %}null{% endif %},
        'items': [
            {% for line_item in order.line_items %}
            {
                'item_id': '{{ line_item.sku | default: line_item.variant_id }}',
                'item_name': '{{ line_item.title | escape }}',
                'price': {{ line_item.price | divided_by: 100.0 }},
                'quantity': {{ line_item.quantity }},
                'item_category': '{{ line_item.product.type | escape }}',
                'item_brand': '{{ line_item.vendor | escape }}'
            }{% unless forloop.last %},{% endunless %}
            {% endfor %}
        ]
    }
});

// Also track customer type for attribution
securepixel.track('customer_info', {
    customer_type: {% if customer.orders_count == 1 %}'new'{% else %}'returning'{% endif %},
    total_orders: {{ customer.orders_count | default: 1 }},
    accepts_marketing: {{ customer.accepts_marketing }}
});
</script>
{% endif %}
```

### Step 3: Product View Tracking

**Template - Add to product.liquid**:
```liquid
<script>
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
    'event': 'view_item',
    'ecommerce': {
        'currency': '{{ shop.currency }}',
        'value': {{ product.price | divided_by: 100.0 }},
        'items': [{
            'item_id': '{{ product.selected_or_first_available_variant.sku | default: product.selected_or_first_available_variant.id }}',
            'item_name': '{{ product.title | escape }}',
            'price': {{ product.price | divided_by: 100.0 }},
            'item_category': '{{ product.type | escape }}',
            'item_brand': '{{ product.vendor | escape }}'
        }]
    }
});
</script>
```

**Real Example - Shopify Product View**:
```liquid
<script>
// Track product page views
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
    'event': 'view_item',
    'ecommerce': {
        'currency': '{{ shop.currency }}',
        'value': {{ product.price | divided_by: 100.0 }},
        'items': [{
            'item_id': '{{ product.selected_or_first_available_variant.sku | default: product.selected_or_first_available_variant.id }}',
            'item_name': '{{ product.title | escape }}',
            'price': {{ product.price | divided_by: 100.0 }},
            'item_category': '{{ product.type | escape }}',
            'item_brand': '{{ product.vendor | escape }}',
            'variant': '{{ product.selected_or_first_available_variant.title | escape }}',
            'in_stock': {{ product.available }}
        }]
    }
});

// Track additional product engagement if user scrolls past product images
window.addEventListener('scroll', function() {
    if (window.scrollY > 400 && !window.productEngagementTracked) {
        window.productEngagementTracked = true;
        securepixel.track('product_engagement', {
            product_sku: '{{ product.selected_or_first_available_variant.sku }}',
            engagement_type: 'detailed_view',
            product_price: {{ product.price | divided_by: 100.0 }},
            product_available: {{ product.available }}
        });
    }
});
</script>
```

### Step 4: Cart Events

**Template - AJAX Cart Integration**:
```javascript
// Add to cart.js or theme JavaScript files
document.addEventListener('DOMContentLoaded', function() {
    // Override Shopify's AJAX add to cart
    document.addEventListener('submit', function(e) {
        if (e.target.matches('form[action*="/cart/add"]')) {
            e.preventDefault();
            
            const form = e.target;
            const formData = new FormData(form);
            const variantId = formData.get('id');
            const quantity = parseInt(formData.get('quantity')) || 1;
            
            // Track add to cart before sending to Shopify
            fetch('/products/' + window.location.pathname.split('/').pop() + '.js')
                .then(response => response.json())
                .then(product => {
                    const variant = product.variants.find(v => v.id == variantId);
                    
                    window.dataLayer = window.dataLayer || [];
                    window.dataLayer.push({
                        'event': 'add_to_cart',
                        'ecommerce': {
                            'currency': '{SHOP_CURRENCY}',
                            'value': (variant.price / 100) * quantity,
                            'items': [{
                                'item_id': variant.sku || variant.id,
                                'item_name': product.title,
                                'price': variant.price / 100,
                                'quantity': quantity
                            }]
                        }
                    });
                });
            
            // Continue with Shopify add to cart
            form.submit();
        }
    });
});
```

**Real Example - Working Shopify Cart Tracking**:
```javascript
// Complete cart tracking for Shopify stores
document.addEventListener('DOMContentLoaded', function() {
    // Track add to cart events
    document.addEventListener('submit', function(e) {
        if (e.target.matches('form[action*="/cart/add"]')) {
            e.preventDefault();
            
            const form = e.target;
            const formData = new FormData(form);
            const variantId = formData.get('id');
            const quantity = parseInt(formData.get('quantity')) || 1;
            
            // Get product data and track
            fetch('/products/' + window.location.pathname.split('/').pop() + '.js')
                .then(response => response.json())  
                .then(product => {
                    const variant = product.variants.find(v => v.id == variantId);
                    
                    // Track via dataLayer (automatically processed by SecurePixel)
                    window.dataLayer = window.dataLayer || [];
                    window.dataLayer.push({
                        'event': 'add_to_cart',
                        'ecommerce': {
                            'currency': 'USD',
                            'value': (variant.price / 100) * quantity,
                            'items': [{
                                'item_id': variant.sku || variant.id,
                                'item_name': product.title,
                                'price': variant.price / 100,
                                'quantity': quantity,
                                'item_category': product.product_type,
                                'item_brand': product.vendor
                            }]
                        }
                    });
                })
                .catch(error => {
                    console.error('Product fetch error:', error);
                })
                .finally(() => {
                    // Continue with Shopify add to cart
                    form.submit();
                });
        }
    });
    
    // Track begin checkout
    document.addEventListener('click', function(e) {
        if (e.target.matches('.btn[href*="/checkout"], .checkout-button, [href*="/checkout"]')) {
            fetch('/cart.js')
                .then(response => response.json())
                .then(cart => {
                    window.dataLayer = window.dataLayer || [];
                    window.dataLayer.push({
                        'event': 'begin_checkout',
                        'ecommerce': {
                            'currency': 'USD',
                            'value': cart.total_price / 100,
                            'items': cart.items.map(item => ({
                                'item_id': item.sku || item.variant_id,
                                'item_name': item.product_title,
                                'price': item.price / 100,
                                'quantity': item.quantity
                            }))
                        }
                    });
                });
        }
    });
});
```

## WooCommerce Integration

### Step 1: Add SecurePixel Script

**Template - Add to functions.php**:
```php
function add_securepixel_script() {
    ?>
    <script src="https://pixel-management-275731808857.us-central1.run.app/pixel/{YOUR_CLIENT_ID}/tracking.js" async></script>
    <?php
}
add_action('wp_head', 'add_securepixel_script');
```

**Real Example**:
```php
function add_securepixel_script() {
    ?>
    <script src="https://pixel-management-275731808857.us-central1.run.app/pixel/acme-woo-2024/tracking.js" async></script>
    <?php
}
add_action('wp_head', 'add_securepixel_script');
```

### Step 2: Order Tracking

**Template - Purchase Tracking**:
```php
function securepixel_track_purchase($order_id) {
    $order = wc_get_order($order_id);
    if (!$order) return;
    
    $items = array();
    foreach ($order->get_items() as $item) {
        $product = $item->get_product();
        $items[] = array(
            'item_id' => $product->get_sku() ?: $product->get_id(),
            'item_name' => $item->get_name(),
            'price' => floatval($item->get_subtotal() / $item->get_quantity()),
            'quantity' => $item->get_quantity(),
            'item_category' => '{PRODUCT_CATEGORY}'
        );
    }
    ?>
    <script>
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
        'event': 'purchase',
        'ecommerce': {
            'transaction_id': '<?php echo $order->get_order_number(); ?>',
            'value': <?php echo $order->get_total(); ?>,
            'currency': '<?php echo $order->get_currency(); ?>',
            'shipping': <?php echo $order->get_shipping_total(); ?>,
            'tax': <?php echo $order->get_total_tax(); ?>,
            'items': <?php echo json_encode($items); ?>
        }
    });
    </script>
    <?php
}
add_action('woocommerce_thankyou', 'securepixel_track_purchase');
```

**Real Example - Complete WooCommerce Purchase Tracking**:
```php
function securepixel_track_purchase($order_id) {
    $order = wc_get_order($order_id);
    if (!$order) return;
    
    // Prevent duplicate tracking
    if ($order->get_meta('_securepixel_tracked')) return;
    $order->update_meta_data('_securepixel_tracked', true);
    $order->save();
    
    $items = array();
    foreach ($order->get_items() as $item) {
        $product = $item->get_product();
        $categories = wp_get_post_terms($product->get_id(), 'product_cat', array('fields' => 'names'));
        
        $items[] = array(
            'item_id' => $product->get_sku() ?: $product->get_id(),
            'item_name' => $item->get_name(),
            'price' => floatval($item->get_subtotal() / $item->get_quantity()),
            'quantity' => $item->get_quantity(),
            'item_category' => !empty($categories) ? $categories[0] : 'General',
            'item_brand' => get_post_meta($product->get_id(), '_brand', true) ?: get_bloginfo('name')
        );
    }
    
    // Determine customer type
    $customer_orders = wc_get_customer_order_count($order->get_customer_id());
    $customer_type = $customer_orders <= 1 ? 'new' : 'returning';
    
    ?>
    <script>
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
        'event': 'purchase',
        'ecommerce': {
            'transaction_id': '<?php echo $order->get_order_number(); ?>',
            'value': <?php echo $order->get_total(); ?>,
            'currency': '<?php echo $order->get_currency(); ?>',
            'shipping': <?php echo $order->get_shipping_total() ?: 0; ?>,
            'tax': <?php echo $order->get_total_tax() ?: 0; ?>,
            'coupon': '<?php echo implode(', ', $order->get_coupon_codes()); ?>',
            'items': <?php echo json_encode($items); ?>
        }
    });
    
    // Track customer info
    securepixel.track('customer_info', {
        customer_type: '<?php echo $customer_type; ?>',
        total_orders: <?php echo $customer_orders; ?>,
        customer_value: <?php echo wc_get_customer_total_spent($order->get_customer_id()); ?>
    });
    </script>
    <?php
}
add_action('woocommerce_thankyou', 'securepixel_track_purchase');
```

### Step 3: Product View Tracking

**Template - Product Page Tracking**:
```php
function securepixel_track_product_view() {
    if (!is_product()) return;
    
    global $product;
    $categories = wp_get_post_terms($product->get_id(), 'product_cat', array('fields' => 'names'));
    ?>
    <script>
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
        'event': 'view_item',
        'ecommerce': {
            'currency': '<?php echo get_woocommerce_currency(); ?>',
            'value': <?php echo $product->get_price(); ?>,
            'items': [{
                'item_id': '<?php echo $product->get_sku() ?: $product->get_id(); ?>',
                'item_name': '<?php echo $product->get_name(); ?>',
                'price': <?php echo $product->get_price(); ?>,
                'item_category': '<?php echo !empty($categories) ? $categories[0] : 'General'; ?>',
                'in_stock': <?php echo $product->is_in_stock() ? 'true' : 'false'; ?>
            }]
        }
    });
    </script>
    <?php
}
add_action('wp_footer', 'securepixel_track_product_view');
```

**Real Example - WooCommerce Product Tracking**:
```php
function securepixel_track_product_view() {
    if (!is_product()) return;
    
    global $product;
    $categories = wp_get_post_terms($product->get_id(), 'product_cat', array('fields' => 'names'));
    $brand = get_post_meta($product->get_id(), '_brand', true) ?: get_bloginfo('name');
    $tags = wp_get_post_terms($product->get_id(), 'product_tag', array('fields' => 'names'));
    ?>
    <script>
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
        'event': 'view_item',
        'ecommerce': {
            'currency': '<?php echo get_woocommerce_currency(); ?>',
            'value': <?php echo $product->get_price() ?: 0; ?>,
            'items': [{
                'item_id': '<?php echo $product->get_sku() ?: $product->get_id(); ?>',
                'item_name': '<?php echo esc_js($product->get_name()); ?>',
                'price': <?php echo $product->get_price() ?: 0; ?>,
                'item_category': '<?php echo !empty($categories) ? esc_js($categories[0]) : 'General'; ?>',
                'item_brand': '<?php echo esc_js($brand); ?>',
                'in_stock': <?php echo $product->is_in_stock() ? 'true' : 'false'; ?>,
                'item_variant': '<?php echo $product->get_type() === 'variable' ? 'variable' : 'simple'; ?>'
            }]
        }
    });
    
    // Track product engagement after 10 seconds on page
    setTimeout(function() {
        securepixel.track('product_engagement', {
            product_id: '<?php echo $product->get_id(); ?>',
            product_sku: '<?php echo $product->get_sku() ?: $product->get_id(); ?>',
            engagement_type: 'time_on_page',
            time_threshold: 10,
            product_type: '<?php echo $product->get_type(); ?>'
        });
    }, 10000);
    </script>
    <?php
}
add_action('wp_footer', 'securepixel_track_product_view');
```

### Step 4: AJAX Cart Tracking

**Template - Add to Cart Events**:
```php
function securepixel_add_to_cart_script() {
    ?>
    <script>
    jQuery(document).ready(function($) {
        $('body').on('added_to_cart', function(event, fragments, cart_hash, button) {
            var product_id = button.data('product_id');
            var quantity = button.data('quantity') || 1;
            
            // Get product data via AJAX
            $.post('<?php echo admin_url('admin-ajax.php'); ?>', {
                action: 'get_product_data',
                product_id: product_id
            }, function(data) {
                if (data.success) {
                    window.dataLayer = window.dataLayer || [];
                    window.dataLayer.push({
                        'event': 'add_to_cart',
                        'ecommerce': {
                            'currency': data.data.currency,
                            'value': parseFloat(data.data.price) * quantity,
                            'items': [{
                                'item_id': data.data.sku,
                                'item_name': data.data.name,
                                'price': parseFloat(data.data.price),
                                'quantity': quantity,
                                'item_category': data.data.category
                            }]
                        }
                    });
                }
            });
        });
    });
    </script>
    <?php
}
add_action('wp_footer', 'securepixel_add_to_cart_script');

// AJAX handler for product data
function get_product_data_ajax() {
    $product_id = intval($_POST['product_id']);
    $product = wc_get_product($product_id);
    
    if (!$product) {
        wp_die('Product not found');
    }
    
    $categories = wp_get_post_terms($product_id, 'product_cat', array('fields' => 'names'));
    
    wp_send_json_success(array(
        'sku' => $product->get_sku() ?: $product_id,
        'name' => $product->get_name(),
        'price' => $product->get_price(),
        'category' => !empty($categories) ? $categories[0] : 'General',
        'currency' => get_woocommerce_currency()
    ));
}
add_action('wp_ajax_get_product_data', 'get_product_data_ajax');
add_action('wp_ajax_nopriv_get_product_data', 'get_product_data_ajax');
```

## WordPress (Non-E-commerce) Integration

### Step 1: Basic Script Installation

**Template - Add to functions.php**:
```php
function add_securepixel_to_wordpress() {
    ?>
    <script src="https://pixel-management-275731808857.us-central1.run.app/pixel/{YOUR_CLIENT_ID}/tracking.js" async></script>
    <?php
}
add_action('wp_head', 'add_securepixel_to_wordpress');
```

**Real Example - Content Site Tracking**:
```php
function add_securepixel_to_wordpress() {
    ?>
    <script src="https://pixel-management-275731808857.us-central1.run.app/pixel/content-site-2024/tracking.js" async></script>
    
    <script>
    // Track page type and content info
    document.addEventListener('DOMContentLoaded', function() {
        securepixel.track('page_context', {
            page_type: '<?php 
                if (is_home() || is_front_page()) echo 'homepage';
                elseif (is_single()) echo 'blog_post';
                elseif (is_page()) echo 'page';
                elseif (is_category()) echo 'category';
                elseif (is_archive()) echo 'archive';
                else echo 'other';
            ?>',
            <?php if (is_single()): ?>
            post_id: <?php echo get_the_ID(); ?>,
            post_title: '<?php echo esc_js(get_the_title()); ?>',
            post_category: '<?php echo esc_js(get_the_category()[0]->name ?? 'Uncategorized'); ?>',
            post_author: '<?php echo esc_js(get_the_author()); ?>',
            post_date: '<?php echo get_the_date('c'); ?>',
            <?php endif; ?>
            user_logged_in: <?php echo is_user_logged_in() ? 'true' : 'false'; ?>
        });
    });
    </script>
    <?php
}
add_action('wp_head', 'add_securepixel_to_wordpress');
```

### Step 2: Contact Form Integration

**Template - Contact Form 7 Integration**:
```php
function securepixel_cf7_tracking($contact_form) {
    ?>
    <script>
    document.addEventListener('wpcf7mailsent', function(event) {
        securepixel.trackLead({
            form_id: 'cf7-' + event.detail.contactFormId,
            lead_type: 'contact',
            lead_value: 100,
            source: 'contact_form',
            form_title: '<?php echo $contact_form->title(); ?>'
        });
    });
    </script>
    <?php
}
add_action('wpcf7_before_send_mail', 'securepixel_cf7_tracking');
```

## Custom HTML/JavaScript Sites

### Template - Complete Integration
```html
<!DOCTYPE html>
<html>
<head>
    <!-- SecurePixel Tracking -->
    <script src="https://pixel-management-275731808857.us-central1.run.app/pixel/{YOUR_CLIENT_ID}/tracking.js" async></script>
</head>
<body>
    <!-- Your content -->
    
    <script>
    // Wait for SecurePixel to load
    document.addEventListener('DOMContentLoaded', function() {
        if (typeof securepixel !== 'undefined') {
            // Track custom page events
            securepixel.track('{EVENT_TYPE}', {
                {PROPERTY_NAME}: '{PROPERTY_VALUE}'
            });
        }
    });
    </script>
</body>
</html>
```

**Real Example - Landing Page**:
```html
<!DOCTYPE html>
<html>
<head>
    <title>ACME Product Launch</title>
    <!-- SecurePixel Tracking -->
    <script src="https://pixel-management-275731808857.us-central1.run.app/pixel/acme-landing-2024/tracking.js" async></script>
</head>
<body>
    <header>
        <h1>Revolutionary Product Launch</h1>
        <button id="cta-button" class="cta-btn">Get Early Access</button>
    </header>
    
    <main>
        <section id="features">
            <h2>Amazing Features</h2>
            <!-- Feature content -->
        </section>
        
        <section id="signup-section">
            <form id="early-access-form" data-securepixel-lead="true" data-lead-type="early_access" data-lead-value="500">
                <input type="email" name="email" placeholder="Enter your email" required>
                <button type="submit">Get Early Access</button>
            </form>
        </section>
    </main>
    
    <script>
    document.addEventListener('DOMContentLoaded', function() {
        // Track landing page context
        securepixel.track('landing_page_view', {
            campaign: 'product_launch_q4',
            page_variant: 'hero_video',
            traffic_source: new URLSearchParams(window.location.search).get('utm_source') || 'direct'
        });
        
        // Track CTA button clicks
        document.getElementById('cta-button').addEventListener('click', function() {
            securepixel.track('cta_click', {
                cta_text: this.textContent,
                cta_location: 'hero_section',
                page_section: 'header'
            });
            
            // Scroll to signup form
            document.getElementById('signup-section').scrollIntoView({ behavior: 'smooth' });
        });
        
        // Track form interaction
        const form = document.getElementById('early-access-form');
        form.addEventListener('submit', function(e) {
            e.preventDefault();
            
            // Track the lead (automatically detected by SecurePixel)
            // Manual tracking for additional context
            securepixel.track('early_access_signup', {
                email_provided: this.querySelector('input[name="email"]').value ? true : false,
                referrer: document.referrer || 'direct',
                time_on_page: Math.round((Date.now() - performance.timing.navigationStart) / 1000)
            });
            
            // Submit the form
            this.submit();
        });
        
        // Track scroll depth engagement
        let maxScroll = 0;
        window.addEventListener('scroll', function() {
            const scrollPercent = Math.round((window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100);
            
            if (scrollPercent > maxScroll && scrollPercent % 25 === 0) {
                maxScroll = scrollPercent;
                securepixel.track('scroll_milestone', {
                    scroll_depth: scrollPercent,
                    content_section: getCurrentSection()
                });
            }
        });
        
        function getCurrentSection() {
            const sections = ['header', 'features', 'signup-section'];
            for (let section of sections) {
                const element = document.getElementById(section);
                if (element && isElementInViewport(element)) {
                    return section;
                }
            }
            return 'unknown';
        }
        
        function isElementInViewport(el) {
            const rect = el.getBoundingClientRect();
            return rect.top >= 0 && rect.bottom <= window.innerHeight;
        }
    });
    </script>
</body>
</html>
```

## Testing Your Integration

### 1. Verify Script Loading

**Check Browser Console**:
```javascript
// Look for these messages:
[SecurePixel] Analytics pixel loaded for client: your-client-id
[SecurePixel] DataLayer integration active

// Test manual tracking:
securepixel.track('test_event', { test: true });
```

### 2. Debug Mode

**Enable Debug Output**:
```javascript
securepixel.debug(true);
console.log(securepixel.getStats());
```

### 3. Network Monitoring

**Check Network Tab in DevTools**:
- Look for requests to `/collect` endpoint
- Verify JSON payloads contain expected data
- Check for 200 OK responses

### 4. DataLayer Verification

**Test DataLayer Events**:
```javascript
// Check dataLayer exists
console.log(window.dataLayer);

// Test manual dataLayer push
window.dataLayer.push({
    event: 'test_purchase',
    ecommerce: {
        transaction_id: 'TEST-123',
        value: 99.99,
        currency: 'USD'
    }
});
```

## Common Integration Issues

### 1. Script Loading Problems
- **Issue**: SecurePixel not defined
- **Solution**: Ensure script loads before other tracking code
- **Fix**: Add `async` but verify loading with event listeners

### 2. Shopify Liquid Syntax Errors  
- **Issue**: Liquid template errors
- **Solution**: Properly escape variables and handle missing data
- **Fix**: Use `| default:` filters and null checks

### 3. WooCommerce Hook Conflicts
- **Issue**: Purchase tracking fires multiple times
- **Solution**: Use order meta to prevent duplicates
- **Fix**: Check `_securepixel_tracked` meta before tracking

### 4. AJAX Cart Issues
- **Issue**: Add to cart events not firing
- **Solution**: Hook into platform-specific AJAX events
- **Fix**: Use proper event listeners for each platform

## Platform-Specific Support

Need help with a specific platform not covered here? SecurePixel integrates with:

- **Magento**: Custom module available
- **BigCommerce**: App store integration
- **Squarespace**: Code injection method
- **Webflow**: Custom code integration
- **React/Vue/Angular**: NPM package available

Contact support for platform-specific integration guides and technical assistance.