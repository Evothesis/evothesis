# E-commerce Tracking Guide

Comprehensive e-commerce tracking for product views, cart events, purchases, and revenue attribution with SecurePixel.

## Purchase Tracking

### Template - Order Completion
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
            quantity: {QUANTITY},
            category: '{PRODUCT_CATEGORY}'
        }
    ],
    shipping: {SHIPPING_COST},
    tax: {TAX_AMOUNT},
    customer_type: '{NEW_OR_RETURNING}',
    coupon_code: '{DISCOUNT_CODE}'
});
```

### Real Example - Order Confirmation Page
```javascript
// Track completed purchase on order confirmation page
securepixel.trackPurchase({
    order_id: 'ORD-2024-001',
    revenue: 149.99,
    currency: 'USD',
    products: [
        {
            sku: 'WIDGET-BLUE-M',
            name: 'Blue Widget Medium',
            price: 49.99,
            quantity: 3,
            category: 'Widgets'
        }
    ],
    shipping: 9.99,
    tax: 12.50,
    customer_type: 'returning',
    coupon_code: 'SAVE10'
});
```

## Product View Tracking

### Template - Product Page Views
```javascript
securepixel.trackProduct({
    sku: '{PRODUCT_SKU}',
    name: '{PRODUCT_NAME}',
    price: {PRODUCT_PRICE},
    category: '{PRODUCT_CATEGORY}',
    brand: '{PRODUCT_BRAND}',
    currency: '{CURRENCY_CODE}',
    in_stock: {STOCK_STATUS}
});
```

### Real Example - Product Page Tracking
```javascript
// Track product view on product detail pages
window.addEventListener('DOMContentLoaded', function() {
    // Get product data from page elements or JavaScript variables
    const productData = {
        sku: document.querySelector('[data-product-sku]').textContent,
        name: document.querySelector('h1.product-title').textContent,
        price: parseFloat(document.querySelector('.price').textContent.replace('$', '')),
        category: document.querySelector('.breadcrumb').textContent.split('/')[1],
        brand: document.querySelector('.brand-name').textContent,
        currency: 'USD',
        in_stock: document.querySelector('.stock-status').textContent === 'In Stock'
    };
    
    securepixel.trackProduct(productData);
});
```

## Shopping Cart Events

### Template - Add to Cart
```javascript
securepixel.trackAddToCart({
    sku: '{PRODUCT_SKU}',
    name: '{PRODUCT_NAME}',
    price: {UNIT_PRICE},
    quantity: {QUANTITY_ADDED},
    currency: '{CURRENCY_CODE}',
    cart_value: {TOTAL_CART_VALUE}
});
```

### Real Example - Add to Cart Button
```javascript
// Track add to cart events
document.querySelectorAll('.add-to-cart-btn').forEach(function(button) {
    button.addEventListener('click', function(e) {
        e.preventDefault();
        
        const productCard = this.closest('.product-card');
        const sku = productCard.dataset.productSku;
        const name = productCard.querySelector('.product-name').textContent;
        const price = parseFloat(productCard.querySelector('.price').textContent.replace('$', ''));
        const quantity = parseInt(productCard.querySelector('.quantity-selector').value) || 1;
        
        // Track the add to cart event
        securepixel.trackAddToCart({
            sku: sku,
            name: name,
            price: price,
            quantity: quantity,
            currency: 'USD',
            cart_value: calculateCartTotal() + (price * quantity)
        });
        
        // Continue with adding item to cart
        addItemToCart(sku, quantity);
    });
});

