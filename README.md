# Bangla Blood

Production-ready MERN platform for Bangladesh blood donation management with role-based access, location hierarchy, donor workflows, reporting, and notifications.

## Documentation Index

- Setup instructions: docs/setup.md
- API reference: docs/api.md
- User roles and access model: docs/roles-and-access.md
- Deployment guide: docs/deployment.md

## Quick Start

1. Install dependencies:

   ```bash
   npm install
   ```

2. Create environment files:

   ```bash
   copy server/.env.example server/.env
   copy client/.env.example client/.env
   ```

3. Run development servers:

   ```bash
   npm run dev
   ```

## Key Tech

- Frontend: React + Vite
- Backend: Express + MongoDB
- Security: JWT, Helmet, CORS, rate limiting
- CI: GitHub Actions
- Deployment: Docker Compose and Node runtime options
