# GitHub Actions CI/CD Setup Guide

This document explains how to set up and use the GitHub Actions workflows for automated deployment to your Kubernetes cluster.

## Overview

The CI/CD pipeline consists of:
- **CI Workflow**: Builds and tests all services on pull requests
- **Dev Deployment**: Automatically deploys to dev environment on push to `develop`/`dev` branches
- **Prod Deployment**: Deploys to production on push to `main`/`master` or when tags are created

## Required GitHub Secrets

You need to configure the following secrets in your GitHub repository:

### 1. SSH Access Secrets
- `SSH_PRIVATE_KEY`: Your SSH private key for server access
- `SSH_HOST`: Your server IP or hostname (e.g., `72.62.40.154`)
- `SSH_USER`: SSH username (e.g., `root`)

### 2. Kubernetes Secrets (Optional)
- `KUBECONFIG`: Base64 encoded kubeconfig file (if using remote kubectl)

### 3. Docker Registry Secrets (Optional - if using remote registry)
- `DOCKER_REGISTRY`: Docker registry URL (leave empty for local builds on server)
- `DOCKER_USERNAME`: Docker registry username
- `DOCKER_PASSWORD`: Docker registry password/token

## Setting Up Secrets

1. Go to your GitHub repository: `https://github.com/Rensights/[your-repo]`
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** for each secret above

### Getting SSH Private Key
```bash
# If you don't have an SSH key, generate one:
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/github_actions

# Copy the private key content:
cat ~/.ssh/github_actions

# Add the public key to your server:
ssh-copy-id -i ~/.ssh/github_actions.pub root@72.62.40.154
```

### Getting KUBECONFIG (Optional)
```bash
# On your server, get the kubeconfig:
cat ~/.kube/config | base64 -w 0

# Copy the output and add it as KUBECONFIG secret
```

## Workflow Files

### 1. `.github/workflows/ci.yml`
- Runs on every pull request and push
- Builds and tests all services
- Validates code quality

### 2. `.github/workflows/deploy-dev.yml`
- Triggers on push to `develop`/`dev`/`main` branches
- Builds Docker images for all services
- Deploys to `dev` namespace in Kubernetes
- Can be manually triggered with service selection

### 3. `.github/workflows/deploy-prod.yml`
- Triggers on push to `main`/`master` or tag creation
- Builds Docker images for production services
- Deploys to `prod` namespace
- Requires manual approval for production deployments

### 4. Reusable Workflows
- `build-and-push.yml`: Builds and pushes Docker images
- `deploy-k8s.yml`: Deploys to Kubernetes cluster

## Deployment Process

### Automatic Deployment (Dev)
1. Push code to `develop` or `dev` branch
2. GitHub Actions automatically:
   - Builds Docker images
   - Pushes to registry (or builds on server)
   - Updates Kubernetes deployments
   - Restarts pods

### Manual Deployment
1. Go to **Actions** tab in GitHub
2. Select the workflow (e.g., "Deploy to Dev Environment")
3. Click **Run workflow**
4. Choose the service to deploy
5. Click **Run workflow**

### Production Deployment
1. Create a release tag:
   ```bash
   git tag -a v1.0.0 -m "Release version 1.0.0"
   git push origin v1.0.0
   ```
2. Or push to `main` branch (for automatic deployment)
3. The workflow will build and deploy to production

## How It Works

### Build Process
1. Checks out code
2. Sets up Docker Buildx
3. Builds Docker image with appropriate tags
4. Pushes to registry (if configured) or prepares for local deployment

### Deployment Process
1. Connects to server via SSH
2. Builds Docker image on server (if not using registry)
3. Updates Kubernetes deployment with new image
4. Restarts the deployment
5. Waits for rollout to complete
6. Verifies deployment status

## Environment-Specific Configuration

### Dev Environment
- Namespace: `dev`
- Image tags: `dev-<commit-sha>`
- Values file: `env/dev/*.values.yaml`

### Production Environment
- Namespace: `prod`
- Image tags: `prod-<commit-sha>` or version tags
- Values file: `env/prod/*.values.yaml`

## Troubleshooting

### Build Failures
- Check build logs in GitHub Actions
- Verify Dockerfile paths are correct
- Ensure all dependencies are available

### Deployment Failures
- Verify SSH access works: `ssh root@72.62.40.154`
- Check kubectl access on server
- Verify Helm charts are correct
- Check Kubernetes namespace exists

### Image Not Found
- If using local Docker registry, ensure images are built on server
- If using remote registry, verify credentials are correct
- Check image tags match deployment configuration

## Customization

### Adding New Services
1. Add build job in `deploy-dev.yml` or `deploy-prod.yml`
2. Add deployment job referencing the build job
3. Create Helm chart in `charts/` directory
4. Add values file in `env/<environment>/`

### Changing Build Arguments
Update the `build_args` parameter in the workflow files:
```yaml
build_args: 'NEXT_PUBLIC_API_URL=https://api.example.com'
```

### Custom Deployment Steps
Modify `.github/workflows/deploy-k8s.yml` to add custom deployment steps like:
- Database migrations
- Cache warming
- Health checks
- Rollback procedures

## Security Best Practices

1. **Never commit secrets** to the repository
2. **Use environment-specific secrets** for different environments
3. **Rotate SSH keys** regularly
4. **Use least privilege** for service accounts
5. **Enable branch protection** for main branches
6. **Require approvals** for production deployments

## Monitoring Deployments

After deployment, monitor:
- Pod status: `kubectl get pods -n <namespace>`
- Deployment status: `kubectl rollout status deployment/<service>-<env> -n <namespace>`
- Application logs: `kubectl logs -f deployment/<service>-<env> -n <namespace>`
- GitHub Actions logs: Check the Actions tab for detailed logs

## Support

For issues or questions:
1. Check GitHub Actions logs
2. Review Kubernetes events: `kubectl get events -n <namespace>`
3. Check server logs: `journalctl -u docker` or `docker logs <container>`

