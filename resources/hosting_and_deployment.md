# n8n Hosting and Deployment

This document covers self-hosting n8n including installation, configuration, and scaling.

## Installation Methods

### Docker (Recommended)

```bash
# Quick start
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

### Docker Compose

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=${DOMAIN}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://${DOMAIN}/
      - GENERIC_TIMEZONE=${TIMEZONE}
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
    driver: local
```

### npm Installation

```bash
# Global install
npm install n8n -g

# Start n8n
n8n start

# With tunnel (for development)
n8n start --tunnel
```

### npx (No Install)

```bash
npx n8n
```

---

## Database Configuration

### SQLite (Default)

No configuration needed. Data stored in `~/.n8n/database.sqlite`.

### PostgreSQL (Recommended for Production)

```bash
# Environment variables
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=secure_password
DB_POSTGRESDB_SCHEMA=public
```

### MySQL/MariaDB

```bash
DB_TYPE=mysqldb
DB_MYSQLDB_HOST=localhost
DB_MYSQLDB_PORT=3306
DB_MYSQLDB_DATABASE=n8n
DB_MYSQLDB_USER=n8n
DB_MYSQLDB_PASSWORD=secure_password
```

---

## Environment Variables

### Core Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `N8N_HOST` | Host/domain | localhost |
| `N8N_PORT` | Port number | 5678 |
| `N8N_PROTOCOL` | http or https | http |
| `WEBHOOK_URL` | Public webhook URL | - |
| `N8N_ENCRYPTION_KEY` | 32-char encryption key | auto-generated |
| `GENERIC_TIMEZONE` | Timezone | America/New_York |

### Authentication

| Variable | Description | Default |
|----------|-------------|---------|
| `N8N_BASIC_AUTH_ACTIVE` | Enable basic auth | false |
| `N8N_BASIC_AUTH_USER` | Username | - |
| `N8N_BASIC_AUTH_PASSWORD` | Password | - |

### Execution

| Variable | Description | Default |
|----------|-------------|---------|
| `EXECUTIONS_MODE` | queue | regular |
| `EXECUTIONS_TIMEOUT` | Max execution time | -1 (no limit) |
| `EXECUTIONS_TIMEOUT_MAX` | Max configurable timeout | 3600 |
| `EXECUTIONS_DATA_SAVE_ON_ERROR` | Save failed executions | all |
| `EXECUTIONS_DATA_SAVE_ON_SUCCESS` | Save successful executions | all |
| `EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS` | Save manual runs | true |

### Logging

| Variable | Description | Options |
|----------|-------------|---------|
| `N8N_LOG_LEVEL` | Log verbosity | info, warn, error, debug |
| `N8N_LOG_OUTPUT` | Log destination | console, file |
| `N8N_LOG_FILE_LOCATION` | Log file path | - |

### Performance

| Variable | Description | Default |
|----------|-------------|---------|
| `N8N_PAYLOAD_SIZE_MAX` | Max request size | 16 (MB) |
| `N8N_MEMORY` | Memory limit | - |

---

## User Management

### Owner Setup

First user to sign up becomes the owner.

### User Roles

| Role | Permissions |
|------|-------------|
| **Owner** | Full access, manage users |
| **Admin** | Full access, no user management |
| **Member** | Limited access |

### Environment Variables

```bash
# Email configuration for invites
N8N_EMAIL_MODE=smtp
N8N_SMTP_HOST=smtp.example.com
N8N_SMTP_PORT=587
N8N_SMTP_USER=user@example.com
N8N_SMTP_PASS=password
N8N_SMTP_SENDER=n8n@example.com
```

---

## Scaling

### Queue Mode

For high-volume production deployments.

#### Architecture

```
                    ┌─────────────┐
                    │   Webhook   │
                    │  Processor  │
                    └──────┬──────┘
                           │
┌─────────────┐     ┌──────▼──────┐     ┌─────────────┐
│   Main      │────▶│    Redis    │◀────│   Worker    │
│  Instance   │     │   Queue     │     │   Process   │
└─────────────┘     └─────────────┘     └─────────────┘
                           ▲
                           │
                    ┌──────┴──────┐
                    │   Worker    │
                    │   Process   │
                    └─────────────┘
```

