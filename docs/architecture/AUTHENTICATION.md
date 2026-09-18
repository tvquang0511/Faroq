# Authentication Architecture

## Goal

FairRoom should support a hybrid authentication model:
- Google OAuth as the primary login method for mobile users.
- Email/password as a fallback when OAuth is unavailable or the user wants a direct login path.
- A shared token-based session layer for both login methods.

This keeps the mobile experience fast while avoiding full dependency on Google.

## Core Decisions

### 1. Use access + refresh tokens

The API should return:
- `accessToken`: short-lived token for API requests.
- `refreshToken`: long-lived token for session renewal.

Recommended behavior:
- Access token lifetime: 10-15 minutes.
- Refresh token lifetime: 30-90 days.
- Rotate refresh tokens on every refresh.
- Revoke refresh tokens on logout.

### 2. Separate user profile from login method

Do not store auth data directly on the `User` model only.
Instead, model authentication as separate identities/sessions.

Suggested domain objects:
- `User`: the app profile.
- `AuthIdentity`: the login provider identity.
- `Session` or `RefreshToken`: issued token record for revocation and rotation.

This makes it easy to support future providers like Apple or Facebook.

### 3. Google-first, password fallback

Recommended user journeys:
- Sign in with Google by default.
- Allow email/password registration for fallback.
- Allow a Google user to later set a password if needed.
- Allow a local account to later link Google if desired.

### 4. Password recovery and verification

For a normal email/password account, the auth system should also support:
- Forgot password.
- Change password while logged in.
- Email verification after sign up.

This makes the fallback path production-ready rather than just a backup login form.

### 5. API Route Protection (Guards & JWT)

All protected routes should use `@UseGuards(AccessTokenGuard)`. 
- **No manual `userId` in body:** Mobile/Frontend clients should NEVER send `userId` in the request body for standard operations (like creating a room or adding a resource). 
- **Automatic extraction:** The `AccessTokenGuard` automatically decrypts the `Bearer token`, extracts the `userId`, and attaches it to the request. Controllers should retrieve it using the `@CurrentUser()` decorator. This prevents impersonation vulnerabilities.

## Recommended Data Model

Minimum schema shape:

```text
User
- id
- displayName
- email
- avatarUrl
- emailVerifiedAt (nullable)
- createdAt
- updatedAt

AuthIdentity
- id
- userId
- provider (GOOGLE | LOCAL)
- providerUserId
- passwordHash (nullable, only for LOCAL)
- passwordUpdatedAt (nullable)
- createdAt
- updatedAt

Session / RefreshToken
- id
- userId
- deviceName (nullable)
- deviceId (nullable)
- tokenHash
- expiresAt
- revokedAt (nullable)
- createdAt
- updatedAt

PasswordResetToken
- id
- userId
- tokenHash
- expiresAt
- usedAt (nullable)
- createdAt

EmailVerificationToken
- id
- userId
- tokenHash
- expiresAt
- verifiedAt (nullable)
- createdAt
```

## Final Chosen Schema

This is the schema direction to implement for FairRoom auth.

### User

Represents the app profile, not the login method.

Fields:
- `id`
- `displayName`
- `email`
- `avatarUrl`
- `emailVerifiedAt` (nullable)
- `createdAt`
- `updatedAt`

### AuthIdentity

Represents one login method for one user.

Fields:
- `id`
- `userId`
- `provider` (`GOOGLE` | `LOCAL`)
- `providerUserId` (nullable for local only, required for OAuth)
- `passwordHash` (nullable, only for local accounts)
- `passwordUpdatedAt` (nullable)
- `createdAt`
- `updatedAt`

Rules:
- One user can have multiple identities.
- Google login uses `providerUserId = sub` from Google.
- Local login uses `passwordHash`.
- A user can start with Google and later add a password.
- A user can start with local login and later link Google.

### Session

Represents a logged-in device/session and refresh-token state.

Fields:
- `id`
- `userId`
- `deviceName` (nullable)
- `deviceId` (nullable)
- `tokenHash`
- `expiresAt`
- `revokedAt` (nullable)
- `createdAt`
- `updatedAt`

### PasswordResetToken

Represents a one-time forgot-password token.

Fields:
- `id`
- `userId`
- `tokenHash`
- `expiresAt`
- `usedAt` (nullable)
- `createdAt`

### EmailVerificationToken

Represents a one-time email verification token.

Fields:
- `id`
- `userId`
- `tokenHash`
- `expiresAt`
- `verifiedAt` (nullable)
- `createdAt`

Notes:
- `providerUserId` for Google should use the `sub` claim from Google ID token.
- Do not store raw refresh tokens in plain text if you can avoid it; store a hash.
- Keep `email` unique only where it makes sense for your account linking policy.

