# Privacy and Compliance Guide

Configure SecurePixel for GDPR, HIPAA, and other privacy compliance requirements with automatic data filtering and consent management.

## Privacy Levels Overview

SecurePixel supports three privacy levels configured at the client level:

- **Standard**: Full data collection with basic privacy protections
- **GDPR**: Enhanced privacy filtering and consent-based tracking
- **HIPAA**: Maximum privacy protection with comprehensive data filtering

## GDPR Compliance

### Automatic Data Filtering

**Template - GDPR Configuration**:
```javascript
// GDPR clients automatically receive filtered data
// No additional configuration required - handled server-side
{
  "privacy_level": "gdpr",
  "ip_collection": {
    "enabled": true,
    "hash_required": true,
    "salt": "{CLIENT_SPECIFIC_SALT}"
  },
  "consent": {
    "required": true,
    "default_behavior": "block"
  }
}
```

**Real Example - GDPR Client Configuration**:
```javascript
// Example of how GDPR configuration affects data collection
// This configuration is managed in SecurePixel dashboard

// Original form data:
{
  "eventType": "form_submit",
  "eventData": {
    "email": "user@example.com",    // FILTERED OUT
    "phone": "555-123-4567",        // FILTERED OUT
    "name": "John Smith",           // FILTERED OUT
    "company": "ACME Corp",         // KEPT
    "message": "Interested in demo", // KEPT
    "budget": "50000"               // KEPT
  }
}

// Automatically filtered result:
{
  "eventType": "form_submit",
  "eventData": {
    "company": "ACME Corp",
    "message": "Interested in demo",
    "budget": "50000"
  }
}
```

### Consent Management Integration

**Template - Consent Banner Integration**:
```html
<!-- Consent banner (your choice of provider) -->
<div id="consent-banner">
    <p>We use cookies to improve your experience.</p>
    <button id="accept-all">Accept All</button>
    <button id="accept-necessary">Necessary Only</button>
</div>

<script>
document.getElementById('accept-all').addEventListener('click', function() {
    // Enable all tracking
    securepixel.track('consent_given', {
        consent_type: 'all',
        consent_categories: ['necessary', 'analytics', 'marketing']
    });
    
    // Your consent management code here
    setCookie('consent_analytics', 'true', 365);
});

document.getElementById('accept-necessary').addEventListener('click', function() {
    // Enable only necessary tracking
    securepixel.track('consent_given', {
        consent_type: 'necessary_only',
        consent_categories: ['necessary']
    });
    
    setCookie('consent_analytics', 'false', 365);
});
</script>
```

**Real Example - OneTrust Integration**:
```html
<!-- OneTrust consent integration -->
<script src="https://cdn.cookielaw.org/scripttemplates/otSDKStub.js" charset="UTF-8" data-domain-script="your-onetrust-id"></script>
<script>
function OptanonWrapper() {
    // Check if analytics cookies are allowed
    if (OnetrustActiveGroups.includes('C0002')) {
        // Analytics cookies allowed
        securepixel.track('consent_given', {
            consent_type: 'analytics_enabled',
            consent_categories: ['necessary', 'analytics'],
            consent_provider: 'onetrust',
            consent_id: OnetrustActiveGroups
        });
    } else {
        // Analytics cookies blocked
        securepixel.track('consent_given', {
            consent_type: 'analytics_blocked',
            consent_categories: ['necessary'],
            consent_provider: 'onetrust'
        });
    }
}

// Listen for consent changes
document.addEventListener('OneTrustGroupsUpdated', function() {
    if (OnetrustActiveGroups.includes('C0002')) {
        // Re-enable tracking if consent is given
        securepixel.track('consent_updated', {
            consent_type: 'analytics_enabled',
            previous_state: 'blocked'
        });
    }
});
</script>
```

## HIPAA Compliance

### Enhanced Data Protection

