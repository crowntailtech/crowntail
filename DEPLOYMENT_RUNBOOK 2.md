# AeroMonitor Deployment Runbook

Complete step-by-step guide to deploy AeroMonitor from scratch. This runbook covers everything needed to get the application running in a production-ready Docker environment.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Repository Setup](#repository-setup)
3. [Environment Configuration](#environment-configuration)
4. [Docker Deployment](#docker-deployment)
5. [Initial Setup & Verification](#initial-setup--verification)
6. [Accessing the Application](#accessing-the-application)
7. [Troubleshooting](#troubleshooting)
8. [Production Considerations](#production-considerations)

---

## Prerequisites

### Required Software

1. **Docker** (version 20.10+)
   ```bash
   # Verify Docker installation
   docker --version
   docker-compose --version
   ```

2. **Git** (version 2.30+)
   ```bash
   git --version
   ```

3. **GitHub Personal Access Token (PAT)**
   - Required for accessing private repositories
   - Create one at: https://github.com/settings/tokens
   - Permissions needed: `repo` (full control of private repositories)

4. **Python 3.11+** (for local development/testing only)
   ```bash
   python3 --version
   ```

### System Requirements

- **OS**: macOS, Linux, or Windows (with WSL2)
- **RAM**: Minimum 4GB, Recommended 8GB+
- **Disk Space**: At least 10GB free
- **Network**: Internet connection for Docker image pulls and GitHub access

---

## Repository Setup

AeroMonitor uses a multi-repository architecture. You'll need to clone the main docker repository which uses Git submodules.

### Step 1: Clone the Docker Repository

```bash
# Navigate to your desired directory
cd ~/Documents  # or your preferred location

# Clone the docker repository (includes submodules)
git clone --recurse-submodules https://github.com/crowntailtech/crowntail.git aero-docker

cd aero-docker
```

**If you already cloned without submodules:**

```bash
cd aero-docker
git submodule update --init --recursive
```

### Step 2: Verify Repository Structure

You should see the following structure:

```
aero-docker/
├── docker-compose.yml
├── README.md
├── STATUS.md
├── DEPLOYMENT_RUNBOOK.md (this file)
├── aero-backend-api-split/    (Git submodule)
├── aero-frontend-split/        (Git submodule)
└── packages/                   (Directory for package dependencies)
```

### Step 3: Update Submodules (if needed)

To ensure you have the latest code from all repositories:

```bash
git submodule update --remote --recursive
```

---

## Environment Configuration

### Step 1: Generate Master Encryption Key

The application requires a master encryption key for secure credential storage. Generate one:

```bash
# Option 1: Using Python
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"

# Option 2: Using Docker (if Python not available)
docker run --rm python:3.11-slim python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

**Save this key securely** - you'll need it in the next step.

### Step 2: Create Environment File

Create a `.env` file in the `aero-docker` directory:

```bash
cd aero-docker
cat > .env << 'EOF'
# Database Configuration
DATABASE_URL=postgresql+asyncpg://postgres:postgres@db:5432/aeromonitor

# Redis Configuration  
REDIS_URL=redis://redis:6379/0

# Application Secret Key (CHANGE IN PRODUCTION)
SECRET_KEY=your-secret-key-change-in-production-min-32-chars

# Master Encryption Key (REQUIRED - generated in Step 1)
MASTER_ENCRYPTION_KEY=your-generated-encryption-key-here

# OpenAI API Key (Optional - for AI log summarization)
OPENAI_API_KEY=

# GitHub Personal Access Token (for cloning private repos during Docker build)
PAT=your-github-pat-here

# Backend Port (default: 8002)
BACKEND_PORT=8002

# Frontend Port (default: 3001)
FRONTEND_PORT=3001

# Database Port (default: 5433)
DB_PORT=5433

# Redis Port (default: 6380)
REDIS_PORT=6380
EOF
```

**Important:** Replace the placeholder values:
- `MASTER_ENCRYPTION_KEY`: Use the key generated in Step 1
- `SECRET_KEY`: Generate a random 32+ character string
- `PAT`: Your GitHub Personal Access Token (if using private repos)
- `OPENAI_API_KEY`: Optional, only if using AI features

### Step 3: Verify Environment File

```bash
# Check that .env file exists and has required values
cat .env | grep -E "MASTER_ENCRYPTION_KEY|SECRET_KEY|PAT" | grep -v "^#"
```

---

## Docker Deployment

### Step 1: Build Docker Images

Build all required Docker images:

```bash
cd aero-docker
docker-compose build
```

This will:
- Build the backend API image
- Build the frontend image  
- Build the Celery worker image
- Download PostgreSQL and Redis images
- Take approximately 5-10 minutes on first run

**Troubleshooting Build Issues:**

If you encounter errors during build:
- Ensure your GitHub PAT is correct in `.env`
- Check that submodules are properly initialized
- Verify Docker has enough disk space: `docker system df`

### Step 2: Start All Services

Start all containers:

```bash
docker-compose up -d
```

This starts:
- **PostgreSQL** database (port 5433)
- **Redis** cache/broker (port 6380)
- **Backend API** (port 8002)
- **Frontend** (port 3001)
- **Celery Worker** (background tasks)

### Step 3: Check Service Status

Verify all services are running:

```bash
docker-compose ps
```

Expected output:
```
NAME                    STATUS              PORTS
aero-docker-backend-1   Up (healthy)        ...
aero-docker-celery-1    Up                  ...
aero-docker-db-1        Up (healthy)        ...
aero-docker-frontend-1  Up                  ...
aero-docker-redis-1     Up (healthy)        ...
```

### Step 4: View Logs

Monitor the startup process:

```bash
# View all logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f celery
```

**Wait for:** Backend logs should show:
```
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000
```

Frontend logs should show:
```
VITE v5.x.x  ready in XXX ms
➜  Local:   http://localhost:3000/
```

---

## Initial Setup & Verification

### Step 1: Wait for Database Migrations

The backend automatically runs database migrations on startup. Wait for:

```
INFO:     Application startup complete.
```

**Check migration status:**
```bash
docker-compose logs backend | grep -E "migration|Migration|Running upgrade"
```

### Step 2: Verify API Health

Test that the backend API is responding:

```bash
curl http://localhost:8002/
```

Expected response:
```json
{
  "message": "AeroMonitor API",
  "version": "1.0.0",
  "docs": "/docs"
}
```

**API Documentation:** http://localhost:8002/docs

### Step 3: Verify Frontend

Open in browser: **http://localhost:3001**

You should see the AeroMonitor login page.

---

## Accessing the Application

### Web Interface

- **Frontend**: http://localhost:3001
- **Backend API**: http://localhost:8002
- **API Documentation (Swagger)**: http://localhost:8002/docs
- **Alternative API Docs (ReDoc)**: http://localhost:8002/redoc

### Default Ports

| Service | Port | URL |
|---------|------|-----|
| Frontend | 3001 | http://localhost:3001 |
| Backend API | 8002 | http://localhost:8002 |
| PostgreSQL | 5433 | localhost:5433 |
| Redis | 6380 | localhost:6380 |

**Note:** Ports are configured to avoid conflicts with other applications. You can change them in `.env` and `docker-compose.yml`.

### Creating Your First Account

1. Open http://localhost:3001
2. Click "Sign Up" 
3. Enter:
   - Email: your-email@example.com
   - Password: (minimum 8 characters)
   - Name: Your Name
4. Click "Sign Up"
5. You'll be automatically logged in

### Adding Your First Cloud Account

1. After logging in, navigate to "Cloud Accounts"
2. Click "Add Cloud Account"
3. Select provider (AWS/Azure/GCP)
4. Enter credentials:
   - **AWS**: Access Key ID, Secret Access Key, Region
   - **Azure**: Subscription ID, Tenant ID, Client ID, Client Secret
   - **GCP**: Service Account JSON
5. Click "Save"

The application will now discover and monitor your cloud resources!

---

## Troubleshooting

### Issue: Containers Won't Start

**Check logs:**
```bash
docker-compose logs [service-name]
```

**Common causes:**
- Missing `.env` file or incorrect environment variables
- Port conflicts (check if ports 3001, 8002, 5433, 6380 are in use)
- Docker daemon not running

**Solution:**
```bash
# Check port usage
lsof -i :8002
lsof -i :3001
lsof -i :5433

# Change ports in .env if needed
```

### Issue: Backend Fails with "MASTER_ENCRYPTION_KEY not set"

**Solution:**
```bash
# Verify .env file exists and has the key
cat .env | grep MASTER_ENCRYPTION_KEY

# Restart backend
docker-compose restart backend
```

### Issue: Migration Errors

**Symptoms:**
```
sqlalchemy.exc.ProgrammingError: relation "users" does not exist
```

**Solution:**
```bash
# Reset database (WARNING: deletes all data)
docker-compose down -v
docker-compose up -d db
sleep 5
docker-compose up -d backend
```

### Issue: Frontend Shows "Failed to load" or CORS Errors

**Check:**
- Backend is running: `curl http://localhost:8002/`
- Frontend `.env` has correct `VITE_API_URL`
- No browser console errors

**Solution:**
```bash
# Restart both services
docker-compose restart backend frontend
```

### Issue: Celery Worker Exits

**Check logs:**
```bash
docker-compose logs celery
```

**Common causes:**
- Redis connection issues
- Missing environment variables

**Solution:**
```bash
# Verify Redis is healthy
docker-compose ps redis
docker-compose restart redis celery
```

### Issue: Can't Access GitHub Private Repos

**Symptoms:**
```
ERROR: failed to solve: process "/bin/sh -c git clone..." failed
```

**Solution:**
1. Verify GitHub PAT in `.env`
2. Check PAT has `repo` permissions
3. Rebuild images: `docker-compose build --no-cache`

### Database Connection Issues

**Check database:**
```bash
# Access database directly
docker-compose exec db psql -U postgres -d aeromonitor

# Check connection from backend
docker-compose exec backend python -c "from app.core.config import settings; print(settings.database_url)"
```

### Reset Everything

**Complete reset (deletes all data):**
```bash
# Stop all containers and remove volumes
docker-compose down -v

# Remove images (optional)
docker-compose down --rmi all

# Start fresh
docker-compose up -d
```

---

## Production Considerations

### Security

1. **Change Default Secrets:**
   - Generate strong `SECRET_KEY` (32+ characters)
   - Use secure `MASTER_ENCRYPTION_KEY`
   - Rotate GitHub PAT regularly

2. **Use Environment-Specific `.env`:**
   - Never commit `.env` files to Git
   - Use secrets management (AWS Secrets Manager, HashiCorp Vault, etc.)
   - Different keys for dev/staging/production

3. **Database Security:**
   - Use strong database passwords
   - Enable SSL/TLS for database connections
   - Restrict database access to application network

4. **Network Security:**
   - Use reverse proxy (nginx, Traefik) for HTTPS
   - Enable CORS restrictions in production
   - Use firewall rules to restrict access

### Performance

1. **Resource Limits:**
   ```yaml
   # Add to docker-compose.yml services
   deploy:
     resources:
       limits:
         cpus: '2'
         memory: 2G
   ```

2. **Database Optimization:**
   - Enable connection pooling
   - Regular database backups
   - Monitor query performance

3. **Caching:**
   - Redis is already configured
   - Consider CDN for frontend assets

### Monitoring

1. **Health Checks:**
   - Backend: `http://localhost:8002/health`
   - Use Docker health checks
   - Set up monitoring (Prometheus, Grafana)

2. **Logging:**
   - Centralize logs (ELK stack, CloudWatch)
   - Set up log rotation
   - Monitor error rates

### Backup

1. **Database Backups:**
   ```bash
   # Manual backup
   docker-compose exec db pg_dump -U postgres aeromonitor > backup.sql
   
   # Automated backups (cron job)
   0 2 * * * docker-compose exec -T db pg_dump -U postgres aeromonitor > /backups/aeromonitor-$(date +\%Y\%m\%d).sql
   ```

2. **Configuration Backups:**
   - Backup `.env` files securely
   - Version control configuration (without secrets)
   - Document environment setup

### Scaling

**Horizontal Scaling:**
- Run multiple backend instances behind load balancer
- Scale Celery workers based on task volume
- Use managed databases (RDS, Cloud SQL) for production

**Example:**
```bash
# Scale backend instances
docker-compose up -d --scale backend=3

# Scale Celery workers
docker-compose up -d --scale celery=5
```

---

## Quick Reference Commands

```bash
# Start everything
docker-compose up -d

# Stop everything
docker-compose down

# View logs
docker-compose logs -f [service]

# Restart a service
docker-compose restart [service]

# Access backend shell
docker-compose exec backend sh

# Access database
docker-compose exec db psql -U postgres -d aeromonitor

# View resource usage
docker stats

# Clean up unused resources
docker system prune -a
```

---

## Support & Resources

- **GitHub Repositories:**
  - Backend: https://github.com/crowntailtech/aero-backend-api
  - Frontend: https://github.com/crowntailtech/aero-frontend
  - Docker: https://github.com/crowntail-tech/crowntail

- **API Documentation:** http://localhost:8002/docs

- **Common Issues:** See [Troubleshooting](#troubleshooting) section above

---

## Next Steps

After successful deployment:

1. ✅ Create your user account
2. ✅ Add cloud accounts (AWS/Azure/GCP)
3. ✅ Discover services and instances
4. ✅ Set up monitoring for critical resources
5. ✅ Configure alerts and notifications (if implemented)
6. ✅ Review security settings
7. ✅ Set up automated backups

---

**Last Updated:** December 21, 2025  
**Version:** 1.0.0

