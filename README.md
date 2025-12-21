# AeroMonitor Docker Setup (Split Repositories)

This directory contains the Docker Compose configuration for running AeroMonitor with the split repository architecture.

## 🚀 Quick Start

```bash
cd /Users/nitusinha/Documents/aero-docker

# Set your encryption key (required)
export MASTER_ENCRYPTION_KEY="your-encryption-key-here"

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

## 📦 Services

- **backend**: FastAPI backend (port 8000)
  - Uses packages from `aero-cloud-sdk` and `aero-security` (installed from GitHub)
  - Location: `../aero-backend-api-split`
  
- **frontend**: React frontend (port 3000)
  - Connects to backend at `http://localhost:8000`
  - Location: `../aero-frontend-split`
  
- **db**: PostgreSQL database (port 5432)
  
- **redis**: Redis cache (port 6379)
  
- **celery**: Celery worker for background tasks

## 🔧 Configuration

The backend automatically installs packages from GitHub:
- `aero-cloud-sdk` - Cloud provider integrations
- `aero-security` - Encryption and secrets management

If submodules are available locally, they'll be used instead.

## 🌐 Access

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs

## 📝 Environment Variables

Create a `.env` file or export:
- `MASTER_ENCRYPTION_KEY` - Required for encrypting cloud credentials
- `SECRET_KEY` - JWT secret key
- `OPENAI_API_KEY` - Optional, for AI features

## 🔄 Development

The services are configured with volume mounts for live reload:
- Backend: Changes to code will reload automatically
- Frontend: Changes will hot-reload in the browser

## 🐛 Troubleshooting

1. **Backend won't start**: Check `MASTER_ENCRYPTION_KEY` is set
2. **Frontend can't connect**: Verify backend is running on port 8000
3. **Database errors**: Ensure PostgreSQL container is healthy

Check logs:
```bash
docker-compose logs backend
docker-compose logs frontend
docker-compose logs db
```

