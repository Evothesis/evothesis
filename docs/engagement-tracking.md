# Engagement Tracking Guide

Track user interactions, page views, clicks, and engagement metrics with SecurePixel.

## Automatic Tracking

SecurePixel automatically tracks these engagement events without any additional code:

- **Page Views**: Every page load and navigation
- **Click Events**: All clickable elements
- **Scroll Depth**: User scroll behavior (tracked in 10% increments)
- **Session Data**: Visitor and session identification
- **Form Focus**: When users interact with form fields

## Custom Event Tracking

### Template - Basic Event Tracking
```javascript
securepixel.track('{EVENT_TYPE}', {
    {PROPERTY_NAME}: '{PROPERTY_VALUE}',
    {NUMERIC_PROPERTY}: {NUMERIC_VALUE},
    {BOOLEAN_PROPERTY}: {TRUE_OR_FALSE}
});
```

### Real Example - Video Engagement
```javascript
// Track video play events
document.getElementById('product-demo-video').addEventListener('play', function() {
    securepixel.track('video_play', {
        video_title: 'Product Demo 2024',
        video_duration: 120,
        video_position: 'homepage_hero',
        user_type: 'anonymous'
    });
});

// Track video completion
document.getElementById('product-demo-video').addEventListener('ended', function() {
    securepixel.track('video_complete', {
        video_title: 'Product Demo 2024',
        completion_rate: 100,
        watch_time: 120
    });
});
```

## Button and Link Tracking

### Template - Button Click Tracking
```javascript
document.getElementById('{BUTTON_ID}').addEventListener('click', function() {
    securepixel.track('{EVENT_NAME}', {
        button_text: '{BUTTON_TEXT}',
        button_location: '{LOCATION_DESCRIPTION}',
        page_section: '{SECTION_NAME}'
    });
});
```

### Real Example - CTA Button Tracking
```javascript
// Track "Get Started" button clicks
document.getElementById('get-started-btn').addEventListener('click', function() {
    securepixel.track('cta_click', {
        button_text: 'Get Started Free',
        button_location: 'header_navigation',
        page_section: 'hero',
        destination_url: '/signup'
    });
});

// Track pricing page visits from CTA
document.querySelectorAll('.pricing-cta').forEach(function(button) {
    button.addEventListener('click', function() {
        securepixel.track('pricing_cta_click', {
            button_text: this.textContent.trim(),
            button_location: 'pricing_section',
            plan_name: this.dataset.plan,
            price: this.dataset.price
        });
    });
});
```

## Content Engagement Tracking

### Template - Content Interaction
```javascript
// Track when users engage with specific content
securepixel.track('content_engagement', {
    content_type: '{CONTENT_TYPE}',
    content_title: '{CONTENT_TITLE}',
    engagement_type: '{INTERACTION_TYPE}',
    time_spent: {SECONDS_SPENT}
});
```

### Real Example - Blog Post Engagement
```javascript
// Track blog post reading time
let startTime = Date.now();
let hasScrolledPastTitle = false;

window.addEventListener('scroll', function() {
    if (window.scrollY > 300 && !hasScrolledPastTitle) {
        hasScrolledPastTitle = true;
        securepixel.track('content_engagement', {
            content_type: 'blog_post',
            content_title: document.title,
            engagement_type: 'started_reading',
            scroll_depth: Math.round((window.scrollY / document.body.scrollHeight) * 100)
        });
    }
});

// Track when user leaves page (reading completion)
window.addEventListener('beforeunload', function() {
    if (hasScrolledPastTitle) {
        let timeSpent = Math.round((Date.now() - startTime) / 1000);
        securepixel.track('content_engagement', {
            content_type: 'blog_post',
            content_title: document.title,
            engagement_type: 'reading_complete',
            time_spent: timeSpent,
            final_scroll_depth: Math.round((window.scrollY / document.body.scrollHeight) * 100)
        });
    }
});
```

## Search and Filter Tracking

### Template - Search Events
```javascript
securepixel.track('search', {
    search_term: '{SEARCH_QUERY}',
    search_results_count: {RESULTS_COUNT},
    search_category: '{CATEGORY_NAME}',
    search_location: '{SEARCH_BOX_LOCATION}'
});
```

### Real Example - Site Search Tracking
```javascript
// Track site search usage
document.getElementById('search-form').addEventListener('submit', function(e) {
    let searchTerm = document.getElementById('search-input').value;
    let resultsCount = document.querySelectorAll('.search-result').length;
    
    securepixel.track('search', {
        search_term: searchTerm,
        search_results_count: resultsCount,
        search_category: 'site_search',
        search_location: 'header'
    });
});

// Track filter usage on product pages
document.querySelectorAll('.filter-checkbox').forEach(function(checkbox) {
    checkbox.addEventListener('change', function() {
        securepixel.track('filter_applied', {
            filter_type: this.dataset.filterType,
            filter_value: this.value,
            filter_state: this.checked,
            results_updated: document.querySelectorAll('.product-item').length
        });
    });
});
```

## Download and Resource Tracking

### Template - File Download Tracking
```javascript
securepixel.track('file_download', {
    file_name: '{FILE_NAME}',
    file_type: '{FILE_EXTENSION}',
    file_size: '{FILE_SIZE_MB}',
    download_source: '{DOWNLOAD_LOCATION}'
});
```

