# Component: Notifications

## Purpose
Inform users about important events with reliable, secure, and low-noise messages.

## Responsibilities
- Trigger notifications from domain events.
- Deliver email (phase 1) and in-app toasts/feeds (phase 2).
- Template rendering with variables.
- Throttling, rate limits, and user preferences.
- Delivery tracking and retries.

## Non-Responsibilities
- SMS/push (future).
- Batch marketing campaigns.

## Triggers (events)
- OfferCreated (recruiter → student)
- OfferUpdated/Revoked (recruiter/admin → student)
- VideoCommentAdded (recruiter → student)
- VideoRated (recruiter → student)
- BandInterestReceived (student → recruiter)
- Account actions (email verify, password reset)
- System alerts (optional admin-only)

## Channels
- Email (SES/SendGrid/Postmark)
- In-app notifications (feed + toast) [phase 2]

## Data Model
- `Notification`
  - `Id` (guid)
  - `UserId` (recipient)
  - `Type` (enum)
  - `Channel` (Email | InApp)
  - `Payload` (json)
  - `Status` (Pending | Sent | Failed | Read)
  - `CreatedAt`, `SentAt`, `ReadAt`
- `UserNotificationPreference`
  - `UserId`
  - `EmailEnabled` (bool)
  - `InAppEnabled` (bool)
  - Per-type overrides (json map)
- Indexes: `(UserId, Status, CreatedAt)`, `(Type, CreatedAt)`

## Templates
- Store template files with placeholders:
  - `/templates/email/OfferCreated.html`
  - `/templates/email/OfferUpdated.html`
  - …
- Variables:
  - `{{studentName}}`, `{{recruiterName}}`, `{{bandName}}`, `{{amount}}`, `{{expiresOn}}`, `{{link}}`
- Plain-text fallback for all emails.
- Branding in header/footer.

## API (server)
- POST `/api/notifications/preview`
  - Dev/admin tool to preview a template with sample data.
- GET `/api/notifications/feed`
  - Auth user’s in-app feed (paged).
- POST `/api/notifications/{id}/read`
  - Mark as read.
- Admin (optional):
  - GET `/api/notifications/search?userId=&type=&status=`

## Emission Pattern
- Domain service raises event (e.g., `OfferCreated`).
- Event handler writes `Notification` rows by recipient.
- Outbox pattern publishes to background worker for delivery (idempotent).

## Delivery (email)
- Background worker pulls Pending.
- Renders template with payload.
- Sends via provider SDK.
- Updates status: Sent/Failed with provider messageId.
- Retry with backoff on transient errors.
- Dead-letter after N attempts (alert).

## Delivery (in-app) [phase 2]
- Write to `Notification` with `Channel=InApp`.
- Web client polls or uses SSE/WebSocket later.
- Show toast + add to feed.

## Links
- Every email link goes to a signed, minimal URL:
  - Offer view/download
  - Video detail (comments/ratings)
- Use UTM-like query params for basic telemetry.

## Preferences & Opt-Out
- Default: transactional emails ON.
- Per-type toggles in user settings.
- Students can mute recruiter-generated comments/ratings emails.
- Password reset and verify emails always ON.

## Rate Limits & Digest
- Hard caps per user:
  - Email: 10/hour, 50/day (config).
  - In-app: soft cap with grouping UI.
- Digest mode (optional): “Daily summary” for low-priority items.
- Coalesce bursts (e.g., multiple comments) into one email within 5–10 minutes.

## Security
- Never include sensitive PII in email body beyond names and high-level context.
- Use signed, short-lived links for files (PDF offers).
- Validate recipient authorization when fetching notification feed.
- Store provider API keys in Secrets Manager.

## Observability
- Logs:
  - `notif.created`, `notif.sent`, `notif.failed`, with `type`, `userId`, `messageId`.
- Metrics:
  - Send rate by type/channel
  - Failure rate
  - Open/Click (if provider webhooks added later)
- Alerts:
  - Failure rate > threshold for 10 minutes
  - Backlog age > threshold

## Testing
- Unit: template rendering variables; preference checks; rate-limit guard.
- Integration: trigger → outbox → worker → provider mock → status updates.
- E2E: recruiter creates offer → student receives email and in-app entry.
- Negative: provider outage, template missing variable, muted type.

## Configuration
- `Notifications:Email:Provider` (SES/SendGrid/Postmark)
- `Notifications:Email:FromAddress`
- `Notifications:Limits:PerHour`, `PerDay`
- `Notifications:Digest:WindowMinutes`