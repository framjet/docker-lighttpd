# framjet/lighttpd

[![Docker Hub](https://img.shields.io/docker/pulls/framjet/lighttpd)](https://hub.docker.com/r/framjet/lighttpd)
[![Docker Image Size](https://img.shields.io/docker/image-size/framjet/lighttpd/latest)](https://hub.docker.com/r/framjet/lighttpd)

Lightweight, high-performance Lighttpd web server Docker image based on Alpine Linux. This is the base image with full feature set, suitable for development and production deployments requiring dynamic content support.

## 🚀 Features

- **🪶 Lightweight**: Built on Alpine Linux (~14MB)
- **⚡ High Performance**: Optimized Lighttpd 1.4.79 configuration
- **🔧 Full Feature Set**: All Lighttpd modules available
- **🐳 Container-Ready**: Docker/Kubernetes optimized
- **📊 Health Monitoring**: Built-in `/status` endpoint
- **🔄 Service Dependencies**: wait4x support via environment variables
- **⚙️ Customizable**: Custom entrypoint scripts support
- **🔒 Secure**: Runs as non-root `lighttpd` user

## 🚀 Quick Start

### Basic Usage

```bash
# Run with local directory
docker run -d -p 8080:80 -v $(pwd)/html:/app framjet/lighttpd

# Run with custom configuration
docker run -d -p 8080:80 \
  -v $(pwd)/html:/app \
  -v $(pwd)/lighttpd.conf:/etc/lighttpd/lighttpd.conf \
  framjet/lighttpd
```

### Docker Compose

```yaml
version: '3.8'
services:
  web:
    image: framjet/lighttpd
    ports:
      - "8080:80"
    volumes:
      - ./html:/app
      - ./config:/etc/lighttpd/conf.d
    environment:
      - HTTP_ROOT=/app
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/status"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## 📁 Directory Structure

```
/app/                    # Document root (HTTP_ROOT)
├── index.html          # Default welcome page
└── ...                 # Your content here

/etc/lighttpd/          # Configuration directory
├── lighttpd.conf       # Main configuration
└── conf.d/             # Additional configurations
    ├── 00-mime-types.conf
    ├── 01-server.conf
    ├── 02-webroot.conf
    └── 13-status.conf

/docker-entrypoint.d/   # Custom scripts directory
└── 90-wait-for-x.sh    # wait4x integration
```

## ⚙️ Configuration

### Environment Variables

| Variable                       | Default | Description                                         |
|--------------------------------|---------|-----------------------------------------------------|
| `HTTP_ROOT`                    | `/app`  | Document root directory                             |
| `WAIT_FOR_*`                   | -       | Wait for TCP services (format: `host:port#timeout`) |
| `WAIT_CMD_FOR_*`               | -       | Custom wait4x commands                              |
| `DOCKER_ENTRYPOINT_QUIET_LOGS` | -       | Suppress entrypoint logs if set                     |

### Server Configuration

The image includes optimized server settings:

```
server.max-fds = 8192
server.max-connections = 4096
server.event-handler = "linux-sysepoll"
server.network-backend = "sendfile"
server.max-worker = 4
server.max-keep-alive-requests = 256
```

### Custom Entrypoint Scripts

Add custom initialization scripts to `/docker-entrypoint.d/`:

- `*.sh` - Executable shell scripts (run during startup)
- `*.envsh` - Sourced environment scripts (sourced into shell)

```bash
# Example: Mount custom initialization script
docker run -d \
  -v $(pwd)/my-init.sh:/docker-entrypoint.d/10-my-init.sh \
  -v $(pwd)/html:/app \
  framjet/lighttpd
```

## 🔄 Service Dependencies

Use environment variables to wait for dependent services before starting:

```bash
# Wait for PostgreSQL database
docker run -e WAIT_FOR_DB=postgres:5432#60s framjet/lighttpd

# Wait for multiple services
docker run \
  -e WAIT_FOR_DB=postgres:5432#60s \
  -e WAIT_FOR_CACHE=redis:6379#30s \
  -e WAIT_FOR_API=api-server:8080#45s \
  framjet/lighttpd

# Custom wait4x command
docker run \
  -e WAIT_CMD_FOR_HEALTH="http http://api:8080/health --timeout 30s" \
  framjet/lighttpd
```

## 📊 Health Monitoring

### Built-in Health Endpoint

```bash
# Check server status
curl http://localhost:8080/status

# Use in Docker health checks
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/status || exit 1
```

### Kubernetes Probes

```yaml
livenessProbe:
  httpGet:
    path: /status
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 30

readinessProbe:
  httpGet:
    path: /status
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```

## 📝 Use Cases

### Development Server

```bash
# Quick development server
docker run --rm -p 8080:80 -v $(pwd):/app framjet/lighttpd
```

### Static Website with CGI Support

```dockerfile
FROM framjet/lighttpd
COPY website/ /app/
COPY cgi-bin/ /app/cgi-bin/
COPY lighttpd-cgi.conf /etc/lighttpd/conf.d/99-cgi.conf
```

### Reverse Proxy Setup

```yaml
version: '3.8'
services:
  app:
    image: myapp:latest
    
  web:
    image: framjet/lighttpd
    ports:
      - "80:80"
    volumes:
      - ./public:/app
      - ./lighttpd-proxy.conf:/etc/lighttpd/conf.d/99-proxy.conf
    environment:
      - WAIT_FOR_APP=app:3000#60s
    depends_on:
      - app
```

### Multi-stage Build

```dockerfile
# Build stage
FROM node:alpine AS builder
WORKDIR /build
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Runtime stage
FROM framjet/lighttpd
COPY --from=builder /build/dist /app
```

## 🔧 Available Modules

This base image includes all Lighttpd modules:

- mod_accesslog - Access logging
- mod_auth - Authentication
- mod_cgi - CGI support
- mod_deflate - Compression
- mod_dirlisting - Directory listings
- mod_openssl - SSL/TLS support
- mod_proxy - Reverse proxy
- mod_status - Server status (enabled by default)
- And many more...

## 🐳 Docker Best Practices

### Multi-stage Builds

```dockerfile
FROM framjet/lighttpd AS base
# Add your customizations

FROM base AS development
# Development-specific setup

FROM base AS production
# Production optimizations
```

### Security Considerations

```bash
# Run with read-only filesystem
docker run --read-only --tmpfs /tmp -v $(pwd)/html:/app framjet/lighttpd

# Limit resources
docker run --memory=64m --cpus=0.5 framjet/lighttpd
```

## 📊 Performance Tuning

### For High Traffic

```bash
# Increase connection limits
docker run \
  -e LIGHTTPD_MAX_CONNECTIONS=8192 \
  -v $(pwd)/high-performance.conf:/etc/lighttpd/conf.d/99-performance.conf \
  framjet/lighttpd
```

### Memory Optimization

```bash
# Run with memory limits
docker run --memory=128m --memory-swap=256m framjet/lighttpd
```

## 🤝 Related Images

- [`framjet/lighttpd-static`](../lighttpd-static/) - Minimal static-only variant
- [`framjet/nginx`](https://github.com/framjet/docker-nginx) - Alternative web server

## 📄 License

MIT License - see [LICENSE](../../LICENSE) for details.

## 🆘 Support

- 🐛 [Report Issues](https://github.com/framjet/docker-lighttpd/issues)
- 💬 [Discussions](https://github.com/framjet/docker-lighttpd/discussions)
- 📖 [Documentation](https://github.com/framjet/docker-lighttpd)
