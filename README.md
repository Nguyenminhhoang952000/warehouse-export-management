# Warehouse Workspace

## Projects
- `warehouse-management-be`: NestJS backend
- `warehouse-management-fe`: React + Vite frontend

## Local Development (Docker + Hot Reload)

### 1) Start local stack (DB + Backend in Docker)
```bash
docker compose -f docker-compose.local.yml up -d --build
```

### 2) Run backend commands inside container
```bash
docker compose -f docker-compose.local.yml exec be npm run lint
docker compose -f docker-compose.local.yml exec be npm run test
docker compose -f docker-compose.local.yml exec be npm run build
docker compose -f docker-compose.local.yml exec be npm run db:migrate
docker compose -f docker-compose.local.yml exec be npm run db:seed
```

### 3) Run frontend outside Docker
```bash
cd warehouse-management-fe
npm install
npm run dev
```

- FE URL: `http://localhost:5173`
- BE URL: `http://localhost:3000/api`

### 4) Stop local stack
```bash
docker compose -f docker-compose.local.yml down
```

## Production Stack (Docker Compose)

### 1) Prepare env
```bash
cp .env.example .env
# edit .env values
```

### 2) Build and start
```bash
docker compose -f docker-compose.prod.yml --env-file .env up -d --build
```

### 3) Services
- Nginx (serves FE static and proxies `/api`): `http://localhost`
- Backend internal upstream: `be:3000`
- Postgres internal host: `db:5432`

### 4) Stop production stack
```bash
docker compose -f docker-compose.prod.yml down
```

## Notes
- Local backend uses hot reload via `npm run start:dev` in container.
- Backend API contract uses envelope: `success`, `message`, `data`, `error`, `meta`.
- Sort query format: `sort=field:dir`.

## Database Lifecycle (Backend)

Run in `warehouse-management-be`:

```bash
npm run db:migrate
npm run db:revert
npm run db:seed
```

- Migration datasource: `src/database/typeorm.config.ts`
- Initial schema migration: `src/database/migrations/1714200000000-init-schema.ts`
- Seed script: `src/database/seeds/seed.ts`
