---
name: authos-compliance-automation
description: Streamline regulatory compliance (GDPR, SOC2, HIPAA) using AuthOS. Covers exporting user data for Subject Access Requests (SAR), configuring audit log streaming to SIEM providers (Datadog, Splunk), and managing data retention policies. Use this when a user needs to meet security or privacy compliance requirements.
---

# AuthOS Compliance Automation

AuthOS helps you meet strict compliance requirements (GDPR, CCPA, SOC2) through automated data management and security auditing.

## 1. Audit Log Streaming (SIEM Integration)

Fo SOC2 and security monitoring, you can stream all platform and organization events to your SIEM provider.

### Supported Providers
- **Datadog**: Native integration with custom tags.
- **Splunk**: Via HTTP Event Collector (HEC).
- **Elasticsearch**: Direct indexing.
- **Custom**: Any HTTP endpoint that accepts JSON.

### Configuration
1. **Endpoint**: `POST /api/organizations/:org_slug/siem-configs` (Organization Owner).
2. **Body**:
   ```json
   {
     "provider": "datadog",
     "endpoint_url": "https://http-intake.logs.datadoghq.com/v1/input",
     "api_key": "your-dd-api-key",
     "batch_size": "100",
     "enabled": true
   }
   ```
3. **Behavior**: AuthOS enqueues a background job to stream logs in batches using cursor-based tracking to ensure zero data loss.

## 2. GDPR/CCPA Data Portability

AuthOS provides a single endpoint to export all data associated with a specific user identity across all services.

- **Export Data**: `GET /api/privacy/export/:user_id`
- **Output**: A comprehensive JSON file containing:
  - Profile information.
  - Authentication history (Login events).
  - MFA status and change history.
  - Organization membership details.

## 3. Right to be Forgotten (Anonymization)

Instead of hard-deleting records (which breaks audit logs), AuthOS supports **Anonymization**.
- **Endpoint**: `DELETE /api/privacy/forget/:user_id`
- **Action**: 
  - Redacts `email`, `name`, and `ip_address` from all tables.
  - Replaces them with static strings (e.g., `ANONYMIZED_USER`).
  - Retains the unique ID to maintain the integrity of audit logs for compliance.

## 4. Security Alerts and Risk Assessment

Automate responses to suspicious activity.
- **Impossible Travel**: AuthOS alerts when a user logs in from two distant geographic locations in a short time window.
- **Brute Force Detection**: Automatic rate-limiting and temporary account lockout.
- **Check Alerts**: `GET /api/platform/mfa/suspicious` or via Webhooks (`user.login.failed` with high `risk_score` in data).

## Best Practices
- **Stream logs to a secure location** (like an S3 bucket or cold storage) for the 1-7 year retention periods required by many regulations.
- **Automate SAR requests** by integrating the Export API into your support portal.
- **Configure SIEM alerts** specifically for `promote_platform_owner` and `org.delete` events.
