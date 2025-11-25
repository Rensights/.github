# GitHub Actions CI/CD Workflows

## Workflows Overview

### 1. CI - Build, Test, Scan & Push (`ci.yml`)

**Triggers**: PRs and pushes to `main`/`develop`

**What it does**:
- Builds all services (backend, frontend, admin-backend, admin-frontend)
- Runs tests
- Builds Docker images
- Pushes to Harbor with `sha-<commit>` tag
- Runs Trivy vulnerability scanning
- Generates SBOM (Software Bill of Materials)
- Uploads artifacts

**Required Secrets**:
- `HARBOR_REGISTRY`
- `HARBOR_USERNAME`
- `HARBOR_PASSWORD`

### 2. Deploy to Dev (`deploy-dev.yml`)

**Triggers**: Push to `main`/`develop` (automatic)

**What it does**:
- Gets latest image SHA from CI
- Deploys to dev cluster using Helm
- Uses `env-values/dev/*.yaml` for configuration
- Tags successful deployment as `canary`

**Required Secrets**:
- `DEV_KUBECONFIG` (base64 encoded)

### 3. Deploy to Production (`deploy-prod.yml`)

**Triggers**: Manual (`workflow_dispatch`) or GitHub release

**What it does**:
- Requires approval (GitHub Environments)
- Deploys with `release-vX.Y.Z` tag
- Uses `env-values/prod/*.yaml` for configuration
- Runs post-deploy smoke tests
- Promotes to `latest` on success

**Required Secrets**:
- `PROD_KUBECONFIG` (base64 encoded)

## Image Tagging Strategy

- `sha-<commit>`: Every build (e.g., `sha-abc123`)
- `canary`: Latest successful dev deployment
- `release-vX.Y.Z`: Production releases (e.g., `release-v1.2.3`)
- `latest`: Latest successful production deployment

## Setup Instructions

1. Add GitHub Secrets (Settings → Secrets → Actions)
2. Configure GitHub Environments (Settings → Environments)
   - Create `dev` environment (no approval needed)
   - Create `production` environment (require approval)
3. Push to `develop` branch to trigger automatic dev deployment
4. Create release tag for production deployment

## Troubleshooting

- Check Actions tab for workflow runs
- View logs for failed steps
- Verify secrets are set correctly
- Check kubeconfig is valid