function calculateCartTotal() {
    // Helper function to calculate current cart total
    return Array.from(document.querySelectorAll('.cart-item')).reduce((total, item) => {
        const price = parseFloat(item.querySelector('.item-price').textContent.replace('$', ''));
        const qty = parseInt(item.querySelector('.item-quantity').textContent);
        return total + (price * qty);
    }, 0);
}
```

### Template - Remove from Cart
```javascript
securepixel.trackRemoveFromCart({
    sku: '{PRODUCT_SKU}',
    name: '{PRODUCT_NAME}',
    quantity: {QUANTITY_REMOVED},
    price: {UNIT_PRICE},
    currency: '{CURRENCY_CODE}'
});
```

### Real Example - Cart Page Item Removal
```javascript
// Track remove from cart events
document.querySelectorAll('.remove-item-btn').forEach(function(button) {
    button.addEventListener('click', function(e) {
        e.preventDefault();
        
        const cartItem = this.closest('.cart-item');
        const sku = cartItem.dataset.productSku;
        const name = cartItem.querySelector('.item-name').textContent;
        const price = parseFloat(cartItem.querySelector('.item-price').textContent.replace('$', ''));
        const quantity = parseInt(cartItem.querySelector('.quantity-input').value);
        
        securepixel.trackRemoveFromCart({
            sku: sku,
            name: name,
            quantity: quantity,
            price: price,
            currency: 'USD'
        });
        
        // Remove item from cart
        removeItemFromCart(sku);
    });
});
```

## Checkout Process Tracking

### Template - Begin Checkout
```javascript
securepixel.trackBeginCheckout({
    cart_value: {TOTAL_CART_VALUE},
    currency: '{CURRENCY_CODE}',
    item_count: {TOTAL_ITEMS},
    products: [
        {
            sku: '{PRODUCT_SKU}',
            quantity: {QUANTITY}
        }
    ]
});
```

### Real Example - Checkout Initiation
```javascript
// Track when user begins checkout process
document.getElementById('proceed-to-checkout').addEventListener('click', function() {
    const cartItems = Array.from(document.querySelectorAll('.cart-item'));
    const cartValue = calculateCartTotal();
    const itemCount = cartItems.reduce((count, item) => {
        return count + parseInt(item.querySelector('.quantity-input').value);
    }, 0);
    
    const products = cartItems.map(item => ({
        sku: item.dataset.productSku,
        quantity: parseInt(item.querySelector('.quantity-input').value)
    }));
    
    securepixel.trackBeginCheckout({
        cart_value: cartValue,
        currency: 'USD',
        item_count: itemCount,
        products: products
    });
});
```

### Template - Checkout Progress
```javascript
securepixel.trackCheckoutProgress({
    step: {STEP_NUMBER},
    step_name: '{STEP_DESCRIPTION}',
    cart_value: {CART_VALUE},
    currency: '{CURRENCY_CODE}'
});
```

### Real Example - Multi-Step Checkout
```javascript
// Track progress through checkout steps
let checkoutStep = 1;

function trackCheckoutStep(stepName) {
    securepixel.trackCheckoutProgress({
        step: checkoutStep,
        step_name: stepName,
        cart_value: calculateCartTotal(),
        currency: 'USD'
    });
}

// Track shipping information step
document.getElementById('shipping-form').addEventListener('submit', function(e) {
    e.preventDefault();
    trackCheckoutStep('shipping_info');
    checkoutStep = 2;
    showPaymentStep();
});

// Track payment information step
document.getElementById('payment-form').addEventListener('submit', function(e) {
    e.preventDefault();
    trackCheckoutStep('payment_info');
    checkoutStep = 3;
    showOrderReview();
});

// Track order review step
document.getElementById('place-order-btn').addEventListener('click', function() {
    trackCheckoutStep('order_review');
    checkoutStep = 4;
    // Final purchase tracking happens on confirmation page
});
```

## Product Category and Search Tracking

### Template - Category Browsing
```javascript
securepixel.track('category_view', {
    category_name: '{CATEGORY_NAME}',
    category_level: {CATEGORY_DEPTH},
    products_shown: {PRODUCT_COUNT},
    sort_order: '{SORT_METHOD}',
    filters_applied: {ACTIVE_FILTERS}
});
```

### Real Example - Category Page Tracking
```javascript
// Track category page views and interactions
window.addEventListener('DOMContentLoaded', function() {
    const categoryName = document.querySelector('.category-title').textContent;
    const productCount = document.querySelectorAll('.product-card').length;
    const sortSelect = document.querySelector('.sort-select');
    const activeFilters = Array.from(document.querySelectorAll('.filter-tag')).map(tag => tag.textContent);
    
    securepixel.track('category_view', {
        category_name: categoryName,
        category_level: 1,
        products_shown: productCount,
        sort_order: sortSelect.value,
        filters_applied: activeFilters
    });
});

