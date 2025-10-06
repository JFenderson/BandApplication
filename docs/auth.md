# Component: Authentication and Authorization

## Purpose
Provide secure login, session, and role-based access for Student, Recruiter, Admin.

## Responsibilities
- Account registration and login
- Email verification and password reset
- JWT access and refresh token issuance and rotation
- Role and claim evaluation on every request
- Session revocation and logout
- Basic account audit logs

## Non-Responsibilities
- Social login
- SSO or enterprise IdP
- Fine-grained ABAC beyond roles and a small set of claims

## Dependencies
- ASP.NET Core Identity
- EF Core + PostgreSQL
- DataProtection for token keys
- Serilog for audit logs

## Data Model (Identity tables plus fields)
- AspNetUsers
  - Id, Email, EmailConfirmed
  - PasswordHash, SecurityStamp, LockoutEnd
  - TwoFactorEnabled
  - UserType enum mirror for quick read: Student, Recruiter, Admin
- AspNetRoles: Student, Recruiter, Admin
- RefreshTokens
  - Id, UserId, TokenHash, ExpiresAt, CreatedAt, RevokedAt, ReplacedByTokenHash, IP, UA

## Roles and Permissions
- Student
  - Read self profile, edit self profile
  - Upload own videos
  - Manage own interests
- Recruiter
  - Read students
  - Rate and comment
  - Send offers for own band only
- Admin
  - Manage users and bands
  - Read system stats

## Claims
- sub: user id
- email
- role
- bandId (for recruiter accounts only)
- ver: token schema version for future rotation

## Token Strategy
- Access token: JWT, 15 minutes
- Refresh token: opaque, stored hashed in DB, 30 days sliding
- Rotation: every refresh call issues a new refresh token and revokes the old one
- Invalidation: on password change, role change, or manual admin action, increment a user token version and reject older tokens

## Secrets and Keys
- Signing key: symmetric via Key Vault or AWS Secrets Manager
- DataProtection keys: persisted to a shared store for multi-instance
- No secrets in code or repo

## API Endpoints
- POST `/api/auth/register`
  - Body: email, password, role, optional bandId for recruiter
  - Effects: create user, assign role, send email verify link
  - Errors: 400 validation, 409 email exists
- POST `/api/auth/login`
  - Body: email, password
  - Returns: accessToken, refreshToken, expiresIn
  - Errors: 400 invalid credentials, 423 locked
- POST `/api/auth/refresh`
  - Body: refreshToken
  - Returns: new accessToken, new refreshToken
  - Errors: 401 invalid or revoked token
- POST `/api/auth/logout`
  - Body: refreshToken
  - Effects: revoke given token
- POST `/api/auth/request-email-verify`
  - Body: email
- POST `/api/auth/confirm-email`
  - Query: userId, code
- POST `/api/auth/request-password-reset`
  - Body: email
- POST `/api/auth/reset-password`
  - Body: userId, code, newPassword
- GET `/api/auth/me`
  - Returns: current user profile and claims

## Validation and Policies
- Password policy
  - Min length 12
  - Require upper, lower, digit, symbol
- Email confirmation required before login for Recruiter and Admin
- Lockout after 10 failed attempts for 15 minutes
- CORS restricted to known app origins
- Rate limits
  - Login: 5 per minute per IP
  - Password reset: 3 per hour per user
- MFA (optional now, planned later)
  - TOTP support, enforceable per role with a feature flag

## Middleware Pipeline
- HTTPS redirection
- Correlation ID
- Serilog request logging
- Rate limiting
- Authentication
- Authorization
- ProblemDetails for consistent errors

## Authorization Guards (Backend)
- `[Authorize(Roles="Admin")]` for admin controllers
- `[Authorize(Roles="Recruiter")]` for recruiter actions
- `[Authorize]` plus resource checks for student self actions
- Custom policy: `SameBand` validates recruiter.bandId matches target resource

## Angular Client Notes
- Store access token in memory
- Store refresh token in httpOnly secure cookie or do refresh via silent endpoint using cookie. If not using cookies, store refresh token in IndexedDB encrypted. Avoid localStorage for long-term tokens.
- HTTP interceptor
  - Attach access token
  - On 401, attempt one refresh, then redirect to login
- Idle timeout prompt at 13 minutes, refresh on interaction

## Email Flows
- Email verification on register
- Password reset link with short TTL, one-time use
- Use transactional provider with domain verification

## Audit Logging
- Login success and failure
- Token refresh
- Password reset request and completion
- Role changes
- Admin actions
  - Include userId, IP, User Agent, timestamp, outcome

## Threat Model and Controls
- Credential stuffing
  - Rate limits, lockout, optional MFA
- Token theft
  - Short access token TTL
  - Refresh rotation and revocation
  - Bind refresh tokens to UA/IP fingerprint for anomaly checks
- Privilege escalation
  - Role checks in controllers and services
  - Band-bound checks for recruiter actions
- Injection
  - Parameterized queries via EF Core
  - Input validation and model binding checks
- XSS and CSRF
  - Angular templates escape by default
  - For refresh via cookie, enforce SameSite and CSRF token on state-changing requests
- Replay
  - Nonce for critical actions if needed
  - Reject reused refresh tokens

## Sample Policies (pseudocode)
- StudentSelf:
  - userId == route.userId
- RecruiterBandScope:
  - user.role == Recruiter and user.bandId == target.bandId
- AdminOnly:
  - user.role == Admin

## Error Contract (ProblemDetails)
- 400: `type`, `title`, `errors` by field
- 401: `detail` “invalid credentials” or “token expired”
- 403: `detail` “forbidden”
- 409: `detail` “email already exists”
- 423: `detail` “account locked”

## Sequence: Login and Refresh
- `mermaid sequenceDiagram`
- participant UI as Angular App
- participant API as Auth API
- participant DB as Identity + RefreshTokens

- UI->>API: POST /auth/login (email, password)
- API->>DB: Validate user, password, lockout
- DB-->>API: OK
- API-->>UI: accessToken(15m), refreshToken(30d)

- Note over UI: Use access token for API calls

- UI->>API: POST /auth/refresh (refreshToken)
- API->>DB: Validate and rotate refresh token
- DB-->>API: New refresh stored, old revoked
- API-->>UI: new accessToken, new refreshToken