# Deployment Guide

## Deployment Options

- Option A: Node + Mongo deployment
- Option B: Docker Compose deployment
- Option C: CI pipeline with GitHub Actions

## Option A: Node + Mongo

1. Build frontend and backend dependencies:

```bash
npm install
npm run build
```

2. Set production environment values in server/.env.

3. Start backend:

```bash
npm run start
```

4. Serve client dist via web server (Nginx, Caddy, or static hosting).

## Option B: Docker Compose

1. Prepare env files:

```bash
copy server/.env.example server/.env
copy client/.env.example client/.env
```

2. Build and start containers:

```bash
docker compose up --build -d
```

3. Stop services:

```bash
docker compose down
```

Default ports:

- Client: 8080
- Server: 5000
- MongoDB: 27017

## Option C: CI with GitHub Actions

The workflow file is at:

- .github/workflows/ci.yml

Pipeline steps:

- checkout
- setup node 20
- install dependencies
- lint
- build

## Production Hardening Checklist

- Use strong JWT_SECRET.
- Set NODE_ENV=production.
- Restrict CLIENT_URL to real frontend domain.
- Use managed MongoDB with backups.
- Enable HTTPS at load balancer/reverse proxy.
- Monitor logs and request metrics.
- Configure process manager (PM2/systemd) if not using containers.

## Scaling Notes

- Keep API stateless for horizontal scaling.
- Move in-memory cache to Redis when scaling beyond one instance.
- Add index tuning based on real query patterns.
- Add CDN for frontend assets.
- Add background workers for heavy report generation.
