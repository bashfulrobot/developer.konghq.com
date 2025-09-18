# Simple single-stage Dockerfile for local development
# Uses Ubuntu base with all dependencies and follows Makefile commands exactly

FROM ubuntu:24.04

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

# Install system dependencies
RUN apt-get update && apt-get install -y \
    software-properties-common \
    curl \
    wget \
    gnupg \
    lsb-release \
    ca-certificates \
    apt-transport-https \
    git \
    build-essential \
    ruby-full \
    ruby-dev \
    python3 \
    python3-pip \
    python3-dev \
    libffi-dev \
    libssl-dev \
    zlib1g-dev \
    libsqlite3-dev \
    sqlite3 \
    pkg-config \
    nginx \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js 20.x
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get update && apt-get install -y nodejs

# Install global npm packages and bundler
RUN npm install -g npm@latest && \
    gem install bundler

# Set working directory
WORKDIR /app

# Copy the entire project
COPY . .

# Configure git
RUN git config --global user.email "docker@example.com" && \
    git config --global user.name "Docker Build" && \
    git config --global --add safe.directory /app

# Run Makefile commands exactly as defined
# 1. Install prerequisites (Netlify CLI - needed for some build processes)
RUN make install-prerequisites || echo "Prerequisites install had issues, continuing..."

# 2. Install all dependencies using official Makefile
RUN make install

# 3. Build the site using official Makefile
RUN make build

# Configure nginx to serve the built site
RUN rm -f /etc/nginx/sites-enabled/default
COPY <<EOF /etc/nginx/sites-available/kong-docs
server {
    listen 80;
    server_name localhost;
    
    root /app/dist;
    index index.html;
    
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
        image/svg+xml;
    
    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Handle pretty URLs
    location / {
        try_files \$uri \$uri/ \$uri/index.html =404;
    }
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Remove server version
    server_tokens off;
}
EOF

# Enable the site
RUN ln -s /etc/nginx/sites-available/kong-docs /etc/nginx/sites-enabled/

# Expose port
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]