#### Configuration

```bash
# Main/UI instance
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=redis
QUEUE_BULL_REDIS_PORT=6379

# Worker instance
N8N_WORKER=true
QUEUE_BULL_REDIS_HOST=redis
QUEUE_BULL_REDIS_PORT=6379
```

#### Docker Compose for Queue Mode

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    environment:
      - EXECUTIONS_MODE=queue
      - QUEUE_BULL_REDIS_HOST=redis
    ports:
      - "5678:5678"
    depends_on:
      - postgres
      - redis

  n8n-worker:
    image: docker.n8n.io/n8nio/n8n
    command: worker
    environment:
      - EXECUTIONS_MODE=queue
      - QUEUE_BULL_REDIS_HOST=redis
    depends_on:
      - redis

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data

  postgres:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=secure
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  redis_data:
  postgres_data:
```

### Webhook Processing

Separate webhook handling:

```bash
# Main instance
N8N_DISABLE_PRODUCTION_MAIN_PROCESS=true

# Webhook instance
N8N_WEBHOOK=true
```

---

## Security

### SSL/TLS

```yaml
# Docker with Traefik
services:
  traefik:
    image: traefik:v2.9
    command:
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.email=your@email.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "./letsencrypt:/letsencrypt"

  n8n:
    image: docker.n8n.io/n8nio/n8n
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.n8n.rule=Host(`n8n.example.com`)"
      - "traefik.http.routers.n8n.entrypoints=websecure"
      - "traefik.http.routers.n8n.tls.certresolver=letsencrypt"
```

### SSO (Enterprise)

SAML/LDAP configuration for enterprise.

### Two-Factor Authentication

Enable 2FA in user settings.

### IP Allowlisting

```bash
N8N_ALLOW_ORIGIN=https://example.com
```

---

## Backup and Recovery

### Backup Workflows

```bash
# Export via CLI
n8n export:workflow --all --output=./backups/

# Export specific workflow
n8n export:workflow --id=123 --output=./backups/
```

### Backup Credentials

```bash
# Export credentials
n8n export:credentials --all --output=./backups/
```

### Database Backup

```bash
# PostgreSQL
pg_dump -U n8n -d n8n > backup.sql

# SQLite
cp ~/.n8n/database.sqlite backup.sqlite
```

### Restore

```bash
# Import workflows
n8n import:workflow --input=./backups/

# Import credentials
n8n import:credentials --input=./backups/
```

---

## Monitoring

### Health Check

```bash
curl http://localhost:5678/healthz
```

### Metrics (Enterprise)

Prometheus metrics endpoint available in Enterprise.

### Logging

```bash
# Enable debug logs
N8N_LOG_LEVEL=debug

# File logging
N8N_LOG_OUTPUT=file
N8N_LOG_FILE_LOCATION=/var/log/n8n/
```

---

## Deployment Guides

### DigitalOcean

1. Create Droplet with Docker
2. Install Docker Compose
3. Deploy with compose file
4. Configure domain and SSL

### AWS

1. EC2 instance with Docker
2. RDS for PostgreSQL
3. ElastiCache for Redis
4. Load Balancer for scaling

### Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: n8n
spec:
  replicas: 1
  selector:
    matchLabels:
      app: n8n
  template:
    metadata:
      labels:
        app: n8n
    spec:
      containers:
        - name: n8n
          image: docker.n8n.io/n8nio/n8n
          ports:
            - containerPort: 5678
          env:
            - name: N8N_PROTOCOL
              value: https
          volumeMounts:
            - name: n8n-data
              mountPath: /home/node/.n8n
      volumes:
        - name: n8n-data
          persistentVolumeClaim:
            claimName: n8n-pvc
```

---

## Best Practices

### Production Checklist

- [ ] Use PostgreSQL instead of SQLite
- [ ] Set encryption key (backup it securely)
- [ ] Enable SSL/TLS
- [ ] Configure authentication
- [ ] Set up regular backups
- [ ] Enable queue mode for scale
- [ ] Monitor with health checks
- [ ] Configure execution timeouts
- [ ] Set appropriate log levels
