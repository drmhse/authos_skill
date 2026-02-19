---
name: authos-service-management
description: Architect and manage services within an organization. Includes creating new services (web, mobile, api), managing client credentials, defining pricing plans, and monitoring service-specific usage. Use this when a user is building multiple applications on top of the same AuthOS organization.
---

# AuthOS Service Management

This skill covers how to manage "Services" in AuthOS. A Service represents an application (client) that uses AuthOS for authentication.

## 1. Creating a Service

Services are the primary way to integrate AuthOS into your apps.

- **Endpoint**: `POST /api/organizations/:org_slug/services`
- **Service Types**:
  - `web`: For browser-based apps using the SDK.
  - `api`: For backend-to-backend communication (API Keys).
  - `mobile`: For iOS/Android apps.
  - `desktop`: For native desktop applications.

### Request Body
```json
{
  "slug": "main-app",
  "name": "Acme Main Application",
  "service_type": "web",
  "redirect_uris": ["https://app.acme.com/callback"],
  "google_scopes": ["openid", "email", "profile"]
}
```

### Response
Creating a service returns a `client_id` and a `client_secret`. **Store the `client_secret` immediately** as it is hashed in the database and cannot be retrieved later.

## 2. Managing Client Credentials

If a secret is lost or compromised, you must roll it.
- **Regenerate Secret**: (This typically involves deleting and recreating the service or using a specialized admin endpoint if available).
- **Security Note**: AuthOS hashes `client_secret` using SHA-256 before storage.

## 3. Defining Pricing Plans

Each service can have multiple plans (e.g., Free, Pro, Enterprise). This allows you to restrict user access based on their subcription status.

### Create a Plan
- **Endpoint**: `POST /api/organizations/:org_slug/services/:service_slug/plans`
- **Body**:
  ```json
  {
    "name": "Standard Plan",
    "description": "Up to 1000 requests/mo",
    "price_cents": 1900,
    "currency": "usd",
    "features": ["analytics", "advanced-matching"],
    "is_default": true
  }
  ```

## 4. API Keys for Service-to-Service

For `api` type services, you can generate long-lived API keys instead of using the OAuth flow.

- **List Keys**: `GET /api/organizations/:org_slug/services/:service_slug/api-keys`
- **Create Key**: `POST /api/organizations/:org_slug/services/:service_slug/api-keys`
  - Body: `{"name": "Internal Scraper"}`

## 5. Usage and Limits

Organization tiers limit the number of services you can create.
- **Default Limit**: (Varies by tier, typically 3 for Free).
- **Check Usage**: `GET /api/organizations/:org_slug/services` returns a `usage` object with `current_services` and `max_services`.

## Best Practices
- **Use meaningful slugs**. Slugs are used in SDK initialization and login URLs.
- **Limit Redirect URIs**. Only allow URIs that you control to prevent authorization code theft.
- **Keep scope requests minimal**. Only request the permissions your app actually needs from Google/GitHub/etc.
