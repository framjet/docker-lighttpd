# framjet/lighttpd-static

[![Docker Hub](https://img.shields.io/docker/pulls/framjet/lighttpd-static)](https://hub.docker.com/r/framjet/lighttpd-static)
[![Docker Image Size](https://img.shields.io/docker/image-size/framjet/lighttpd-static/latest)](https://hub.docker.com/r/framjet/lighttpd-static)

Ultra-minimal Lighttpd web server Docker image optimized exclusively for serving static files. Built from the `framjet/lighttpd` base with all dynamic modules removed for maximum performance and minimal attack surface.

## ⚡ Features

- **🪶 Ultra-Lightweight**: Minimal footprint (~11.7MB)
- **⚡ Maximum Performance**: Optimized for static content only
- **🔒 Secure**: Reduced attack surface with dynamic modules removed
- **🎯 Static-Only**: No CGI, SSL, proxy, or dynamic content support
- **📊 Health Monitoring**: Built-in `/status` endpoint (mod_status only)
- **🔄 Service Dependencies**: wait4x support via environment variables
- **⚙️ Customizable**: Custom entrypoint scripts support
- **🐳 Container-Ready**: Perfect for Docker/Kubernetes static deployments

## 🚫 What's Removed

This image has the following modules and libraries removed for minimal footprint:

### Removed Modules
- mod_accesslog, mod_ajp13, mod_authn_*
- mod_cgi, mod_deflate, mod_dirlisting
- mod_openssl, mod_proxy, mod_ssi
- mod_userdir, mod_vhostdb_*, mod_wstunnel
- And many more dynamic modules...

### Removed Libraries
- SSL/TLS libraries (libssl, libcrypto)
- Lua libraries (liblua)
- Compression libraries (libzstd, libz)
- LDAP libraries (libldap, liblber)
- Database libraries (libdbi)

## ✅ What's Included

- **Core Lighttpd server** for static file serving
- **mod_status** for health checks and monitoring
- **MIME type support** for all common static file types
- **Custom entrypoint scripts** support
- **wait4x integration** for service dependencies

## 🚀 Quick Start

### Basic Usage

```bash
# Serve static files from current directory
docker run -d -p 8080:80 -v $(pwd)/public:/app framjet/lighttpd-static

# Serve with custom index files
docker run -d -p 8080:80 \
  -v $(pwd)/dist:/app \
  framjet/lighttpd-static
```

### Docker Compose

```yaml
version: '3.8'
services:
  static-web:
    image: framjet/lighttpd-static
    ports:
      - "8080:80"
    volumes:
      - ./dist:/app:ro  # Read-only for security
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/status"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
```

## 🎯 Perfect Use Cases

### Static Websites

```bash
# Jekyll/Hugo/Next.js build output
docker run -d -p 80:80 -v ./dist:/app framjet/lighttpd-static
```

### Single Page Applications (SPAs)

```dockerfile
FROM node:alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM framjet/lighttpd-static
COPY --from=builder /app/dist /app
```

### CDN Origin Server

```bash
# Serve static assets for CDN
docker run -d \
  --name cdn-origin \
  -p 8080:80 \
  -v /var/www/static:/app:ro \
  --restart unless-stopped \
  --memory=64m \
  framjet/lighttpd-static
```

### Documentation Sites

```yaml
version: '3.8'
services:
  docs:
    image: framjet/lighttpd-static
    ports:
      - "3000:80"
    volumes:
      - ./docs/_site:/app:ro
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.docs.rule=Host(`docs.example.com`)"
```

## 📁 Directory Structure

```
/app/                    # Document root (static files only)
├── index.html          # Default welcome page
├── assets/             # CSS, JS, images
├── css/
├── js/
├── images/
└── ...                 # Your static content

/etc/lighttpd/          # Minimal configuration
├── lighttpd.conf       # Main configuration
└── conf.d/             # Core configurations only
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

### MIME Types

The image includes comprehensive MIME type support for static files:

```
.html, .css, .js       → text/html, text/css, application/javascript
.png, .jpg, .gif, .svg → image/* types
.woff, .woff2, .ttf    → font/* types
.json, .xml            → application/json, text/xml
.pdf, .zip, .mp4       → application/*, video/* types
```

## 🔄 Service Dependencies

Wait for backend services before serving static files:

```bash
# Wait for API server to be ready
docker run \
  -e WAIT_FOR_API=api-server:8080#60s \
  -v $(pwd)/dist:/app \
  framjet/lighttpd-static

# Wait for multiple services
docker run \
  -e WAIT_FOR_DB=postgres:5432#30s \
  -e WAIT_FOR_CACHE=redis:6379#30s \
  -v $(pwd)/public:/app \
  framjet/lighttpd-static
```

## 📊 Performance & Monitoring

### Health Checks

```bash
# Server status endpoint
curl http://localhost:8080/status

# Docker health check
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/status || exit 1
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: static-website
spec:
  replicas: 3
  selector:
    matchLabels:
      app: static-website
  template:
    metadata:
      labels:
        app: static-website
    spec:
      containers:
      - name: web
        image: framjet/lighttpd-static
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"
        livenessProbe:
          httpGet:
            path: /status
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /status
            port: 80
          initialDelaySeconds: 2
          periodSeconds: 5
        volumeMounts:
        - name: static-content
          mountPath: /app
          readOnly: true
      volumes:
      - name: static-content
        configMap:
          name: website-content
```

## 🏗️ Build Examples

### React App

```dockerfile
# Build stage
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM framjet/lighttpd-static
COPY --from=builder /app/build /app
EXPOSE 80
```

### Vue.js App

```dockerfile
FROM node:alpine AS build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM framjet/lighttpd-static AS production-stage
COPY --from=build-stage /app/dist /app
```

### Static Site Generator

```dockerfile
# Hugo build
FROM klakegg/hugo:alpine AS hugo
COPY . /src
WORKDIR /src
RUN hugo --minify

# Serve with lighttpd-static
FROM framjet/lighttpd-static
COPY --from=hugo /src/public /app
```

## 🔧 Advanced Usage

### Custom Index Files

```bash
# Support different index file names
docker run -d \
  -v $(pwd)/public:/app \
  -v $(pwd)/custom-webroot.conf:/etc/lighttpd/conf.d/02-webroot.conf \
  framjet/lighttpd-static
```

### Read-Only Container

```bash
# Maximum security with read-only filesystem
docker run -d \
  --read-only \
  --tmpfs /tmp \
  -v $(pwd)/static:/app:ro \
  framjet/lighttpd-static
```

### Resource Limits

```bash
# Minimal resource usage
docker run -d \
  --memory=32m \
  --cpus=0.1 \
  -v $(pwd)/public:/app:ro \
  framjet/lighttpd-static
```

## 📈 Performance Comparison

| Metric         | framjet/lighttpd | framjet/lighttpd-static |
|----------------|------------------|-------------------------|
| Image Size     | ~14MB            | ~11.7MB                 |
| Memory Usage   | ~16MB            | ~8MB                    |
| Attack Surface | Full modules     | Minimal                 |
| Use Case       | Dynamic content  | Static only             |

## 🚫 Limitations

This image **cannot** serve:
- ❌ CGI scripts or dynamic content
- ❌ SSL/HTTPS (use reverse proxy)
- ❌ Server-side includes (SSI)
- ❌ Directory listings
- ❌ Access logs (use reverse proxy logging)
- ❌ Compression (use reverse proxy or CDN)

## 🤝 Related Images

- [`framjet/lighttpd`](../lighttpd/README.md) - Full-featured base image
- [`framjet/nginx`](https://github.com/framjet/docker-nginx) - Alternative web server

## 📄 License

MIT License - see [LICENSE](../../LICENSE) for details.

## 🆘 Support

- 🐛 [Report Issues](https://github.com/framjet/docker-lighttpd/issues)
- 💬 [Discussions](https://github.com/framjet/docker-lighttpd/discussions)
- 📖 [Documentation](https://github.com/framjet/docker-lighttpd)
