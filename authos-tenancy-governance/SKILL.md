---
name: authos-tenancy-governance
description: Manage the complete lifecycle of tenant organizations from a platform owner perspective. Includes approving new organizations, managing subscription tiers, suspending or activating tenants, and reviewing platform-wide audit logs. Use this when a user needs to perform administrative actions on tenant accounts.
---

# AuthOS Tenancy Governance

This skill covers the administrative tasks required to manage organizations (tenants) on the AuthOS platform. These actions require **Platform Owner** privileges.

## 1. Organization Lifecycle Management

Organizations move through several states. You can manage these via the Platform API or Admin Dashboard.

### Approving a New Organization
When a new organization registers, it typically starts in a `pending` state.
- **Endpoint**: `POST /api/platform/organizations/:id/approve`
- **Body**: `{"tier_id": "tier_pro"}` (Optional, defaults to `tier_free`)
- **Action**: Use this to grant an organization access to the platform.

### Rejecting an Organization
- **Endpoint**: `POST /api/platform/organizations/:id/reject`
- **Body**: `{"reason": "Incomplete documentation"}`
- **Action**: Moves the organization to `rejected` status.

### Suspending and Activating
- **Suspend**: `POST /api/platform/organizations/:id/suspend` (Blocks all authentication for that tenant).
- **Activate**: `POST /api/platform/organizations/:id/activate` (Restores access for a suspended tenant).

## 2. Tier and Limit Management

You can override defaults for specific organizations.

### Updating Tiers
- **Endpoint**: `PATCH /api/platform/organizations/:id/tier`
- **Body**: 
  ```json
  {
    "tier_id": "tier_enterprise",
    "max_services": 50,
    "max_users": 5000
  }
  ```

### Feature Overrides
Enable specific features regardless of the assigned tier.
- **Endpoint**: `PATCH /api/platform/organizations/:id/features`
- **Available Fields**:
  - `allow_custom_domain`: boolean
  - `allow_saml_idp`: boolean
  - `allow_scim`: boolean
  - `allow_siem`: boolean
  - `allow_branding`: boolean
  - `allow_passkeys`: boolean
  - `allowed_social_providers`: string[] (e.g., `["github", "google"]`)

## 3. Platform Analytics and Audit

### Monitoring Growth
- **Overview**: `GET /api/platform/analytics/overview` (Total orgs, active users, growth rate).
- **Status Breakdown**: `GET /api/platform/analytics/organization-status`.
- **Top Organizations**: `GET /api/platform/analytics/top-organizations` (By MAU or login count).

### Audit Logs
The Platform Audit Log tracks every administrative action taken by platform owners.
- **Get Audit Log**: `GET /api/platform/audit-log`
- **Common Event Types**:
  - `approve_organization`
  - `suspend_organization`
  - `update_organization_tier`
  - `promote_platform_owner`

## 4. User Impersonation (Safety Warning)
Platform owners can impersonate a user to troubleshoot issues. This action is heavily audited.
- **Endpoint**: `POST /api/platform/impersonate`
- **Body**: `{"user_id": "..."}`
- **Note**: This generates a session for the target user that the platform owner can use.

## Best Practices
- **Always provide a reason** when rejecting or suspending an organization.
- **Monitor the `pending` queue** daily to ensure a smooth onboarding experience.
- **Verify custom domain requests** before enabling `allow_custom_domain` if it's an enterprise-only feature.