// Track sort changes
document.querySelector('.sort-select').addEventListener('change', function() {
    securepixel.track('product_sort', {
        category_name: document.querySelector('.category-title').textContent,
        sort_method: this.value,
        products_shown: document.querySelectorAll('.product-card').length
    });
});
```

### Template - Product Search
```javascript
securepixel.track('product_search', {
    search_term: '{SEARCH_QUERY}',
    search_results_count: {RESULTS_COUNT},
    category_filter: '{CATEGORY_FILTER}',
    price_range: '{PRICE_FILTER}'
});
```

### Real Example - Search Results Tracking
```javascript
// Track product search usage
document.getElementById('product-search-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    const searchTerm = document.getElementById('search-input').value;
    const categoryFilter = document.querySelector('.category-filter').value;
    const priceFilter = document.querySelector('.price-filter').value;
    
    // Perform search then track results
    performProductSearch(searchTerm, categoryFilter, priceFilter).then(function(results) {
        securepixel.track('product_search', {
            search_term: searchTerm,
            search_results_count: results.length,
            category_filter: categoryFilter || 'all',
            price_range: priceFilter || 'all'
        });
    });
});
```

## Wishlist and Favorites Tracking

### Template - Wishlist Events
```javascript
securepixel.track('wishlist_add', {
    sku: '{PRODUCT_SKU}',
    name: '{PRODUCT_NAME}',
    price: {PRODUCT_PRICE},
    category: '{PRODUCT_CATEGORY}',
    wishlist_size: {TOTAL_WISHLIST_ITEMS}
});
```

### Real Example - Product Wishlist
```javascript
// Track wishlist interactions
document.querySelectorAll('.wishlist-btn').forEach(function(button) {
    button.addEventListener('click', function() {
        const productCard = this.closest('.product-card');
        const sku = productCard.dataset.productSku;
        const name = productCard.querySelector('.product-name').textContent;
        const price = parseFloat(productCard.querySelector('.price').textContent.replace('$', ''));
        const category = productCard.dataset.category;
        
        const isAdding = !this.classList.contains('wishlisted');
        
        if (isAdding) {
            securepixel.track('wishlist_add', {
                sku: sku,
                name: name,
                price: price,
                category: category,
                wishlist_size: getWishlistCount() + 1
            });
            this.classList.add('wishlisted');
        } else {
            securepixel.track('wishlist_remove', {
                sku: sku,
                name: name,
                price: price,
                category: category,
                wishlist_size: getWishlistCount() - 1
            });
            this.classList.remove('wishlisted');
        }
    });
});

function getWishlistCount() {
    return document.querySelectorAll('.wishlist-btn.wishlisted').length;
}
```

## DataLayer Integration for E-commerce

### Template - DataLayer Purchase Event
```javascript
dataLayer.push({
    'event': 'purchase',
    'ecommerce': {
        'transaction_id': '{TRANSACTION_ID}',
        'value': {TOTAL_VALUE},
        'currency': '{CURRENCY_CODE}',
        'items': [
            {
                'item_id': '{ITEM_SKU}',
                'item_name': '{ITEM_NAME}',
                'price': {ITEM_PRICE},
                'quantity': {ITEM_QUANTITY},
                'item_category': '{ITEM_CATEGORY}'
            }
        ]
    }
});
```

### Real Example - DataLayer E-commerce Integration
```javascript
// DataLayer integration for automatic e-commerce tracking
window.dataLayer = window.dataLayer || [];

// Purchase tracking via dataLayer (SecurePixel automatically processes this)
function trackPurchaseViaDataLayer(orderData) {
    dataLayer.push({
        'event': 'purchase',
        'ecommerce': {
            'transaction_id': orderData.orderId,
            'value': orderData.total,
            'currency': 'USD',
            'shipping': orderData.shipping,
            'tax': orderData.tax,
            'items': orderData.items.map(item => ({
                'item_id': item.sku,
                'item_name': item.name,
                'price': item.price,
                'quantity': item.quantity,
                'item_category': item.category,
                'item_brand': item.brand
            }))
        }
    });
}

