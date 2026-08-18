# Docker for Node.js

Simple Node.js + Express API with PostgreSQL running in Docker.

## Run

Start the application:

```bash
docker compose up -d
```

Wait until the API container becomes healthy:

```bash
docker compose ps
```

API is available at:

```text
http://localhost:3000
```

Health check:

```bash
curl http://localhost:3000/health
```

Users:

```bash
curl http://localhost:3000/users
```

Stop containers:

```bash
docker compose down
```

## Development

The development configuration uses `docker-compose.override.yml` with a bind mount and hot reload.

The dev configuration uses a separate image built from the `builder` stage, so `tsx` and other dev dependencies are available.

Run:

```bash
docker compose up --build -d
```

Changes inside `src/` are available in the container without rebuilding the image after the initial build.

## Production-like run

Run only the base compose file without the development override:

```bash
docker compose -f docker-compose.yml up --build -d
```

## PostgreSQL persistence

PostgreSQL data is stored in a named Docker volume.

I checked persistence by adding a user:

```bash
docker compose exec postgres psql -U postgres -d app
```

```sql
INSERT INTO users (name)
VALUES ('Persistence Test');
```

Then I restarted the stack without deleting volumes:

```bash
docker compose down
docker compose up -d
```

The record was still present after restart.

To fully recreate PostgreSQL from `init.sql`, including a new empty volume:

```bash
docker compose down -v
docker compose up -d
```

## Docker image sizes

Single-stage image:

```text
340 MB
```

Final multi-stage image:

```text
245 MB
```

The multi-stage image is smaller because the final stage contains only production dependencies and compiled JavaScript, while the single-stage image also keeps development dependencies and build tools.

## Image size comparison

Build the single-stage image:

```bash
docker build -f Dockerfile.single -t node-api-single .
```

Check its size:

```bash
docker images node-api-single
```

Build the final multi-stage image:

```bash
docker build -t node-api-final .
```

Check its size:

```bash
docker images node-api-final
```

## Non-root user

Check the user inside the production container:

```bash
docker compose -f docker-compose.yml exec api id -u
```

The returned value must not be `0`.

## Healthcheck

Check container health:

```bash
docker compose ps
```

After startup, PostgreSQL and API should have `healthy` status.

You can also inspect API health directly:

```bash
docker inspect --format '{{.State.Health.Status}}' rbdrm-nest-js-docker-for-nodejs-api-1
```

## Clean start check

To check that the project starts correctly from a clean state:

```bash
docker compose down -v
docker compose up --build -d
```

Wait until the containers become healthy:

```bash
docker compose ps
```

Wait until both containers have `healthy` status before running curl commands.

Then check the endpoints:

```bash
curl http://localhost:3000/health
curl http://localhost:3000/users
```