**Template - HIPAA Configuration**:
```javascript
// HIPAA clients receive maximum privacy protection
{
  "privacy_level": "hipaa",
  "ip_collection": {
    "enabled": false,  // IP addresses not collected
    "hash_required": true,
    "salt": "{CLIENT_SPECIFIC_SALT}"
  },
  "consent": {
    "required": true,
    "default_behavior": "block"
  },
  "pii_filtering": "aggressive"
}
```

**Real Example - Healthcare Form Tracking**:
```html
<!-- Healthcare contact form with HIPAA compliance -->
<form id="patient-inquiry-form" 
      data-securepixel-lead="true" 
      data-lead-type="patient_inquiry"
      data-privacy-level="hipaa">
    
    <input type="text" name="condition" placeholder="General inquiry topic">
    <input type="text" name="insurance" placeholder="Insurance provider">
    <input type="email" name="email" placeholder="Contact email">
    <input type="tel" name="phone" placeholder="Phone number">
    <textarea name="symptoms" placeholder="Describe your situation"></textarea>
    
    <button type="submit">Submit Inquiry</button>
</form>

<script>
// HIPAA-compliant form tracking
document.getElementById('patient-inquiry-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    // Only track non-PII data for HIPAA compliance
    securepixel.trackLead({
        form_id: 'patient-inquiry',
        lead_type: 'patient_inquiry',
        lead_value: 200,
        source: 'website',
        // Note: email, phone, symptoms automatically filtered by HIPAA privacy level
        inquiry_category: document.querySelector('[name="condition"]').value ? 'specific' : 'general',
        has_insurance: document.querySelector('[name="insurance"]').value ? true : false
    });
    
    // Continue with secure form submission
    this.submit();
});
</script>
```

## Data Retention and Deletion

### Automatic Data Lifecycle

**Template - Data Retention Configuration**:
```javascript
// Data retention policies (configured per client)
{
  "data_retention": {
    "raw_data_days": {RETENTION_DAYS},
    "processed_data_days": {PROCESSED_RETENTION_DAYS},
    "auto_deletion": {TRUE_OR_FALSE},
    "deletion_schedule": "{SCHEDULE_FREQUENCY}"
  }
}
```

**Real Example - 30-Day Data Retention**:
```javascript
// Example client configuration for short data retention
{
  "client_id": "healthcare-client-2024",
  "privacy_level": "hipaa",
  "data_retention": {
    "raw_data_days": 30,        // Delete raw data after 30 days
    "processed_data_days": 90,  // Keep processed data for 90 days
    "auto_deletion": true,      // Automatic deletion enabled
    "deletion_schedule": "daily" // Check for deletion daily
  },
  "export_schedule": "daily",   // Export data daily before deletion
  "notification_before_deletion": 7 // Notify 7 days before deletion
}
```

### Manual Data Deletion

**Template - Data Subject Rights**:
```javascript
// API call for data deletion (GDPR Article 17 - Right to be forgotten)
fetch('/api/data-deletion', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer {API_KEY}'
    },
    body: JSON.stringify({
        client_id: '{YOUR_CLIENT_ID}',
        visitor_id: '{VISITOR_ID_TO_DELETE}',
        deletion_reason: '{GDPR_ARTICLE_17}',
        requester_email: '{REQUESTER_EMAIL}'
    })
});
```

**Real Example - GDPR Deletion Request**:
```javascript
// Handle GDPR deletion request from your privacy portal
async function handleGDPRDeletion(visitorId, requesterEmail) {
    try {
        const response = await fetch('/api/data-deletion', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer sk_live_abc123def456'
            },
            body: JSON.stringify({
                client_id: 'eu-client-2024',
                visitor_id: visitorId,
                deletion_reason: 'GDPR Article 17 - Right to be forgotten',
                requester_email: requesterEmail,
                deletion_scope: 'all_personal_data',
                verification_method: 'email_confirmation'
            })
        });
        
        const result = await response.json();
        
        if (result.success) {
            // Track the deletion request
            securepixel.track('gdpr_deletion_request', {
                request_id: result.deletion_request_id,
                visitor_id: visitorId,
                estimated_completion: result.completion_date
            });
            
            return result;
        }
    } catch (error) {
        console.error('GDPR deletion request failed:', error);
    }
}
```

