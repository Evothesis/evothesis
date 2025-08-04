# Form and Lead Tracking Guide

Track lead forms, contact forms, and conversion events with SecurePixel's comprehensive form tracking system.

## Automatic Form Detection

SecurePixel automatically detects and tracks forms that contain these indicators:
- Form IDs: `contact`, `lead`, `quote`, `demo`, `trial`, `signup`, `subscribe`
- Form classes: `lead-form`, `contact-form`
- Data attributes: `data-securepixel-lead="true"`

## Basic Lead Form Setup

### Template - Enhanced Form HTML
```html
<form id="{FORM_ID}" 
      data-securepixel-lead="true" 
      data-lead-type="{LEAD_TYPE}" 
      data-lead-value="{ESTIMATED_VALUE}">
    
    <input type="text" name="{FIELD_NAME}" required>
    <input type="email" name="{EMAIL_FIELD}" required>
    <button type="submit">{SUBMIT_BUTTON_TEXT}</button>
</form>
```

### Real Example - Contact Form
```html
<form id="contact-form" 
      data-securepixel-lead="true" 
      data-lead-type="contact" 
      data-lead-value="250">
    
    <input type="text" name="name" placeholder="Your Name" required>
    <input type="email" name="email" placeholder="Your Email" required>
    <input type="text" name="company" placeholder="Company Name">
    <textarea name="message" placeholder="Your Message" required></textarea>
    <button type="submit">Send Message</button>
</form>
```

## Manual Lead Tracking

### Template - Manual Lead Event
```javascript
securepixel.trackLead({
    form_id: '{FORM_IDENTIFIER}',
    lead_type: '{LEAD_TYPE}',
    lead_value: {ESTIMATED_VALUE},
    source: '{TRAFFIC_SOURCE}',
    campaign: '{CAMPAIGN_NAME}'
});
```

### Real Example - Demo Request Tracking
```javascript
// Track demo request form submission
document.getElementById('demo-request-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    // Track the lead
    securepixel.trackLead({
        form_id: 'demo-request',
        lead_type: 'demo',
        lead_value: 500,
        source: 'website',
        campaign: 'q4-product-launch'
    });
    
    // Continue with form submission
    this.submit();
});
```

## Advanced Form Tracking

### Template - Multi-Step Form Tracking
```javascript
// Track form progress through multiple steps
securepixel.track('form_progress', {
    form_id: '{FORM_ID}',
    step_number: {CURRENT_STEP},
    step_name: '{STEP_DESCRIPTION}',
    total_steps: {TOTAL_STEPS},
    completion_rate: {PERCENTAGE_COMPLETE}
});
```

### Real Example - Multi-Step Lead Form
```javascript
// Track progress through a multi-step lead qualification form
let currentStep = 1;
const totalSteps = 4;

function trackFormProgress(stepName) {
    securepixel.track('form_progress', {
        form_id: 'lead-qualification',
        step_number: currentStep,
        step_name: stepName,
        total_steps: totalSteps,
        completion_rate: Math.round((currentStep / totalSteps) * 100)
    });
}

// Track each step
document.getElementById('step1-next').addEventListener('click', function() {
    trackFormProgress('company_info');
    currentStep = 2;
});

document.getElementById('step2-next').addEventListener('click', function() {
    trackFormProgress('contact_details');
    currentStep = 3;
});

document.getElementById('step3-next').addEventListener('click', function() {
    trackFormProgress('requirements');
    currentStep = 4;
});

// Track final submission
document.getElementById('qualification-form').addEventListener('submit', function() {
    trackFormProgress('completed');
    
    securepixel.trackLead({
        form_id: 'lead-qualification',
        lead_type: 'qualified_lead',
        lead_value: 1000,
        source: 'website',
        qualified: true
    });
});
```

## Form Field Interaction Tracking

### Template - Field Focus and Abandonment
```javascript
// Track when users interact with specific form fields
document.querySelector('input[name="{FIELD_NAME}"]').addEventListener('focus', function() {
    securepixel.track('form_field_focus', {
        form_id: '{FORM_ID}',
        field_name: '{FIELD_NAME}',
        field_type: '{INPUT_TYPE}',
        form_step: '{CURRENT_STEP}'
    });
});
```

