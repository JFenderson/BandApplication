# Component: Logging & Monitoring

## Purpose
Support observability, troubleshooting, and performance tracking.

## Logging
- Structured logs (Serilog)
- Context: requestId, userId, role, bandId
- CorrelationId middleware
- Levels: Debug (dev), Info (staging), Warn/Error (prod)

## Metrics
- Request latency (avg, P95)
- Video upload success/failure
- Offer creation counts
- Auth failures

## Health
- `/health` (liveness, DB up)
- `/metrics` for Prometheus (future)

## Alerts
- Failed logins spike
- Video upload aborts > threshold
- Offer PDF generation failures > threshold
- CloudFront 5xx rate > threshold

## Dashboards
- API latency distribution
- Upload throughput
- Active users per role
