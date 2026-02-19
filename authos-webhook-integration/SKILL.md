---
name: authos-webhook-integration
description: Build real-time, event-driven integrations with AuthOS webhooks. Includes common event types (user signup, login, organization settings changes), payload structure, security verification using HMAC signatures, and retry logic. Use this when a user needs to sync data with their own backend.
---

# AuthOS Webhook Integration

Webhooks allow your application to receive real-time notifications when certain events happen in AuthOS.

## 1. Supported Event Types

| Category | Event Types |
|----------|-------------|
| **User** | `user.signup.success`, `user.login.success`, `user.login.failed`, `user.logout` |
| **Organization** | `organization.updated`, `organization.smtp.configured`, `user.invited` |
| **Service** | `service.created`, `service.updated`, `service.deleted` |
| **Security** | `security.mfa.enabled`, `security.mfa.disabled`, `security.password.changed` |
| **Billing** | `subscription.created`, `subscription.updated`, `subscription.canceled` |

## 2. Configuring a Webhook

You can configure webhooks via the Organization Settings API.
- **Endpoint**: `POST /api/organizations/:org_slug/webhooks`
- **Body**:
  ```json
  {
    "url": "https://your-api.com/webhooks/authos",
    "events": ["user.signup.success", "user.login.success"],
    "description": "Production sync webhook"
  }
  ```
- **Response**: returns a `secret`. **Save this secret** to verify incoming requests.

## 3. Webhook Payload Structure

All webhooks follow a consistent JSON format:

```json
{
  "event": "user.signup.success",
  "timestamp": "2025-05-20T14:30:00Z",
  "organization_id": "org_123",
  "actor_user_id": "user_456",
  "actor_email": "newuser@example.com",
  "target_type": "user",
  "target_id": "user_456",
  "data": {
    "provider": "github",
    "ip_address": "1.2.3.4"
  }
}
```

## 4. Verifying Signatures (Security)

AuthOS signs every webhook request with an HMAC-SHA256 signature using your webhook's secret.

### Verification Steps
1. Retrieve the `X-AuthOS-Signature` header from the incoming request.
2. Compute the HMAC-SHA256 signature of the raw request body using your webhook secret.
3. Compare your computed signature with the header value using a constant-time comparison.

**Example (Node.js)**:
```javascript
const crypto = require('crypto');

function verify(secret, body, signature) {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(body);
  const digest = hmac.digest('hex');
  return crypto.timingSafeEqual(Buffer.from(digest), Buffer.from(signature));
}
```

## 5. Retry Logic and Delivery

- **Success**: Your server must return a `2xx` status code within 5 seconds.
- **Retries**: If your server returns anything else (or times out), AuthOS will retry with exponential backoff:
  - 1st retry: 1 minute later
  - 2nd retry: 5 minutes later
  - 3rd retry: 1 hour later
- **Disabling**: If a webhook fails consistently for 48 hours, it will be automatically disabled.

## Best Practices
- **Respond quickly**: Do not perform heavy processing inside the webhook handler. Enqueue a background job in your system and return `200 OK` immediately.
- **Idempotency**: Always check the `event_id` (if provided) or timestamp to ensure you don't process the same event twice.
- **Use HTTPS**: AuthOS only delivers webhooks to secure `https` URLs in production.
