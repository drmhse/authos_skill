---
name: authos-identity-configuration
description: Configure authentication providers for a tenant organization. Includes setting up Bring Your Own OAuth (BYOO) for GitHub, Google, and Microsoft, and configuring Enterprise SSO via SAML IdP. Use this when a user wants to customize how their application end-users authenticate.
---

# AuthOS Identity Configuration

This skill guides you through configuring authentication methods for your organization's end-users. These actions require **Organization Owner** or **Admin** privileges.

## 1. Bring Your Own OAuth (BYOO)

BYOO allows you to use your own OAuth applications (GitHub, Google, Microsoft) so that end-users see your brand during the login process instead of AuthOS.

### Setting Up Credentials
- **Endpoint**: `POST /api/organizations/:org_slug/oauth-credentials/:provider`
- **Path Parameters**:
  - `org_slug`: Your organization's unique slug.
  - `provider`: `github`, `google`, or `microsoft`.
- **Body**:
  ```json
  {
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET"
  }
  ```
- **Note**: Secrets are encrypted at rest using the system-wide `ENCRYPTION_KEY`.

### Provider Configuration
Ensure your OAuth application on the provider side has the correct **Redirect URI**:
`https://<auth-os-api-domain>/auth/:provider/callback`

## 2. Organization SMTP (Transactional Email)

By default, AuthOS sends emails (verification, pass-reset) using platform-level SMTP. You can override this to use your own domain.

### Configuration
- **Endpoint**: `POST /api/organizations/:org_slug/smtp`
- **Body**:
  ```json
  {
    "host": "smtp.mailgun.org",
    "port": 587,
    "username": "postmaster@mg.yourdomain.com",
    "password": "your-smtp-password",
    "from_email": "auth@yourdomain.com",
    "from_name": "Your App Name"
  }
  ```

## 3. Custom Branding

Customize the look and feel of the login pages hosted by AuthOS for your organization.

### Configuration
- **Endpoint**: `PATCH /api/organizations/:org_slug/branding`
- **Body**:
  ```json
  {
    "logo_url": "https://cdn.yourdomain.com/logo.png",
    "primary_color": "#4F46E5"
  }
  ```

## 4. Enterprise SSO (SAML IdP)

AuthOS can act as a SAML Identity Provider, allowing your users to log into other enterprise apps using their AuthOS credentials.

### Configuration
- **Endpoint**: `POST /api/organizations/:org_slug/services/:service_slug/saml`
- **Body**:
  ```json
  {
    "enabled": true,
    "entity_id": "https://sp.enterprise-app.com",
    "acs_url": "https://sp.enterprise-app.com/saml/acs",
    "slo_url": "https://sp.enterprise-app.com/saml/slo",
    "attribute_mapping": {
      "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
      "name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"
    }
  }
  ```

### Metadata Export
To complete the integration, the Service Provider (SP) will need your IDP metadata.
- **Metadata URL**: `GET /saml/:org_slug/:service_slug/metadata`

## Best Practices
- **Test SMTP settings** immediately after saving by requesting a password reset email.
- **Rotate OAuth secrets** if you suspect they have been compromised. AuthOS keeps only the latest valid secret.
- **Use a dedicated OAuth app** for AuthOS to isolate it from your other integrations.
