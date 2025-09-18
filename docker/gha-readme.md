# GitHub Actions Setup

This repository includes a simplified automated workflow for maintaining an up-to-date Docker image of the Kong Developer Documentation.

## Workflow Overview

### Sync Fork and Build Docker Image

**Schedule:** Nightly at 10 PM PST (6 AM UTC)

**What it does:**
1. **Syncs your fork** with the upstream Kong repository
2. **Builds single-stage Docker image** using official Makefile process
3. **Publishes** to GitHub Container Registry (GHCR)
4. **Cleans up** old container versions

**Triggers:**
- **Scheduled**: Nightly at 10 PM PST (6 AM UTC)
- **Manual**: Via GitHub Actions UI (workflow_dispatch) only

**Simplified Design:**
- Single workflow file (`sync-and-build.yml`)
- No base image complexity - everything in one Docker build
- Uses official `make install` and `make build` commands
- Only runs when scheduled or manually triggered

## Automated Workflows

This repository includes automated workflows for maintaining an up-to-date Docker image. The workflow runs daily at 10 PM PST and automatically syncs with upstream changes, builds new images, and publishes them to GitHub Container Registry.

## Using Pre-built Images

Instead of building locally, you can use the automatically built images:

```bash
# Pull from GitHub Container Registry
docker pull ghcr.io/bashfulrobot/kong-docs:latest

# Run the documentation
docker run -d -p 8080:80 ghcr.io/bashfulrobot/kong-docs:latest
```

Or use in Docker Compose:
```yaml
version: '3.8'
services:
  docs:
    image: ghcr.io/bashfulrobot/kong-docs:latest
    ports:
      - "8080:80"
```

**Available tags:**
- `latest` - Latest build from main branch
- `main-<commit-sha>` - Specific commit builds  
- `YYYY-MM-DD` - Daily date-tagged builds

## Setup Requirements

### 1. Enable GitHub Container Registry

1. Go to your repository **Settings** → **Actions** → **General**
2. Under **Workflow permissions**, select **Read and write permissions**
3. Check **Allow GitHub Actions to create and approve pull requests**

### 2. Make Repository Public (Optional)

For public container images, ensure your repository is public, or configure package visibility in your GitHub package settings.

### 3. Manual Trigger

You can manually trigger the workflow:
1. Go to **Actions** tab in your repository
2. Select **Sync Fork and Build Docker Image**
3. Click **Run workflow**

## Workflow Features

### Smart Building
- Only builds when upstream changes are detected
- Supports manual triggering for testing
- Builds when Docker-related files change

### Multi-Architecture Support
- Builds for both `linux/amd64` and `linux/arm64`
- Uses BuildKit caching for faster builds

### Automatic Cleanup
- Keeps last 10 container versions
- Removes untagged images to save space

### Build Summary
Each workflow run provides a summary with:
- Published image tags
- Usage instructions
- Registry information

## Monitoring

### Workflow Status
Monitor your workflows in the **Actions** tab of your repository.

### Container Packages
View published containers in your repository **Packages** tab.

### Logs
Full build logs are available for each workflow run for debugging.

## Troubleshooting

### Sync Conflicts
If the automatic sync fails due to merge conflicts:
1. Manually resolve conflicts in your fork
2. Push the resolution to your main branch
3. The next scheduled run will work normally

### Build Failures
Check the workflow logs for specific error messages:
- Docker build issues: Review Dockerfile in `docker/` directory
- Permission issues: Verify repository workflow permissions
- Registry issues: Check GitHub token permissions

### Container Pull Issues
If you can't pull the container:
1. Ensure the package is public or you're authenticated
2. Check package exists in your repository's Packages tab
3. Verify the image name and tag

## Customization

### Change Schedule
Edit the cron expression in `.github/workflows/sync-and-build.yml`:
```yaml
schedule:
  - cron: '0 6 * * *'  # 6 AM UTC = 10 PM PST
```

### Add Notifications
You can add Slack, Discord, or email notifications to the workflow for build status updates.

### Custom Build Args
Add build arguments to the Docker build step if needed:
```yaml
build-args: |
  JEKYLL_ENV=production
  CUSTOM_VAR=value
```

## Workflow File Structure

The GitHub Action workflow is located at:
```
.github/workflows/sync-and-build.yml
```

Key sections:
- **sync-upstream**: Fetches and merges upstream changes
- **build-and-publish**: Builds and publishes Docker images
- **cleanup**: Removes old container versions

## Security

- Uses `GITHUB_TOKEN` for authentication (automatically provided)
- Builds run in isolated GitHub runners
- Container images are signed and include build metadata
- No secrets or credentials are exposed in logs