### Real Example - Newsletter Signup Analysis
```javascript
// Track newsletter signup form interactions
const newsletterForm = document.getElementById('newsletter-form');
const emailField = newsletterForm.querySelector('input[name="email"]');
let emailFieldFocused = false;
let emailFieldCompleted = false;

emailField.addEventListener('focus', function() {
    if (!emailFieldFocused) {
        emailFieldFocused = true;
        securepixel.track('form_field_focus', {
            form_id: 'newsletter-signup',
            field_name: 'email',
            field_type: 'email',
            form_location: 'footer'
        });
    }
});

emailField.addEventListener('blur', function() {
    if (this.value && !emailFieldCompleted) {
        emailFieldCompleted = true;
        securepixel.track('form_field_complete', {
            form_id: 'newsletter-signup',
            field_name: 'email',
            field_completed: true,
            email_format_valid: this.checkValidity()
        });
    }
});

// Track form abandonment
window.addEventListener('beforeunload', function() {
    if (emailFieldFocused && !emailFieldCompleted) {
        securepixel.track('form_abandonment', {
            form_id: 'newsletter-signup',
            fields_touched: ['email'],
            abandonment_point: 'email_field',
            time_spent: Date.now() - formStartTime
        });
    }
});
```

## Lead Qualification and Scoring

### Template - Lead Scoring
```javascript
securepixel.trackLead({
    form_id: '{FORM_ID}',
    lead_type: '{LEAD_TYPE}',
    lead_value: {CALCULATED_VALUE},
    lead_score: {QUALIFICATION_SCORE},
    qualification_criteria: {
        {CRITERIA_NAME}: {CRITERIA_VALUE},
        {SCORING_FACTOR}: {FACTOR_VALUE}
    }
});
```

### Real Example - Enterprise Lead Scoring
```javascript
// Calculate and track lead score based on form responses
document.getElementById('enterprise-contact-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    let leadScore = 0;
    let leadValue = 100; // Base value
    
    // Score based on company size
    const companySize = document.querySelector('select[name="company_size"]').value;
    switch(companySize) {
        case '1-10': leadScore += 10; leadValue = 250; break;
        case '11-50': leadScore += 25; leadValue = 500; break;
        case '51-200': leadScore += 50; leadValue = 1500; break;
        case '200+': leadScore += 100; leadValue = 5000; break;
    }
    
    // Score based on budget
    const budget = document.querySelector('select[name="budget"]').value;
    if (budget === '10k+') leadScore += 50;
    if (budget === '50k+') leadScore += 75;
    
    // Score based on urgency
    const timeframe = document.querySelector('select[name="timeframe"]').value;
    if (timeframe === 'immediate') leadScore += 40;
    if (timeframe === '1-3-months') leadScore += 25;
    
    securepixel.trackLead({
        form_id: 'enterprise-contact',
        lead_type: 'enterprise_inquiry',
        lead_value: leadValue,
        lead_score: leadScore,
        qualification_criteria: {
            company_size: companySize,
            budget: budget,
            timeframe: timeframe,
            industry: document.querySelector('select[name="industry"]').value
        }
    });
    
    this.submit();
});
```

## Download and Content Gate Tracking

### Template - Gated Content
```javascript
securepixel.trackLead({
    form_id: '{FORM_ID}',
    lead_type: 'content_download',
    content_title: '{CONTENT_NAME}',
    content_type: '{CONTENT_TYPE}',
    download_value: {CONTENT_VALUE}
});
```

### Real Example - Whitepaper Download
```javascript
// Track whitepaper download form
document.getElementById('whitepaper-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    const whitepaperTitle = this.dataset.contentTitle;
    const userEmail = this.querySelector('input[name="email"]').value;
    const userCompany = this.querySelector('input[name="company"]').value;
    
    securepixel.trackLead({
        form_id: 'whitepaper-download',
        lead_type: 'content_download',
        content_title: whitepaperTitle,
        content_type: 'whitepaper',
        download_value: 150,
        lead_metadata: {
            user_email: userEmail,
            user_company: userCompany,
            content_category: 'product_guide'
        }
    });
    
    // Process download
    window.location.href = this.dataset.downloadUrl;
});
```

## Newsletter and Subscription Tracking

### Template - Subscription Events
```javascript
securepixel.trackSubscription({
    subscription_type: '{SUBSCRIPTION_TYPE}',
    list_name: '{EMAIL_LIST}',
    signup_source: '{SIGNUP_LOCATION}',
    double_optin: {TRUE_OR_FALSE}
});
```