### Real Example - Resource Download Tracking
```javascript
// Track whitepaper and resource downloads
document.querySelectorAll('a[href$=".pdf"], a[href$=".zip"], a[href$=".docx"]').forEach(function(link) {
    link.addEventListener('click', function(e) {
        let fileName = this.href.split('/').pop();
        let fileExtension = fileName.split('.').pop();
        
        securepixel.track('file_download', {
            file_name: fileName,
            file_type: fileExtension,
            file_size: this.dataset.fileSize || 'unknown',
            download_source: this.closest('.download-section')?.dataset.section || 'page_content',
            requires_email: this.dataset.gated === 'true'
        });
    });
});
```

## Social and Sharing Tracking

### Template - Social Interaction
```javascript
securepixel.track('social_interaction', {
    social_platform: '{PLATFORM_NAME}',
    interaction_type: '{SHARE_OR_FOLLOW}',
    content_shared: '{CONTENT_TITLE}',
    share_location: '{BUTTON_LOCATION}'
});
```

### Real Example - Social Media Tracking
```javascript
// Track social media shares
document.querySelectorAll('.social-share-btn').forEach(function(button) {
    button.addEventListener('click', function() {
        securepixel.track('social_interaction', {
            social_platform: this.dataset.platform,
            interaction_type: 'share',
            content_shared: document.title,
            share_location: 'article_bottom',
            content_type: 'blog_post'
        });
    });
});

// Track newsletter signup
document.getElementById('newsletter-form').addEventListener('submit', function(e) {
    securepixel.track('social_interaction', {
        social_platform: 'email',
        interaction_type: 'newsletter_signup',
        signup_location: 'footer',
        email_provided: true
    });
});
```

## User Journey Tracking

### Template - Navigation Events
```javascript
securepixel.track('navigation', {
    from_page: '{PREVIOUS_PAGE}',
    to_page: '{CURRENT_PAGE}',
    navigation_type: '{CLICK_OR_DIRECT}',
    session_page_number: {PAGE_NUMBER_IN_SESSION}
});
```

### Real Example - User Journey Tracking
```javascript
// Track navigation between key pages
let sessionPageCount = parseInt(sessionStorage.getItem('page_count') || '0') + 1;
sessionStorage.setItem('page_count', sessionPageCount.toString());

securepixel.track('navigation', {
    from_page: document.referrer || 'direct',
    to_page: window.location.pathname,
    navigation_type: document.referrer ? 'referral' : 'direct',
    session_page_number: sessionPageCount,
    time_since_session_start: Date.now() - parseInt(sessionStorage.getItem('session_start') || Date.now())
});

// Track exit intent
document.addEventListener('mouseleave', function(e) {
    if (e.clientY < 0) {
        securepixel.track('exit_intent', {
            time_on_page: Math.round((Date.now() - performance.timing.navigationStart) / 1000),
            scroll_depth: Math.round((window.scrollY / document.body.scrollHeight) * 100),
            page_section_visible: getCurrentVisibleSection()
        });
    }
});

function getCurrentVisibleSection() {
    // Helper function to determine which page section is currently visible
    let sections = ['hero', 'features', 'pricing', 'testimonials', 'footer'];
    for (let section of sections) {
        let element = document.querySelector(`#${section}, .${section}`);
        if (element && isElementInViewport(element)) {
            return section;
        }
    }
    return 'unknown';
}

function isElementInViewport(el) {
    let rect = el.getBoundingClientRect();
    return rect.top >= 0 && rect.bottom <= window.innerHeight;
}
```

## Performance and Error Tracking

### Template - Performance Events
```javascript
securepixel.track('performance', {
    metric_name: '{METRIC_NAME}',
    metric_value: {NUMERIC_VALUE},
    page_load_time: {MILLISECONDS},
    connection_type: '{CONNECTION_SPEED}'
});
```

### Real Example - Page Performance Tracking
```javascript
// Track page load performance
window.addEventListener('load', function() {
    let timing = performance.timing;
    let loadTime = timing.loadEventEnd - timing.navigationStart;
    
    securepixel.track('performance', {
        metric_name: 'page_load',
        metric_value: loadTime,
        page_load_time: loadTime,
        connection_type: navigator.connection?.effectiveType || 'unknown',
        page_size: document.documentElement.innerHTML.length
    });
});

// Track JavaScript errors
window.addEventListener('error', function(e) {
    securepixel.track('javascript_error', {
        error_message: e.message,
        error_source: e.filename,
        error_line: e.lineno,
        page_url: window.location.href,
        user_agent: navigator.userAgent
    });
});
```

## Best Practices

### 1. Event Naming Convention
Use consistent, descriptive event names:
- `button_click` instead of `click`
- `video_play` instead of `play`
- `form_submit` instead of `submit`

### 2. Property Consistency
Maintain consistent property names across similar events:
- Always use `button_location` for button position
- Always use `content_type` for content classification
- Always use `time_spent` for duration measurements

### 3. Data Quality
- Validate data before sending
- Use meaningful, searchable property values
- Include context (page, section, user state)

### 4. Performance Considerations
- Batch events when possible
- Avoid tracking every scroll or mouse movement
- Use debouncing for high-frequency events

## Debug Mode

Enable debug mode to see tracking events in the browser console:

```javascript
securepixel.debug(true);
```

This will log all events being sent, helping you verify your tracking implementation.