# Component: CI/CD

## Purpose
Automate build, test, and deploy across backend and frontend. Enforce quality gates.

## Responsibilities
- Build and test every PR
- Enforce lint, analyzers, coverage thresholds
- Run DB migrations on deploy
- Publish Docker images
- Deploy to AWS staging, then prod

## GitHub Actions Workflows
- `.github/workflows/ci.yml`
  - Triggers on PR and push
  - Steps:
    - Restore → Build → Test → Coverage
    - Frontend lint + test
    - Build Docker images
    - Upload artifacts
- `.github/workflows/deploy.yml`
  - Manual approval for prod
  - Run DB migrations
  - Push images to ECR
  - Deploy via ECS or Lambda/API Gateway

## Rollback
- Previous Docker image redeploy
- DB migration rollback scripts stored in repo

## Quality Gates
- Backend unit test coverage ≥ 80%
- Frontend unit test coverage ≥ 70%
- No lint errors
- Security scan (future: Trivy, Snyk)

## Environments
- Local: docker-compose
- Staging: full stack + seed data
- Prod: secure, no test data
