# Docker Standards

## Image Rules
- Always use a specific version tag — never `latest`
- Multi-stage builds to minimize final image size
- Final stage: non-root user

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS runtime
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --chown=app:app . .
USER app
CMD ["node", "index.js"]
```

## Layer Ordering
Order Dockerfile instructions from least-to-most-frequently-changed:
1. Base image
2. System dependencies (`apt-get`, `apk add`)
3. Package files (`package.json`, `requirements.txt`)
4. `npm install` / `pip install`
5. Application source code

This maximizes cache hits.

## Security
- No secrets in Dockerfile or image — use runtime env vars or secrets mounts
- Run as non-root user in final stage
- Scan images with `docker scout` or equivalent before pushing
- `.dockerignore` must exclude: `.git`, `node_modules`, `.env`, credentials

## Compose (Dev Only)
- `docker-compose.yml` for local development only — never for production deploys
- Use named volumes for persistent data
- Services should declare `depends_on` with health check conditions
