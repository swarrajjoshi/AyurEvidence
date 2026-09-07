# Run AyurEvidence on Windows

Use **Windows PowerShell**. WSL commands are not required after Docker Desktop is installed and running.

## First start

1. Start Docker Desktop and wait until it reports that Docker Engine is running.

2. Open PowerShell and enter the exact Desktop project folder:

```powershell
Set-Location -LiteralPath "C:\Users\Swarraj\OneDrive\Desktop\SIH AyurEvidence"
```

3. Create the local environment file once:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

4. Build and start all services in the background:

```powershell
docker compose up --build -d
```

5. Confirm that all three containers are `Up` and Neo4j is `healthy`:

```powershell
docker compose ps
```

6. Load the bundled DEMO knowledge-graph data:

```powershell
docker compose exec -T backend python scripts/ingest_data.py
```

7. Open:

- AyurEvidence: http://localhost:3000
- API documentation: http://localhost:8000/docs
- API health: http://localhost:8000/api/health
- Neo4j Browser: http://localhost:7474

Neo4j credentials are `neo4j` and the `NEO4J_PASSWORD` value in `.env`. The default is `ayurevidence-demo`.

## Normal start on later days

```powershell
Set-Location -LiteralPath "C:\Users\Swarraj\OneDrive\Desktop\SIH AyurEvidence"
docker compose up -d
docker compose ps
```

## Stop the project

```powershell
docker compose down
```

This preserves the Neo4j volume and uploaded-document storage.

## Rebuild after changing code

```powershell
docker compose down
docker compose up --build -d
docker compose ps
```

## Inspect errors

```powershell
docker compose logs --tail=100 frontend
docker compose logs --tail=100 backend
docker compose logs --tail=100 neo4j
```

To follow logs continuously, use `docker compose logs -f` and press `Ctrl+C` to stop viewing logs; the containers keep running.

## Important notes

- Run commands from the folder containing `docker-compose.yml`.
- Do not merge `COPY . .` into the `pnpm install` line in `Dockerfile.frontend`.
- Do not run `pnpm approve-builds` in PowerShell. Docker reads the reviewed package list from `pnpm-workspace.yaml`.
- `.dockerignore` prevents copied `node_modules`, `.venv`, caches, and runtime storage from entering the Docker build context.
- Do not use `docker compose down -v` unless you intentionally want to delete the Neo4j database volume.