## Cookie Management

### Cookie-Free Tracking Option

**Template - Cookieless Configuration**:
```javascript
// Configure cookieless tracking for privacy compliance
{
  "cookie_usage": {
    "enabled": {TRUE_OR_FALSE},
    "tracking_method": "{COOKIES_OR_FINGERPRINT}",
    "session_storage_only": {TRUE_OR_FALSE}
  }
}
```

**Real Example - Cookieless E-commerce Tracking**:
```html
<!-- Cookieless tracking implementation -->
<script>
// Override default cookie behavior for privacy compliance
window.addEventListener('DOMContentLoaded', function() {
    // Check if cookies are disabled or blocked
    const cookiesEnabled = navigator.cookieEnabled && 
                          document.cookie.indexOf('test=1') !== -1;
    
    if (!cookiesEnabled) {
        // Use session storage for cookieless tracking
        securepixel.track('privacy_mode_detected', {
            tracking_method: 'session_storage',
            cookie_status: 'disabled',
            privacy_preference: 'high'
        });
    }
    
    // Track e-commerce events without persistent cookies
    document.querySelectorAll('.add-to-cart').forEach(button => {
        button.addEventListener('click', function() {
            const productData = {
                sku: this.dataset.sku,
                name: this.dataset.name,
                price: parseFloat(this.dataset.price),
                currency: 'USD'
            };
            
            // Track without setting persistent cookies
            securepixel.trackAddToCart({
                ...productData,
                tracking_mode: 'cookieless',
                session_only: true
            });
        });
    });
});
</script>
```

## Regional Compliance

### International Data Transfer

**Template - Data Localization**:
```javascript
// Configure data storage regions for compliance
{
  "data_localization": {
    "primary_region": "{EU_OR_US}",
    "allowed_regions": ["{REGION_LIST}"],
    "cross_border_transfer": {TRUE_OR_FALSE},
    "adequacy_decision": "{GDPR_ADEQUACY_STATUS}"
  }
}
```

**Real Example - EU Data Residency**:
```javascript
// EU client with strict data residency requirements
{
  "client_id": "eu-manufacturing-2024",
  "privacy_level": "gdpr",
  "data_localization": {
    "primary_region": "eu-west-1",           // EU data center
    "allowed_regions": ["eu-west-1", "eu-central-1"], // EU only
    "cross_border_transfer": false,          // No US data transfer
    "adequacy_decision": "eu_gdpr_compliant",
    "data_processor_agreement": "signed",
    "schrems_ii_compliant": true
  },
  "export_configuration": {
    "s3_bucket_region": "eu-west-1",
    "encryption_at_rest": true,
    "encryption_in_transit": true,
    "access_logging": true
  }
}
```

## Compliance Monitoring

### Privacy Audit Trail

**Template - Compliance Tracking**:
```javascript
securepixel.track('privacy_compliance_event', {
    event_type: '{COMPLIANCE_EVENT_TYPE}',
    compliance_framework: '{GDPR_OR_HIPAA}',
    data_subject_rights: '{RIGHTS_EXERCISED}',
    processing_lawful_basis: '{LAWFUL_BASIS}'
});
```

**Real Example - Consent Audit Trail**:
```javascript
// Track consent changes for compliance auditing
function trackConsentChange(consentType, previousState, newState) {
    securepixel.track('privacy_compliance_event', {
        event_type: 'consent_change',
        compliance_framework: 'gdpr',
        consent_type: consentType,
        previous_state: previousState,
        new_state: newState,
        timestamp: new Date().toISOString(),
        user_agent: navigator.userAgent,
        ip_address: 'hashed', // IP is hashed for GDPR compliance
        data_subject_rights: 'consent_withdrawal',
        processing_lawful_basis: 'consent_article_6_1_a',
        retention_period: '2_years_from_last_interaction'
    });
}

// Usage when user updates consent preferences
document.getElementById('consent-preferences').addEventListener('change', function(e) {
    if (e.target.name === 'analytics_consent') {
        trackConsentChange(
            'analytics',
            e.target.dataset.previousValue,
            e.target.checked ? 'granted' : 'denied'
        );
    }
});
```

