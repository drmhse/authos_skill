---
name: authos-backend-integration
description: Secure backend APIs by verifying AuthOS JWTs. Includes integrating the Node.js verifier, configuring Express middleware, and fetching public keys from the JWKS (JSON Web Key Set) endpoint for manual verification in any language. Use this when a user is building an API that needs to authorize incoming requests.
---

# AuthOS Backend Integration

This skill covers how to protect your backend services by validating the JWT access tokens issued by AuthOS. This is essential when using BYOO or standard AuthOS login flows.

## 1. How Token Verification Works

AuthOS issues JSON Web Tokens (JWT) signed with an asymmetric RS256 private key. To verify these tokens, your backend must fetch the corresponding public keys from the AuthOS JWKS (JSON Web Key Set) endpoint.

- **JWKS Endpoint**: `GET /.well-known/jwks.json`
- **OIDC Discovery**: `GET /.well-known/openid-configuration`

Your backend should decode the incoming JWT header, read the `kid` (Key ID), and find the matching public key in the JWKS to verify the token's cryptographic signature.

## 2. Node.js Integration

AuthOS provides a dedicated Node.js package that handles JWKS caching and token verification for you.

### Installation
```bash
npm install @drmhse/authos-node
```

### Direct Token Verification
Use `createTokenVerifier` to programmatically verify a token. This automatically fetches and caches the JWKS.

```typescript
import { createTokenVerifier } from '@drmhse/authos-node';

const verifier = createTokenVerifier({
  baseURL: 'https://api.your-authos-platform.com' // Do not include /api
});

try {
  // Pass the raw JWT string
  const verified = await verifier.verifyToken(token);
  
  console.log('User ID:', verified.claims.sub);
  console.log('User Email:', verified.claims.email);
} catch (error) {
  console.error('Invalid or expired token', error);
}
```

### Express Middleware
If you are using Express, use the built-in middleware to protect your routes.

```typescript
import express from 'express';
import { createAuthMiddleware } from '@drmhse/authos-node/express';

const app = express();

const { requireAuth } = createAuthMiddleware({
  baseURL: 'https://api.your-authos-platform.com'
});

// Protect a route
app.get('/api/protected', requireAuth, (req, res) => {
  // The verified token is automatically attached to req.auth
  res.json({ message: 'Success', user: req.auth.claims.sub });
});
```

## 3. Manual Verification (Other Languages)

If you are not using Node.js, you can manually verify tokens using standard JWT libraries available in Python, Go, Ruby, etc.

1. **Extract** the token from the `Authorization: Bearer <token>` header.
2. **Fetch** the public keys from `https://<auth-os-domain>/.well-known/jwks.json`. Note: You should cache this response to avoid hitting AuthOS on every request.
3. **Verify** the RS256 signature using the appropriate public key that matches the `kid` in the token's header.
4. **Validate** the claims: ensure the token is not expired (`exp`).

## Best Practices
- **Never verify tokens symmetrically** (e.g., using a shared secret). Always use the JWKS endpoint.
- **Cache JWKS** aggressively. AuthOS master keys only rotate every few months.
- **Respect Clock Skew** by allowing a few seconds of tolerance when checking expiration times.