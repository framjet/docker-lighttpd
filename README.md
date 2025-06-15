# FramJet Lighttpd Docker Images

[![Build Status](https://github.com/framjet/docker-lighttpd/actions/workflows/build-images.yaml/badge.svg)](https://github.com/framjet/docker-lighttpd/actions/workflows/build-images.yaml)
[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-framjet-blue)](https://hub.docker.com/u/framjet)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

High-performance, lightweight Lighttpd web server Docker images optimized for different use cases. Built on Alpine Linux for minimal footprint and maximum efficiency.

## 🚀 Available Images

| Image                                                                         | Description                      | Size    | Use Case                               |
|-------------------------------------------------------------------------------|----------------------------------|---------|----------------------------------------|
| [`framjet/lighttpd`](https://hub.docker.com/r/framjet/lighttpd)               | Base image with full feature set | ~14MB   | Development, full-featured deployments |
| [`framjet/lighttpd-static`](https://hub.docker.com/r/framjet/lighttpd-static) | Minimal static-only server       | ~11.7MB | Static websites, CDN origins, SPAs     |

## ⭐ Key Features

- **🪶 Lightweight**: Built on Alpine Linux for minimal footprint
- **⚡ High Performance**: Optimized Lighttpd configuration for static content
- **🔒 Secure**: Minimal attack surface with unnecessary modules removed (static variant)
- **🐳 Container-Ready**: Built for Docker/Kubernetes environments
- **🔧 Customizable**: Support for custom entrypoint scripts and configuration
- **📊 Monitoring**: Built-in health check endpoint at `/status`
- **⏱️ Dependency Management**: Built-in wait4x support for service dependencies

## 🚀 Quick Start

### Basic Static Website

```bash
# Using the base image
docker run -d -p 8080:80 -v $(pwd)/html:/app framjet/lighttpd

# Using the static-only image (recommended for static content)
docker run -d -p 8080:80 -v $(pwd)/html:/app framjet/lighttpd-static
```

### Docker Compose

```yaml
version: '3.8'
services:
  web:
    image: framjet/lighttpd-static
    ports:
      - "8080:80"
    volumes:
      - ./html:/app
    environment:
      - HTTP_ROOT=/app
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/status"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: static-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: static-web
  template:
    metadata:
      labels:
        app: static-web
    spec:
      containers:
      - name: web
        image: framjet/lighttpd-static
        ports:
        - containerPort: 80
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
        volumeMounts:
        - name: static-content
          mountPath: /app
      volumes:
      - name: static-content
        configMap:
          name: static-website
```

## 📁 Configuration

### Environment Variables

| Variable                       | Default | Description                                         |
|--------------------------------|---------|-----------------------------------------------------|
| `HTTP_ROOT`                    | `/app`  | Document root directory                             |
| `WAIT_FOR_*`                   | -       | Wait for TCP services (format: `host:port#timeout`) |
| `WAIT_CMD_FOR_*`               | -       | Custom wait4x commands                              |
| `DOCKER_ENTRYPOINT_QUIET_LOGS` | -       | Suppress entrypoint logs if set                     |

### Custom Configuration

You can customize the server configuration by mounting your own config files:

```bash
docker run -d \
  -p 8080:80 \
  -v $(pwd)/html:/app \
  -v $(pwd)/lighttpd.conf:/etc/lighttpd/lighttpd.conf \
  framjet/lighttpd
```

### Custom Entrypoint Scripts

Add custom initialization scripts to `/docker-entrypoint.d/`:

```bash
# Mount custom script
docker run -d \
  -p 8080:80 \
  -v $(pwd)/html:/app \
  -v $(pwd)/my-init.sh:/docker-entrypoint.d/10-my-init.sh \
  framjet/lighttpd
```

Supported script types:
- `*.sh` - Executable shell scripts
- `*.envsh` - Sourced environment scripts

## 🔄 Service Dependencies

Use environment variables to wait for dependent services:

```bash
# Wait for database to be ready
docker run -e WAIT_FOR_DB=postgres:5432#60s framjet/lighttpd

# Multiple dependencies
docker run \
  -e WAIT_FOR_DB=postgres:5432#60s \
  -e WAIT_FOR_CACHE=redis:6379#30s \
  framjet/lighttpd
```

## 📊 Health Monitoring

All images include a health check endpoint at `/status`:

```bash
# Check server status
curl http://localhost:8080/status

# Use in health checks
wget --quiet --tries=1 --spider http://localhost/status
```

## 🏗️ Building from Source

```bash
# Clone the repository
git clone https://github.com/framjet/docker-lighttpd.git
cd docker-lighttpd

# Build base image
docker build -t my-lighttpd images/lighttpd/

# Build static-only image
docker build -t my-lighttpd-static images/lighttpd-static/
```

## 📝 Examples

### Static Website with SSL Termination (via Reverse Proxy)

```yaml
version: '3.8'
services:
  web:
    image: framjet/lighttpd-static
    volumes:
      - ./website:/app
    
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl
    depends_on:
      - web
```

### SPA with History API Support

```dockerfile
FROM framjet/lighttpd-static
COPY dist/ /app/
COPY lighttpd-spa.conf /etc/lighttpd/conf.d/99-spa.conf
```

### CDN Origin Server

```bash
docker run -d \
  --name cdn-origin \
  -p 8080:80 \
  -v /var/www/static:/app:ro \
  --restart unless-stopped \
  framjet/lighttpd-static
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🏷️ Tags and Versioning

- `latest` - Latest stable release
- `x.y.z` - Specific version tags
- Multi-architecture support: `linux/amd64`, `linux/arm64`

## 🆘 Support

- 📖 [Documentation](https://github.com/framjet/docker-lighttpd)
- 🐛 [Issue Tracker](https://github.com/framjet/docker-lighttpd/issues)
- 💬 [Discussions](https://github.com/framjet/docker-lighttpd/discussions)

---

**FramJet** - Building efficient, production-ready Docker images.
