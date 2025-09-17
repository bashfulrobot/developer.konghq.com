# Building Kong Developer Documentation Docker Image

This guide covers building the Docker image locally from the Dockerfile.

## Prerequisites

- Docker (version 20.10 or later)
- Docker Compose (optional, for easier orchestration)

## Quick Start

### Option 1: Docker Compose (Recommended)

```bash
# Build and run in one command
docker-compose up

# Or run in background
docker-compose up -d

# Stop
docker-compose down
```

The documentation will be available at [http://localhost:8080](http://localhost:8080)

### Option 2: Direct Docker Commands

```bash
# Build the image
docker build -t kong-docs-offline .

# Run the container
docker run -d -p 8080:80 --name kong-docs kong-docs-offline

# Stop and remove
docker stop kong-docs
docker rm kong-docs
```

## Dockerfile Explained

### `Dockerfile`
- **Base**: Ubuntu 24.04 with latest packages
- **Features**: Full Jekyll build with git context and submodules
- **Node.js**: Latest LTS (20.x) from NodeSource
- **Output**: Complete documentation with styling and navigation
- **Build**: Multi-stage build for optimized production images

## Configuration

### Environment Variables

The build process supports these environment variables:

- `JEKYLL_ENV`: Set to `production` or `development` (default: `production`)
- `VITE_PORTAL_API_URL`: Portal API URL (optional, for full feature support)

### Build Arguments

```bash
# Build with specific Jekyll environment
docker build --build-arg JEKYLL_ENV=development -t kong-docs .
```

## Customization

### Nginx Configuration

The nginx configuration supports:
- **Pretty URLs** for Jekyll-generated content
- **Directory browsing** as fallback
- **Gzip compression** for faster loading
- **Security headers** for safe browsing
- **Health check endpoint** at `/health`

### Port Mapping

Default port is 8080. To use a different port:

```bash
# Use port 3000 instead
docker run -d -p 3000:80 kong-docs-offline

# Or update docker-compose.yml:
ports:
  - "3000:80"
```

## Troubleshooting

### Build Issues

If the build fails:

1. **Check Docker version**: Ensure you're using Docker 20.10+
2. **Network issues**: Some corporate networks may block package downloads
3. **Memory**: Large builds may need more Docker memory (4GB+ recommended)

```bash
# Check build logs
docker build -t kong-docs-offline . --no-cache

# Check container logs
docker logs kong-docs
```

### Runtime Issues

If the container doesn't start or pages don't load:

```bash
# Check container status
docker ps

# Check health
curl http://localhost:8080/health

# View nginx logs
docker exec kong-docs tail -f /var/log/nginx/access.log
```

### Jekyll Build Failures

If Jekyll build fails during Docker build:
- The container will still run, serving raw markdown files
- Check build logs for specific error messages
- Missing external APIs are expected and handled gracefully

## Development

### Local Development vs Docker

- **Local development**: Use `make run` for live reloading and full feature set
- **Docker**: Use for offline documentation and production-like serving

### Updating Documentation

After updating documentation files:

```bash
# Rebuild the image
docker build -t kong-docs-offline .

# Restart container
docker-compose restart
```

## Performance

### Build Time
- Initial build: 5-15 minutes (downloads packages and builds Jekyll site)
- Subsequent builds: 2-5 minutes (Docker layer caching)

### Runtime Performance
- **Memory usage**: ~50MB (nginx + static files)
- **Startup time**: ~2 seconds
- **Response time**: <100ms for static files

## Security

The Docker setup includes:
- Non-root nginx process
- Security headers (X-Frame-Options, X-Content-Type-Options, etc.)
- No exposed build tools in final image
- Health check monitoring

## Support

For issues related to:
- **Docker building**: Review troubleshooting section above
- **Documentation content**: See the main project README
- **Jekyll build process**: See project's Makefile and build scripts
- **GitHub Actions automation**: See [gha-readme.md](gha-readme.md)

## Files Structure

```
├── Dockerfile              # Main production Dockerfile
├── docker-compose.yml      # Docker Compose configuration
├── .dockerignore           # Files to exclude from build context
├── README.md               # Quick start guide
├── docker-build-readme.md  # This file - local build instructions
└── gha-readme.md           # GitHub Actions automation setup
```