## Data Processing Agreements

### Technical and Organizational Measures

**Template - Security Configuration**:
```javascript
// Security measures automatically implemented
{
  "security_measures": {
    "encryption_at_rest": true,
    "encryption_in_transit": true,
    "access_controls": "{RBAC_ENABLED}",
    "audit_logging": true,
    "data_pseudonymization": "{ENABLED_FOR_GDPR}",
    "backup_encryption": true
  }
}
```

**Real Example - Healthcare Security**:
```javascript
// HIPAA-compliant security configuration
{
  "client_id": "healthcare-provider-2024",
  "privacy_level": "hipaa",
  "security_measures": {
    "encryption_at_rest": true,           // AES-256 encryption
    "encryption_in_transit": true,        // TLS 1.3
    "access_controls": "rbac_enabled",    // Role-based access
    "audit_logging": true,                // All access logged
    "data_pseudonymization": true,        // PII pseudonymized
    "backup_encryption": true,            // Encrypted backups
    "access_review_frequency": "quarterly", // Regular access reviews
    "incident_response_plan": "enabled",   // Security incident procedures
    "business_associate_agreement": "signed" // HIPAA BAA in place
  },
  "compliance_certifications": [
    "SOC2_TYPE_II",
    "HIPAA_COMPLIANT",
    "ISO27001"
  ]
}
```

## Privacy Configuration Checklist

### Implementation Checklist

**For GDPR Compliance**:
- [ ] Privacy level set to 'gdpr' in SecurePixel dashboard
- [ ] Consent management system integrated
- [ ] Cookie banner with granular consent options
- [ ] Data retention policies configured (max 2 years recommended)
- [ ] Data subject rights request process implemented
- [ ] Privacy policy updated with SecurePixel as processor
- [ ] Data Processing Agreement signed with SecurePixel

**For HIPAA Compliance**:
- [ ] Privacy level set to 'hipaa' in SecurePixel dashboard
- [ ] Business Associate Agreement signed
- [ ] IP address collection disabled
- [ ] Aggressive PII filtering enabled
- [ ] Minimum necessary data principle applied
- [ ] Access controls and audit logging enabled
- [ ] Incident response procedures documented
- [ ] Staff training on PHI handling completed

**For General Privacy**:
- [ ] Privacy policy includes analytics tracking disclosure
- [ ] Users can opt-out of tracking
- [ ] Data retention period clearly communicated
- [ ] Security measures documented and implemented
- [ ] Regular privacy impact assessments conducted

## Testing Privacy Configuration

### Verify Data Filtering

```javascript
// Test that PII is being filtered correctly
securepixel.debug(true);

// Submit test data that should be filtered
securepixel.track('test_privacy_filtering', {
    email: 'test@example.com',      // Should be filtered for GDPR/HIPAA
    phone: '555-123-4567',          // Should be filtered for GDPR/HIPAA
    name: 'John Doe',               // Should be filtered for GDPR/HIPAA
    company: 'ACME Corp',           // Should be kept
    interest: 'Product demo'        // Should be kept
});

// Check network tab to verify filtered data
```

### Consent Testing

```javascript
// Test consent management integration
function testConsentFlow() {
    // Test denied consent
    securepixel.track('consent_test', {
        consent_status: 'denied',
        tracking_should_be: 'blocked'
    });
    
    // Test granted consent
    securepixel.track('consent_test', {
        consent_status: 'granted',
        tracking_should_be: 'enabled'
    });
}
```

This privacy configuration ensures SecurePixel complies with major privacy regulations while maintaining useful analytics capabilities for business intelligence and marketing attribution.