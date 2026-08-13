# Docker for Node.js

Simple Node.js + Express API with PostgreSQL running in Docker.

## Run

Start the application:

```bash
docker compose up -d
```

Check running containers:

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

Run:

```bash
docker compose up --build
```

Changes inside `src/` are available in the container without rebuilding the image.

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

## Docker image sizes

Builder image:

```text
340 MB
```

Final image:

```text
245 MB
```

The final image is smaller because it contains only production dependencies and compiled JavaScript, while the builder also contains TypeScript and development dependencies.

## Build stages

Build the final image:

```bash
docker build -t node-docker-api .
```

Build only the builder stage:

```bash
docker build --target builder -t node-docker-api-builder .
```

Check image sizes:

```bash
docker images node-docker-api
docker images node-docker-api-builder
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
docker compose -f docker-compose.yml ps
```

The API container should have `healthy` status.