## Mobile Flow

### Google login

1. Mobile opens Google auth flow with PKCE.
2. Google returns authorization code.
3. Backend exchanges the code or verifies Google ID token.
4. Backend finds or creates the `User` and related `AuthIdentity`.
5. Backend returns `accessToken` and `refreshToken`.
6. Mobile stores tokens in secure storage.

### Email/password login

1. User signs up or logs in with email/password.
2. If sign up is new, backend creates a local `AuthIdentity` and sends a verification email.
3. Backend verifies password hash on login.
4. Backend returns `accessToken` and `refreshToken`.
5. Mobile stores tokens in secure storage.

### Forgot password

1. User requests password reset with email.
2. Backend creates a one-time reset token.
3. Backend sends a reset link or code by email.
4. User confirms the reset.
5. Backend updates the password hash and invalidates old sessions if required.

### Change password

1. User is already authenticated.
2. User provides current password and new password.
3. Backend validates the current password.
4. Backend updates the password hash.
5. Backend revokes or rotates sensitive sessions if policy requires it.

### Email verification

1. User registers with email/password.
2. Backend sends a verification email.
3. User clicks the verification link or enters a verification code.
4. Backend marks the email as verified.
5. Some protected actions can require verified email before proceeding.

### Token refresh

1. Mobile sends `refreshToken`.
2. Backend validates it against stored session state.
3. Backend rotates refresh token.
4. Backend returns a new token pair.

## Mobile Storage

Use secure storage only:
- iOS: Keychain
- Android: Keystore
- Expo: SecureStore

Avoid plain AsyncStorage for auth tokens.

## Environment Variables

Required API env vars for auth:

- `JWT_SECRET`: secret used to sign access tokens.
- `JWT_ACCESS_TTL`: access token lifetime, for example `15m`.
- `JWT_REFRESH_TOKEN_DAYS`: refresh token lifetime in days, for example `30`.
- `PASSWORD_SALT_ROUNDS`: bcrypt salt rounds, for example `12`.
- `GOOGLE_OAUTH_CLIENT_ID`: Google OAuth client id used to validate Google ID tokens.

Keep these values out of source control in real deployments.

## API Surface To Build

Suggested endpoints:
- `POST /auth/google`
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `POST /auth/logout`
- `GET /auth/me`
- `POST /auth/link/google`
- `POST /auth/set-password`
- `POST /auth/change-password`
- `POST /auth/forgot-password`
- `POST /auth/reset-password`
- `POST /auth/send-verification-email`
- `POST /auth/verify-email`

Recommended responsibilities:
- `POST /auth/google`: Google OAuth sign in or account creation.
- `POST /auth/register`: local email/password sign up.
- `POST /auth/login`: local email/password sign in.
- `POST /auth/refresh`: rotate access/refresh tokens.
- `POST /auth/logout`: revoke current session.
- `GET /auth/me`: return the current authenticated user.
- `POST /auth/link/google`: attach Google identity to an existing account.
- `POST /auth/set-password`: add a password to an existing Google account.
- `POST /auth/change-password`: change local password while authenticated.
- `POST /auth/forgot-password`: start forgot-password flow.
- `POST /auth/reset-password`: complete forgot-password flow.
- `POST /auth/send-verification-email`: resend email verification.
- `POST /auth/verify-email`: confirm email verification token.

## Security Notes

- Hash passwords with a strong algorithm such as Argon2 or bcrypt.
- Rate-limit login and refresh endpoints.
- Validate Google tokens server-side.
- Revoke refresh tokens on logout and account compromise.
- Keep access tokens short-lived.

## Implementation Plan

### Phase 1: Core auth foundation
- Add auth-related Prisma models.
- Implement password hashing and verification.
- Implement access token creation and verification.
- Implement refresh token storage, rotation, and revoke.
- Implement email verification and password reset token flows.

### Phase 2: Google OAuth
- Add Google login endpoint.
- Validate Google tokens or authorization codes.
- Map Google identity to user records.
- Support linking Google to an existing account.

### Phase 3: Mobile integration
- Add login/register screens in mobile.
- Store tokens securely.
- Add token refresh handling on app startup.
- Add logout and session recovery flows.

### Phase 4: Hardening
- Add rate limiting.
- Add audit/logging for auth events.
- Add tests for login, refresh, logout, and account linking.

## Decision Summary

For FairRoom mobile:
- Google should be the default login.
- Email/password should remain as fallback.
- Password recovery and email verification should be first-class flows for the fallback path.
- Sessions should track device metadata even if the app only uses one device at first.
- Access/refresh token architecture is still the right choice.
- Separate auth identity/session data from user profile data.

This is the final direction for implementation.
