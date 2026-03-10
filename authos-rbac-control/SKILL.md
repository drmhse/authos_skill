---
name: authos-rbac-control
description: Implement granular access control and team management. Includes managing default (Owner, Admin, Member) and custom roles, inviting new team members, and configuring SCIM provisioning for automated user management from external identity providers. Use this when a user needs to manage their team's permissions.
---

# AuthOS RBAC and team management

This skill covers the implementation of Role-Based Access Control (RBAC) and team management within an AuthOS organization.

## 1. Understanding Default Roles

AuthOS comes with three built-in roles that cannot be deleted:

| Role | Permissions | Description |
|------|-------------|-------------|
| **Owner** | `*` | Full access to everything, including billing and organization deletion. |
| **Admin** | `org:manage` | Can manage members, services, and configuration, but cannot delete the organization. |
| **Member**| `org:view` | Basic access. Can view assigned services but cannot modify organization settings. |

## 2. Managing Custom Roles

For more granular control, you can create custom roles with specific permission sets.

### Create a Custom Role
- **Endpoint**: `POST /api/organizations/:org_slug/roles`
- **Body**:
  ```json
  {
    "slug": "billing-manager",
    "name": "Billing Manager",
    "description": "Can only access invoices and payment methods",
    "permissions": ["org:billing", "org:view"]
  }
  ```

### Assigning Roles
Roles are assigned to users via their **Membership**.
- **Update Member Role**: `PATCH /api/organizations/:org_slug/members/:user_id`
- **Body**: `{"role": "billing-manager"}`

## 3. Team Invitations

Invite new users to your organization via email.

### Sending an Invitation
- **Endpoint**: `POST /api/organizations/:org_slug/invitations`
- **Body**:
  ```json
  {
    "email": "colleague@company.com",
    "role": "member"
  }
  ```
- **Process**: The user receives an email with a unique token. Once they accept, they are added to the organization with the specified role.

### Managing Invitations
- **List Invitations**: `GET /api/organizations/:org_slug/invitations`
- **Cancel Invitation**: `POST /api/organizations/:org_slug/invitations/:id/cancel`

## 4. Enterprise Provisioning (SCIM)

SCIM (System for Cross-domain Identity Management) allows you to automate user provisioning from providers like Okta or Azure AD.

### Enabling SCIM
1. Generate a SCIM Bearer Token:
   - **Endpoint**: `POST /api/organizations/:org_slug/scim-tokens`
   - **Body**: `{"name": "Okta Provisioning"}`
2. Use the provided token and the SCIM Base URL in your Identity Provider.
   - **SCIM Base URL**: `https://<api-domain>/scim/v2`

### Supported SCIM Operations
- `GET /scim/v2/Users`: List or filter users.
- `POST /scim/v2/Users`: Create a new user.
- `PATCH /scim/v2/Users/:id`: Update user attributes or status.
- `DELETE /scim/v2/Users/:id`: Remove a user.

## Best Practices
- **Principle of Least Privilege**: Start users with the `Member` role and only upgrade them to `Admin` if they need to manage other users.
- **Use Custom Roles** for specialized tasks (e.g., "Developer", "Support") to avoid over-provisioning permissions.
- **Audit your team regularly**. Use the Organization Audit Log (`GET /api/organizations/:org_slug/audit-log`) to track role changes.
