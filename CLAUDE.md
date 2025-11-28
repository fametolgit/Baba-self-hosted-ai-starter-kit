# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **n8n Self-hosted AI Starter Kit** - a Docker Compose-based template for quickly setting up a local AI development environment. It combines n8n (low-code workflow platform), Ollama (local LLMs), Qdrant (vector database), and PostgreSQL to enable building self-hosted AI workflows.

**Core Philosophy**: This is a learning and experimentation platform, NOT production-ready infrastructure. It prioritizes simplicity over completeness and should work completely offline/locally by default.

## Running the Stack

### Initial Setup
```bash
cp .env.example .env  # Update secrets and passwords in .env
```

### Start Services by Profile

**For Nvidia GPU users:**
```bash
docker compose --profile gpu-nvidia up
```

**For AMD GPU users (Linux):**
```bash
docker compose --profile gpu-amd up
```

**For Mac/Apple Silicon or CPU-only:**
```bash
docker compose --profile cpu up
```

**For Mac users running Ollama locally (not in Docker):**
```bash
docker compose up  # Without profile
```
Then set `OLLAMA_HOST=host.docker.internal:11434` in `.env` and update the Ollama credential URL at http://localhost:5678/home/credentials

### Accessing Services
- n8n UI: http://localhost:5678/
- Ollama API: http://localhost:11434
- Qdrant: http://localhost:6333
- Demo workflow: http://localhost:5678/workflow/srOnR8PAY3u4RSwb

### Upgrading
```bash
# For GPU-Nvidia:
docker compose --profile gpu-nvidia pull
docker compose create && docker compose --profile gpu-nvidia up

# For CPU:
docker compose --profile cpu pull
docker compose create && docker compose --profile cpu up

# For Mac/Apple Silicon:
docker compose pull
docker compose create && docker compose up
```

## Architecture

### Docker Compose Structure

**Services:**
- `postgres`: PostgreSQL 16 database for n8n data persistence
- `n8n-import`: One-time service that imports demo credentials and workflows from `n8n/demo-data/`
- `n8n`: Main n8n workflow platform (port 5678)
- `qdrant`: Vector database for embeddings (port 6333)
- `ollama-cpu`, `ollama-gpu`, `ollama-gpu-amd`: Ollama LLM service with different profiles (port 11434)
- `ollama-pull-llama-*`: Init containers that download the llama3.2 model on first run

**Volumes:**
- `n8n_storage`: n8n user data and workflows
- `postgres_storage`: PostgreSQL data
- `ollama_storage`: Ollama models (shared across profiles)
- `qdrant_storage`: Vector database storage
- `./shared`: Host directory mounted to `/data/shared` in n8n container for local file access

**Networks:**
- `demo`: Internal Docker network connecting all services

### Key Configuration

**Environment Variables** (from `.env.example`):
- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`: PostgreSQL credentials
- `N8N_ENCRYPTION_KEY`: Encryption key for n8n credentials
- `N8N_USER_MANAGEMENT_JWT_SECRET`: JWT secret for n8n authentication
- `OLLAMA_HOST`: Ollama connection string (defaults to `ollama:11434`, set to `host.docker.internal:11434` for local Mac Ollama)

**n8n Configuration** (in `docker-compose.yml`):
- Uses PostgreSQL for persistence (`DB_TYPE=postgresdb`)
- Telemetry disabled (`N8N_DIAGNOSTICS_ENABLED=false`, `N8N_PERSONALIZATION_ENABLED=false`)
- Shared folder mounted for local file access

### Demo Data

Demo workflows and credentials are stored in `n8n/demo-data/`:
- `workflows/`: Pre-configured n8n workflows (JSON)
- `credentials/`: Pre-configured credentials for Ollama, Qdrant, etc. (JSON)

Imported on first run by the `n8n-import` service.

## File System Access in n8n

The `./shared` directory on the host is mounted to `/data/shared` inside the n8n container. When using n8n nodes that interact with the filesystem (Read/Write Files, Local File Trigger, Execute Command), use the path `/data/shared`.

## Development Guidelines

### What Belongs in This Starter Kit
- Core component configurations (n8n, Ollama, Qdrant, PostgreSQL)
- Basic Docker Compose profiles for different hardware
- Essential environment variables with sensible defaults
- Demo workflows showcasing AI capabilities
- Simple documentation for getting started

### What Does NOT Belong
- Production infrastructure (reverse proxies, SSL/TLS, load balancers, monitoring)
- Advanced networking or security hardening
- Alternative technology stacks or multiple backend options
- Enterprise features (auth systems, multi-tenancy, access controls)

### Contribution Requirements
- **Small PRs only** - focus on a single feature or fix per PR
- **No typo-only PRs** - insufficient justification
- Maintain simplicity - this is a learning platform, not a production system
- Keep everything working offline/locally by default
- Follow the vision: "fastest path from zero to working AI workflows"

## Common Tasks

### Stop all services
```bash
docker compose down
```

### View logs
```bash
docker compose logs -f [service-name]
```

### Remove all data and start fresh
```bash
docker compose down -v  # Warning: deletes all volumes
```

### Pull a different Ollama model
```bash
docker exec -it ollama ollama pull <model-name>
```

### Access n8n CLI
```bash
docker exec -it n8n n8n [command]
```
