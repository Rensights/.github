# Quick Setup Guide for GitHub Actions CI/CD

## Step 1: Generate SSH Key for GitHub Actions

```bash
# Generate a new SSH key specifically for GitHub Actions
ssh-keygen -t ed25519 -C "github-actions@rensights" -f ~/.ssh/github_actions_rensights

# Copy the public key to your server
ssh-copy-id -i ~/.ssh/github_actions_rensights.pub root@72.62.40.154

# Display the private key (you'll need this for GitHub secret)
cat ~/.ssh/github_actions_rensights
```

## Step 2: Add GitHub Secrets

Go to: `https://github.com/orgs/Rensights/[your-repo]/settings/secrets/actions`

Add these secrets:

### Required Secrets:

1. **SSH_PRIVATE_KEY**
   - Value: Content of `~/.ssh/github_actions_rensights` (the private key)
   - Copy the entire output including `-----BEGIN OPENSSH PRIVATE KEY-----` and `-----END OPENSSH PRIVATE KEY-----`

2. **SSH_HOST**
   - Value: `72.62.40.154` (or your server IP)

3. **SSH_USER**
   - Value: `root` (or your SSH username)

### Optional Secrets:

4. **KUBECONFIG** (if you want to use remote kubectl)
   ```bash
   # On your server, get the kubeconfig:
   cat ~/.kube/config | base64 -w 0
   # Copy the output and paste as KUBECONFIG secret
   ```

5. **DOCKER_REGISTRY** (if using remote Docker registry)
   - Value: Your registry URL (e.g., `docker.io`, `ghcr.io`)

6. **DOCKER_USERNAME** (if using remote Docker registry)
   - Value: Your Docker registry username

7. **DOCKER_PASSWORD** (if using remote Docker registry)
   - Value: Your Docker registry password/token

## Step 3: Test the Setup

1. Make a small change to any service
2. Push to `develop` or `dev` branch
3. Go to **Actions** tab in GitHub
4. Watch the workflow run

## Step 4: Verify Deployment

After deployment, verify on your server:

```bash
# SSH to your server
ssh root@72.62.40.154

# Check pods
kubectl get pods -n dev

# Check deployments
kubectl get deployments -n dev

# View logs
kubectl logs -f deployment/backend-dev -n dev
```

## Troubleshooting

### SSH Connection Issues
```bash
# Test SSH connection
ssh -i ~/.ssh/github_actions_rensights root@72.62.40.154

# If it fails, check:
# 1. Public key is in ~/.ssh/authorized_keys on server
# 2. SSH service is running: systemctl status sshd
# 3. Firewall allows SSH: ufw status
```

### Deployment Not Found
If you see "deployment not found", check the actual deployment name:
```bash
kubectl get deployments -n dev
# Update the deployment_name in workflow files to match
```

### Build Failures
- Check Docker is running on server: `systemctl status docker`
- Verify Dockerfile paths are correct
- Check build logs in GitHub Actions

### Permission Issues
```bash
# Ensure kubectl has proper permissions
kubectl auth can-i create deployments -n dev

# Check service account permissions
kubectl get serviceaccount -n dev
```

## Next Steps

1. **Set up branch protection** for main branches
2. **Configure notifications** for deployment status
3. **Add staging environment** if needed
4. **Set up monitoring** and alerts

## Workflow Triggers

- **Dev Environment**: Triggers on push to `develop`, `dev`, or `main` branches
- **Production**: Triggers on push to `main`/`master` or tag creation
- **Manual**: Can be triggered manually from Actions tab with service selection

## Customization

To customize for your setup:

1. **Change deployment names**: Update `deployment_name` in workflow files
2. **Add new services**: Follow the pattern in `deploy-dev.yml`
3. **Change build args**: Update `build_args` in workflow files
4. **Add environments**: Copy `deploy-dev.yml` and modify for new environment

