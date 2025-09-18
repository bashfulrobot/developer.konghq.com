# Kong Developer Documentation - Docker

Run the Kong Developer Documentation offline using Docker with a simplified single-stage build.

## Quick Start

### Option 1: Pre-built Image (Recommended)

```bash
# Pull and run the latest image
docker pull ghcr.io/bashfulrobot/kong-docs:latest
docker run -d -p 8080:80 ghcr.io/bashfulrobot/kong-docs:latest
```

Visit [http://localhost:8080](http://localhost:8080) to view the documentation.

### Option 2: Build Locally

```bash
# Clone this repository and navigate to docker folder
cd docker/

# Build and run with Docker Compose
docker-compose up
```

Visit [http://localhost:8080](http://localhost:8080) to view the documentation.

## What You Get

- **Complete Kong documentation** with full styling and navigation
- **Offline access** - no internet required after initial download  
- **Fast performance** - optimized nginx serving
- **Simplified build** - single-stage Docker using official Makefile process
- **Auto-updating** - pre-built images are updated nightly

## Available Images

- `latest` - Most recent build
- `YYYY-MM-DD` - Date-tagged builds
- `main-<commit-sha>` - Specific commit builds

## Alternative Ports

To use a different port:

```bash
# Use port 3000 instead of 8080
docker run -d -p 3000:80 ghcr.io/bashfulrobot/kong-docs:latest
```

## Stopping the Container

```bash
# Find the container
docker ps

# Stop by name or ID
docker stop <container-name-or-id>
```

## Using Just (Optional)

If you have [just](https://github.com/casey/just) installed, you can use convenient shortcuts:

```bash
# Build and run with docker-compose
just up

# Build locally and run
just dev

# Use pre-built image
just pull-and-run

# See all available commands
just
```

## Documentation

For more detailed information:

- **[docker-build-readme.md](docker-build-readme.md)** - Local building instructions
- **[gha-readme.md](gha-readme.md)** - GitHub Actions automation setup

## Requirements

- Docker 20.10 or later
- 4GB+ available memory (for building locally)
- Internet connection (for initial image download)

## License

Same license as the main Kong Developer Documentation project.