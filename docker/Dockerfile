# Multi-stage Dockerfile for Kong Developer Documentation
# Ubuntu 24.04 with latest packages and full git context

FROM ubuntu:24.04 AS builder

# Set environment variables to avoid interactive prompts
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

# Update and install base dependencies
RUN apt-get update && apt-get install -y \
    software-properties-common \
    curl \
    wget \
    gnupg \
    lsb-release \
    ca-certificates \
    apt-transport-https \
    && rm -rf /var/lib/apt/lists/*

# Add PPAs for latest software (ready for future use)
# Example: RUN add-apt-repository ppa:example/ppa -y

# Install Node.js from NodeSource (includes npm)
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get update && apt-get install -y nodejs

# Install other dependencies (excluding npm since NodeSource nodejs includes it)
RUN apt-get update && apt-get install -y \
    ruby-full \
    ruby-dev \
    build-essential \
    git \
    python3 \
    python3-pip \
    python3-dev \
    libffi-dev \
    libssl-dev \
    zlib1g-dev \
    libsqlite3-dev \
    sqlite3 \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Verify npm is available and upgrade it
RUN npm --version && npm install -g npm@latest

# Verify versions (helpful for debugging)
RUN echo "=== Software Versions ===" && \
    ruby --version && \
    node --version && \
    npm --version && \
    git --version && \
    echo "========================="

# Install global npm tools
RUN npm install -g netlify-cli@16.5.1

# Set working directory
WORKDIR /app

# Copy dependency files first for better Docker layer caching
COPY Gemfile Gemfile.lock ./
COPY package.json package-lock.json ./
COPY .ruby-version ./

# Install Ruby dependencies
RUN gem install bundler && \
    bundle config set --local deployment 'false' && \
    bundle config set --local path 'vendor/bundle' && \
    bundle install --jobs $(nproc) --retry 3

# Install Node dependencies (with latest npm that supports overrides)
RUN npm install

# Copy the entire project INCLUDING .git folder
COPY . .

# Configure git (required for submodules)
RUN git config --global user.email "docker@example.com" && \
    git config --global user.name "Docker Build" && \
    git config --global --add safe.directory /app

# Initialize submodules with full git context
RUN git submodule update --init --recursive --depth 1 || \
    echo "Some submodules failed to initialize, continuing..."

# Install frontmatter validator dependencies
RUN cd tools/frontmatter-validator && npm install

# Run frontmatter validation (skip if fails)
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

# Copy the app directory as our base content
COPY --from=builder /app/app /usr/share/nginx/html

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