### Real Example - Email List Signup
```javascript
// Track different newsletter signups
document.querySelectorAll('.newsletter-signup-form').forEach(function(form) {
    form.addEventListener('submit', function(e) {
        e.preventDefault();
        
        const listType = this.dataset.listType || 'general';
        const signupLocation = this.dataset.location || 'unknown';
        
        securepixel.trackSubscription({
            subscription_type: 'email_newsletter',
            list_name: listType,
            signup_source: signupLocation,
            double_optin: true
        });
        
        // Also track as a lead for marketing qualified leads
        if (listType === 'product_updates') {
            securepixel.trackLead({
                form_id: 'newsletter-' + signupLocation,
                lead_type: 'newsletter_signup',
                lead_value: 25,
                source: 'website'
            });
        }
        
        this.submit();
    });
});
```

## Form Validation and Error Tracking

### Template - Form Error Tracking
```javascript
securepixel.track('form_validation_error', {
    form_id: '{FORM_ID}',
    field_name: '{FIELD_WITH_ERROR}',
    error_type: '{VALIDATION_ERROR_TYPE}',
    error_message: '{ERROR_MESSAGE}'
});
```

### Real Example - Contact Form Validation
```javascript
// Track form validation errors to improve user experience
document.getElementById('contact-form').addEventListener('submit', function(e) {
    const requiredFields = this.querySelectorAll('[required]');
    let hasErrors = false;
    
    requiredFields.forEach(function(field) {
        if (!field.value.trim()) {
            hasErrors = true;
            securepixel.track('form_validation_error', {
                form_id: 'contact-form',
                field_name: field.name,
                error_type: 'required_field_empty',
                error_message: 'Required field left blank'
            });
        }
        
        if (field.type === 'email' && field.value && !field.checkValidity()) {
            hasErrors = true;
            securepixel.track('form_validation_error', {
                form_id: 'contact-form',
                field_name: field.name,
                error_type: 'invalid_email_format',
                error_message: 'Invalid email format'
            });
        }
    });
    
    if (hasErrors) {
        e.preventDefault();
        securepixel.track('form_submission_blocked', {
            form_id: 'contact-form',
            error_count: document.querySelectorAll('.error').length,
            fields_with_errors: Array.from(document.querySelectorAll('.error')).map(el => el.name)
        });
    }
});
```

## A/B Testing Form Variations

### Template - Form Variant Tracking
```javascript
securepixel.track('form_variant_view', {
    form_id: '{FORM_ID}',
    variant_name: '{VARIANT_IDENTIFIER}',
    test_name: '{AB_TEST_NAME}',
    user_segment: '{USER_SEGMENT}'
});
```

### Real Example - Contact Form A/B Test
```javascript
// Track different form layouts and field configurations
const formVariant = localStorage.getItem('contact_form_variant') || 'control';

// Track variant exposure
securepixel.track('form_variant_view', {
    form_id: 'contact-form',
    variant_name: formVariant,
    test_name: 'contact_form_fields_test',
    user_segment: 'website_visitors'
});

// Track variant performance
document.getElementById('contact-form').addEventListener('submit', function() {
    securepixel.track('form_variant_conversion', {
        form_id: 'contact-form',
        variant_name: formVariant,
        test_name: 'contact_form_fields_test',
        conversion_type: 'form_submission'
    });
    
    securepixel.trackLead({
        form_id: 'contact-form',
        lead_type: 'contact',
        lead_value: 200,
        ab_test_variant: formVariant
    });
});
```

## Best Practices for Form Tracking

### 1. Privacy Compliance
- Never track sensitive information (passwords, SSNs, payment details)
- Use data attributes to control tracking: `data-securepixel-ignore="true"`
- Respect user consent preferences

### 2. Lead Qualification
- Assign appropriate lead values based on business impact
- Use consistent lead types across your organization
- Include qualification criteria for sales team context

### 3. Form Optimization
- Track field-level interactions to identify friction points
- Monitor form abandonment to improve conversion rates
- A/B test form layouts and track performance differences

### 4. Data Quality
- Validate form data before tracking
- Use consistent naming conventions
- Include relevant context (page, campaign, source)

## Troubleshooting Form Tracking

### Forms Not Being Tracked?
1. Check if form has proper ID or data attributes
2. Ensure SecurePixel script loads before form interaction
3. Verify no JavaScript errors in console

### Lead Values Not Appearing?
1. Confirm `data-lead-value` is set on form element
2. Check that lead tracking is called with proper parameters
3. Verify numeric values are not wrapped in quotes

### Debug Form Events
```javascript
// Enable debug mode to see all form events
securepixel.debug(true);

// Check form detection
console.log('Forms detected:', document.querySelectorAll('[data-securepixel-lead="true"]').length);
```