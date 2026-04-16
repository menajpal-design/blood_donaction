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

Minimum production server values:

```env
NODE_ENV=production
PORT=5000
MONGODB_URI=mongodb+srv://<user>:<password>@<cluster>/bangla-blood
CLIENT_URL=https://your-frontend-domain.com
JWT_SECRET=<use-strong-random-32+char-secret>
JWT_EXPIRES_IN=7d
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=200
```

3. Start backend:

```bash
npm run start
```

4. Serve client dist via web server (Nginx, Caddy, or static hosting).

Client build must point to your backend base URL:

```env
VITE_API_BASE_URL=https://your-api-domain.com/api/v1
```

## Option B: Docker Compose

1. Prepare env files:

```bash
# Windows PowerShell
copy server/.env.example server/.env
copy client/.env.example client/.env

# Linux/macOS
cp server/.env.example server/.env
cp client/.env.example client/.env
```

2. Build and start containers:

```bash
docker compose up --build -d
```

Container defaults in this project:

- Server uses `NODE_ENV=production`
- Server Mongo connection is forced to `mongodb://mongo:27017/bangla-blood`
- Client build arg defaults to `VITE_API_BASE_URL=/api/v1`
- Nginx proxies `/api/*` to the server container (`server:5000`)
- Healthchecks are enabled for Mongo, server, and client

Useful overrides at deploy time:

```bash
set CLIENT_URL=https://your-frontend-domain.com
set VITE_API_BASE_URL=/api/v1
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

Health endpoints:

- Client container: `GET /health`
- Server container: `GET /api/v1/health`

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
- Disable public MongoDB port exposure in production unless absolutely required.
- Monitor logs and request metrics.
- Configure process manager (PM2/systemd) if not using containers.

## Scaling Notes

- Keep API stateless for horizontal scaling.
- Move in-memory cache to Redis when scaling beyond one instance.
- Add index tuning based on real query patterns.
- Add CDN for frontend assets.
- Add background workers for heavy report generation.
