# AeroMonitor Split Repository Docker Setup - Status

## ✅ Services Running

### Split Repository Setup (New)
- **Frontend**: ✅ Running on http://localhost:3001
- **Backend**: ⚠️ Starting (check logs if not responding)
- **Database**: ✅ Running on port 5433
- **Redis**: ✅ Running on port 6380

### Original Setup (Still Active)
- **Frontend**: ✅ Running on http://localhost:3000  
- **Backend**: ✅ Running on http://localhost:8000
- **Database**: ✅ Running on port 5432
- **Redis**: ✅ Running on port 6379

## 🔧 Troubleshooting

If backend (8002) is not responding:

1. **Check logs**:
   ```bash
   cd /Users/nitusinha/Documents/aero-docker
   docker-compose logs backend
   ```

2. **Check container status**:
   ```bash
   docker-compose ps
   ```

3. **Restart backend**:
   ```bash
   docker-compose restart backend
   ```

4. **Common issues**:
   - Migration errors: Non-fatal, backend should still start
   - Port conflicts: Services use ports 8002, 3001, 5433, 6380
   - Database connection: Ensure DB container is healthy

## 📝 Notes

- Both setups can run simultaneously
- They use separate databases (different ports)
- Frontend on 3001 connects to backend on 8002
- Original setup unchanged on ports 8000/3000

