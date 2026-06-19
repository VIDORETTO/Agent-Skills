# Agent Instructions - Production Environment

## Environment

- **Location**: Running inside a VPS (Hostinger) with production services
- **Project**: MeuControleML - SaaS application
- **Path**: `/root/saas/meucontroleml`
- **Stack**: Node.js backend + Next.js frontend + Docker + Nginx + MySQL

## Critical Rules

1. **NEVER deploy or restart services without explicit user confirmation**
2. **Always check `docker-compose.yml` and running containers before making changes**
3. **Backup important files before editing production configs**
4. **Test changes in non-destructive way when possible**
5. **Check disk usage, memory, and logs before operations that affect the server**

## Before Starting Any Task

1. Read this file (AGENTS.md)
2. Understand the current state: `docker ps`, `df -h`, `free -m`
3. Ask user for confirmation before any destructive action
4. Use the `vps-production-agent` skill for production operations

## Project Structure

- `backend/` - API server (Node.js)
- `frontend/` - Web interface (Next.js)
- `deploy/` - Deployment configs
- `scripts/` - Utility scripts
- `docker-compose.yml` - Container orchestration

## Safety

- This is a LIVE production environment with real users
- Downtime affects actual business operations
- When in doubt, ASK the user before proceeding
