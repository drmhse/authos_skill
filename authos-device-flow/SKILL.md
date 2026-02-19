---
name: authos-device-flow
description: Implement RFC 8628 (OAuth 2.0 Device Authorization Grant) for input-constrained devices like CLIs, Smart TVs, or headless IoT devices. Includes requesting device codes, polling for tokens, and creating custom activation pages. Use this when a user is building a CLI or hardware-based integration.
---

# AuthOS Device Flow (RFC 8628)

The Device Authorization Grant is used by applications that have no browser or are input-constrained (e.g., CLIs, IoT devices).

## 1. Flow Overview

1. **Device** requests a code from AuthOS.
2. **Device** displays a `user_code` and `verification_uri` to the user.
3. **User** visits the URI on another device (e.g., phone), enters the code, and logs in.
4. **Device** polls AuthOS until the user completes the login.
5. **AuthOS** returns tokens to the device.

## 2. Implementation Steps

### Step 1: Request Device Code
The device calls the API to initialize the flow.
- **Endpoint**: `POST /api/auth/device/code`
- **Body**:
  ```json
  {
    "client_id": "cli-client",
    "org": "acme-corp",
    "service": "main-cli"
  }
  ```
- **Response**:
  ```json
  {
    "device_code": "...",
    "user_code": "ABCD-EFGH",
    "verification_uri": "https://auth.acme.com/activate",
    "expires_in": 300,
    "interval": 5
  }
  ```

### Step 2: User Authorization
The user must visit the `verification_uri` and enter the `user_code`.
- **Frontend Action**: Your activation page should call `POST /api/auth/device/verify` with the `user_code` to show the correct login options (e.g., "Login with Google", "Login with GitHub").

### Step 3: Polling for Tokens
The device polls the token endpoint.
- **Endpoint**: `POST /api/auth/device/token`
- **Body**:
  ```json
  {
    "client_id": "cli-client",
    "device_code": "...",
    "grant_type": "urn:ietf:params:oauth:grant-type:device_code"
  }
  ```
- **Poling Interval**: Respect the `interval` returned in Step 1 (usually 5 seconds).
- **Errors**:
  - `authorization_pending`: Keep polling.
  - `slow_down`: Increase polling interval by 5 seconds.
  - `expired_token`: Stop polling; the user took too long.

### Step 4: Token Receipt
Once the user authorizes, Step 3 returns:
```json
{
  "access_token": "...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## 3. SDK Implementation

If using the TypeScript SDK in a CLI:

```typescript
const deviceAuth = await sso.auth.deviceCode.request({
  client_id: 'my-cli',
  org: 'acme-corp',
  service: 'main-cli'
});

console.log(`Open: ${deviceAuth.verification_uri}`);
console.log(`Code: ${deviceAuth.user_code}`);

// The SDK has a built-in helper for polling
const tokens = await sso.auth.deviceCode.poll(deviceAuth.device_code, 'my-cli');
console.log('Logged in successfully!');
```

## Security Considerations
- **MFA**: If the user has MFA enabled, they MUST complete it in the browser during the activation step. The device flow will fail with an error if MFA is required but not completed.
- **Expiration**: Device codes are short-lived (typically 5-10 minutes).
- **Client ID**: The `client_id` used for device flow must correspond to a service of type `api` or `desktop`.
