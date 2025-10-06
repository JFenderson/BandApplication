# Component: Testing

## Purpose
Guarantee correctness and stability of features. Provide fast feedback.

## Layers
- Unit: services, validators, policies
- Integration: API endpoints + DB
- E2E: critical user flows (login, upload, offer)

## Frameworks
- Backend: xUnit + FluentAssertions
- Frontend: Jest + Angular Testing Library
- E2E: Playwright

## Coverage Targets
- Backend: ≥80%
- Frontend: ≥70%
- E2E: cover top 10 flows

## Seed Data
- Students with sample videos
- Bands + recruiters
- Offers (active, revoked, expired)

## Automation
- Run in CI on every PR
- Coverage report artifact
- Testcontainers for DB + S3 mock

## Negative Cases
- Wrong password lockout
- Invalid video upload type
- Recruiter out-of-band offer
- Expired refresh token
