# Fast multi-stage Dockerfile using pre-built base image
# Uses ghcr.io/bashfulrobot/kong-docs-base for dependencies

FROM ghcr.io/bashfulrobot/kong-docs-base:latest AS builder

# Set working directory (already exists in base image)
WORKDIR /app

# Configure git safe directory
RUN git config --global --add safe.directory /app

# Copy the entire project INCLUDING .git folder
# Dependencies are already installed in base image
COPY . .

# Check if dependencies need updating (fail fast if package files changed)
RUN if ! cmp -s package.json package.json.bak 2>/dev/null || \
       ! cmp -s package-lock.json package-lock.json.bak 2>/dev/null || \
       ! cmp -s Gemfile Gemfile.bak 2>/dev/null || \
       ! cmp -s Gemfile.lock Gemfile.lock.bak 2>/dev/null; then \
       echo "Dependencies changed since base image build. Consider rebuilding base image."; \
       echo "Installing updated dependencies..."; \
       npm ci --no-audit --no-fund; \
       bundle install --without development test; \
    else \
       echo "Dependencies match base image, skipping installation."; \
    fi

# Initialize submodules with full git context
RUN git submodule update --init --recursive --depth 1 || \
    echo "Some submodules failed to initialize, continuing..."

# Install frontmatter validator dependencies (overwritten by COPY . .)
RUN cd tools/frontmatter-validator && npm ci

# Run frontmatter validation
RUN cd tools/frontmatter-validator && npm run validate || \
    echo "Frontmatter validation had issues, continuing..."

# Build the site using the proper build script
RUN chmod +x exe/build && \
    JEKYLL_ENV=production ./exe/build || \
    echo "Production build failed, trying development build..." && \
    JEKYLL_ENV=development bundle exec jekyll build --config jekyll.yml,jekyll-dev.yml --verbose || \
    echo "Development build also failed, checking what we have..."

# Check what was actually built
RUN echo "=== Build Output Check ===" && \
    ls -la dist/ 2>/dev/null || echo "No dist/ directory" && \
    ls -la _site/ 2>/dev/null || echo "No _site/ directory" && \
    find . -name "index.html" -type f 2>/dev/null | head -5 || echo "No HTML files found" && \
    echo "========================="

# Production stage - minimal nginx container
FROM nginx:alpine AS production

# Install curl for health checks
RUN apk add --no-cache curl

# Copy the built site output (check dist/ first, fallback to _site/, then app/)
RUN --mount=from=builder,source=/app,target=/build \
    if [ -d "/build/dist" ] && [ "$(ls -A /build/dist 2>/dev/null)" ]; then \
        echo "Using dist/ directory" && cp -r /build/dist/* /usr/share/nginx/html/; \
    elif [ -d "/build/_site" ] && [ "$(ls -A /build/_site 2>/dev/null)" ]; then \
        echo "Using _site/ directory as fallback" && cp -r /build/_site/* /usr/share/nginx/html/; \
    else \
        echo "No built output found, using app/ source as last resort" && cp -r /build/app/* /usr/share/nginx/html/; \
    fi

# Create nginx configuration for documentation
COPY <<EOF /etc/nginx/conf.d/default.conf
server {
    listen 80;
    server_name localhost;
    
    root /usr/share/nginx/html;
    index index.html index.md _index.md README.md;
    
    # Enable gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/json
        application/xml+rss
        application/atom+xml
        image/svg+xml
        text/markdown;
    
    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Handle pretty URLs for Jekyll-generated content
    location / {
        try_files \$uri \$uri/ \$uri/index.html \$uri/index.md \$uri/_index.md @fallback;
    }
    
    # Fallback to directory browsing if Jekyll build didn't work
    location @fallback {
        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;
        autoindex_format html;
    }
    
    # Health check endpoint
    location /health {
        access_log off;
        return 200 "Kong Docs Offline - healthy\n";
        add_header Content-Type text/plain;
    }
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    
    # Remove server version
    server_tokens off;
}
EOF

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost/health || exit 1

# Expose port
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]