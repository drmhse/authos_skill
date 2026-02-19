---
name: authos-web-integration
description: Integrate the AuthOS "Invisible SDK" into web applications. Covers session management, automatic token rotation, multi-tenant login flows, and handling authentication state changes. Use this when a developer is adding AuthOS to their frontend.
---

# AuthOS Web Integration

This skill explains how to integrate the AuthOS TypeScript SDK into your web applications properly.

## 1. Installation

```bash
npm install @drmhse/sso-sdk
```

## 2. Initialization

The SDK is designed to be a singleton in your application. It automatically manages tokens in `localStorage`.

```typescript
import { SsoClient } from '@drmhse/sso-sdk';

const sso = new SsoClient({
  baseURL: 'https://sso.your-platform.com'
});
```

## 3. Implementing Login Flows

AuthOS supports two main patterns: **OAuth Redirect** (recommended for BYOO) and **Native Password Auth**.

### OAuth Redirect Flow (Step-by-Step)
1. Redirect the user to the provider:
   ```typescript
   const loginUrl = sso.auth.getLoginUrl('google', {
     org: 'acme-corp',
     service: 'main-app',
     redirect_uri: window.location.origin + '/callback'
   });
   window.location.href = loginUrl;
   ```
2. Handle the callback:
   AuthOS returns tokens in the URL fragment (`#token=...`) to prevent them from hitting your server logs.
   ```typescript
   if (window.location.hash) {
     const params = new URLSearchParams(window.location.hash.substring(1));
     const token = params.get('access_token');
     if (token) {
       // Reset SDK with the token
       const sso = new SsoClient({ baseURL: '...', token });
       window.history.replaceState(null, '', window.location.pathname);
     }
   }
   ```

### Native Password Auth
```typescript
await sso.auth.login({
  email: 'user@example.com',
  password: 'SecurePass123!',
  org_slug: 'acme-corp',
  service_slug: 'main-app'
});
```

## 4. Handling Auth State

Use `onAuthStateChange` to update your UI (e.g., show/hide login buttons).

```typescript
sso.onAuthStateChange((isAuthenticated) => {
  if (isAuthenticated) {
    console.log('User is logged in');
  } else {
    console.log('User is logged out');
  }
});
```

## 5. Automatic Token Rotation (The "Invisible" Part)

The SDK uses a custom `fetch` wrapper that intercepts 401 errors.
- If a request fails with 401, it automatically calls `/api/auth/refresh`.
- If refresh succeeds, it retries the original request with the new token.
- **You do not need to manually handle token expiration in your business logic.**

## 6. Accessing User Data

```typescript
try {
  const profile = await sso.user.getProfile();
  const permissions = await sso.user.getPermissions('org-slug');
} catch (error) {
  // If getProfile throws, the session is likely invalid/expired
}
```

## Common Integration Pitfalls
- **Incorrect `BASE_URL`**: Ensure it includes the protocol (`https://`) and no trailing slash.
- **Missing `org_slug` during login**: While optional, omitting it will return a platform-level JWT which might not have the correct service permissions.
- **CORS Issues**: Ensure your web app's domain is allowed in the `PLATFORM_CORS_ALLOWED_ORIGINS` (if configured) or that the API is configured to allow your origin.
