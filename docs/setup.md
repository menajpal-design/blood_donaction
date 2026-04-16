# Setup Guide

## Requirements

- Node.js 20+
- npm 10+
- MongoDB 7+ (local or remote)
- Docker (optional for containerized setup)

## Project Structure

```text
bangla-blood/
  client/              # React + Vite frontend
  server/              # Express + MongoDB backend
  docs/                # Project documentation
  package.json         # Root workspaces and scripts
  docker-compose.yml   # Multi-service local deployment
```

## Install

```bash
npm install
```

## Environment Files

Create environment files from examples:

```bash
# Windows PowerShell
copy server/.env.example server/.env
copy client/.env.example client/.env

# Linux/macOS
cp server/.env.example server/.env
cp client/.env.example client/.env
```

### Server Environment Variables

- NODE_ENV: development | test | production
- PORT: server port (default 5000)
- MONGODB_URI: MongoDB connection string
- CLIENT_URL: allowed client origin for CORS
- JWT_SECRET: JWT signing secret (min 32 chars)
- JWT_EXPIRES_IN: token lifetime (example: 7d)
- RATE_LIMIT_WINDOW_MS: API rate limit window
- RATE_LIMIT_MAX_REQUESTS: max requests per window

### Client Environment Variables

- VITE_APP_NAME: app display name
- VITE_API_BASE_URL: backend base URL
- VITE_WATERMARK_TEXT: watermark brand text
- VITE_WATERMARK_TAGLINE: watermark subtitle
- VITE_WATERMARK_POSITION: top-left | top-right | bottom-left | bottom-right | center
- VITE_WATERMARK_OPACITY: watermark opacity (0.0 to 1.0)
- VITE_WATERMARK_COLOR: watermark color value

## Run in Development

```bash
npm run dev
```

This starts both apps:

- Client: http://localhost:5173
- Server: http://localhost:5000

## Seed Bangladesh Locations

Populate the full Bangladesh location hierarchy (Division, District, Upazila, Union/Pouroshava):

```bash
npm run seed:locations --workspace server
```

## Lint and Build

```bash
npm run lint
npm run build
```

## Start Server in Production Mode

```bash
npm run start
```

## Troubleshooting

- JWT errors on startup:
  - Verify JWT_SECRET and JWT_EXPIRES_IN in server/.env.
- CORS errors from browser:
  - Ensure CLIENT_URL matches frontend origin exactly.
- Mongo connection errors:
  - Verify MONGODB_URI and database availability.
- Missing API base URL in client:
  - Ensure VITE_API_BASE_URL exists in client/.env.