// Add to cart via dataLayer
function trackAddToCartViaDataLayer(product) {
    dataLayer.push({
        'event': 'add_to_cart',
        'ecommerce': {
            'currency': 'USD',
            'value': product.price * product.quantity,
            'items': [{
                'item_id': product.sku,
                'item_name': product.name,
                'price': product.price,
                'quantity': product.quantity,
                'item_category': product.category
            }]
        }
    });
}
```

## Revenue Attribution and Customer Analytics

### Template - Customer Lifetime Value
```javascript
securepixel.track('customer_analytics', {
    customer_id: '{CUSTOMER_ID}',
    customer_type: '{NEW_OR_RETURNING}',
    lifetime_orders: {TOTAL_ORDERS},
    lifetime_value: {TOTAL_CLV},
    average_order_value: {AVG_ORDER_VALUE},
    days_since_first_purchase: {DAYS_SINCE_FIRST}
});
```

### Real Example - Customer Value Tracking
```javascript
// Track customer value metrics on purchase
function trackCustomerValue(customerId, orderValue) {
    // Fetch customer data from your system
    fetchCustomerData(customerId).then(function(customerData) {
        const isNewCustomer = customerData.totalOrders === 1;
        const daysSinceFirst = Math.floor((Date.now() - new Date(customerData.firstPurchaseDate)) / (1000 * 60 * 60 * 24));
        
        securepixel.track('customer_analytics', {
            customer_id: customerId,
            customer_type: isNewCustomer ? 'new' : 'returning',
            lifetime_orders: customerData.totalOrders,
            lifetime_value: customerData.totalSpent,
            average_order_value: customerData.totalSpent / customerData.totalOrders,
            days_since_first_purchase: daysSinceFirst,
            current_order_value: orderValue
        });
    });
}
```

## Conversion Funnel Tracking

### Template - Funnel Step Tracking
```javascript
securepixel.track('funnel_step', {
    funnel_name: '{FUNNEL_NAME}',
    step_number: {STEP_NUMBER},
    step_name: '{STEP_DESCRIPTION}',
    products_in_scope: {PRODUCT_COUNT},
    cart_value: {CURRENT_CART_VALUE}
});
```

### Real Example - E-commerce Conversion Funnel
```javascript
// Track e-commerce conversion funnel
class EcommerceFunnel {
    constructor() {
        this.funnelName = 'product_purchase';
        this.currentStep = 0;
    }
    
    trackStep(stepName, additionalData = {}) {
        this.currentStep++;
        securepixel.track('funnel_step', {
            funnel_name: this.funnelName,
            step_number: this.currentStep,
            step_name: stepName,
            ...additionalData
        });
    }
}

const funnel = new EcommerceFunnel();

// Step 1: Product view
funnel.trackStep('product_view', {
    products_in_scope: 1,
    cart_value: 0
});

// Step 2: Add to cart
document.querySelector('.add-to-cart').addEventListener('click', function() {
    funnel.trackStep('add_to_cart', {
        products_in_scope: 1,
        cart_value: calculateCartTotal()
    });
});

// Step 3: Begin checkout
document.querySelector('.checkout-btn').addEventListener('click', function() {
    funnel.trackStep('begin_checkout', {
        products_in_scope: getCartItemCount(),
        cart_value: calculateCartTotal()
    });
});

// Step 4: Purchase complete (on confirmation page)
if (window.location.pathname.includes('/order-confirmation')) {
    funnel.trackStep('purchase_complete', {
        products_in_scope: getOrderItemCount(),
        cart_value: getOrderTotal()
    });
}
```

## Best Practices for E-commerce Tracking

### 1. Data Accuracy
- Ensure SKUs match your product catalog exactly
- Use consistent currency codes throughout
- Validate numeric values (prices, quantities)

### 2. Attribution
- Track customer acquisition sources
- Maintain session continuity across checkout
- Link purchases to marketing campaigns

### 3. Performance
- Batch similar events when possible
- Avoid tracking every price hover or quick interaction
- Use efficient product data lookup methods

### 4. Privacy Compliance
- Don't track payment information or personal details
- Respect user consent for analytics cookies
- Use hashed customer IDs when possible

## Debugging E-commerce Tracking

### Debug Mode
```javascript
// Enable debug mode to see all e-commerce events
securepixel.debug(true);

// Check specific tracking methods are working
console.log('SecurePixel methods available:', typeof securepixel.trackPurchase);
```

### Verify Purchase Tracking
```javascript
// Test purchase tracking with sample data
securepixel.trackPurchase({
    order_id: 'TEST-' + Date.now(),
    revenue: 99.99,
    currency: 'USD',
    products: [{
        sku: 'TEST-SKU',
        name: 'Test Product',
        price: 99.99,
        quantity: 1
    }]
});
```

### Common Issues
- **Products not tracking**: Verify SKU format and required fields
- **Revenue attribution problems**: Check currency consistency and numeric formatting
- **Missing cart events**: Ensure event handlers are attached after DOM load
- **DataLayer not processing**: Verify dataLayer is initialized before SecurePixel loads