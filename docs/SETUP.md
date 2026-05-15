# Setup & Deployment Guide - Abbasi Hotel

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Local Development](#local-development)
3. [Docker Deployment](#docker-deployment)
4. [Configuration](#configuration)
5. [Troubleshooting](#troubleshooting)

## Prerequisites

### For Docker (Recommended)
- Docker 20.10+
- Docker Compose 2.0+
- 2GB RAM minimum
- 10GB disk space

### For Local Development
- Node.js 18.0+
- npm 9.0+
- MongoDB 5.0+
- 2GB RAM

## Local Development

### 1. Clone Repository
```bash
git clone https://github.com/masoudb1805-byte/Abbasi-Hotel.git
cd Abbasi-Hotel
```

### 2. Backend Setup
```bash
cd backend
cp .env.example .env

# Edit .env with your configuration
nano .env

# Install dependencies
npm install

# Start development server
npm run dev
```

Server runs on `http://localhost:5000`

### 3. Guest App Setup
```bash
cd ../frontend/guest-app
npm install
npm run dev
```

App runs on `http://localhost:5173`

### 4. Waiter App Setup
```bash
cd ../frontend/waiter-app
npm install
npm run dev
```

App runs on `http://localhost:5174`

### 5. Admin Panel Setup
```bash
cd ../frontend/admin-panel
npm install
npm run dev
```

App runs on `http://localhost:5175`

### 6. MongoDB Setup (if local)
```bash
# macOS (Homebrew)
brew services start mongodb-community

# Linux
sudo systemctl start mongod

# Windows
net start MongoDB
```

## Docker Deployment

### 1. Build & Start Services

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f backend
docker-compose logs -f mongo
```

### 2. Verify Services

```bash
# Check running containers
docker-compose ps

# Test backend health
curl http://localhost:5000/health

# Test API
curl -X POST http://localhost:5000/api/requests \
  -H "Content-Type: application/json" \
  -d '{"tableId":"T01","guestId":"G001","requestType":"water"}'
```

### 3. Stop Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v
```

### 4. Scale Services

```bash
# Scale waiter app to 3 instances
docker-compose up -d --scale waiter-app=3
```

## Configuration

### Environment Variables

Create `backend/.env`:

```env
# Server
PORT=5000
NODE_ENV=production
HOST=0.0.0.0

# Database
MONGODB_URI=mongodb://mongo:27017/abbasi-hotel
DB_NAME=abbasi-hotel

# JWT
JWT_SECRET=your-production-secret-key-change-this

# URLs
GUEST_APP_URL=http://yourdomain.com
WAITER_APP_URL=http://yourdomain.com/waiter
ADMIN_PANEL_URL=http://yourdomain.com/admin

# Waiter IPs (comma-separated)
WAITER_ALLOWED_IPS=192.168.1.0/24,10.0.0.0/8

# Features
ENABLE_NOTIFICATIONS=true
SOCKET_IO_CORS=http://yourdomain.com,http://yourdomain.com/waiter,http://yourdomain.com/admin
```

### Domain Configuration (Nginx)

Create `nginx/conf.d/default.conf`:

```nginx
upstream backend {
  server backend:5000;
}

upstream guest_app {
  server guest-app:3000;
}

upstream waiter_app {
  server waiter-app:3000;
}

upstream admin_panel {
  server admin-panel:3000;
}

server {
  listen 80;
  server_name yourdomain.com;

  location / {
    proxy_pass http://guest_app;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
  }

  location /waiter {
    proxy_pass http://waiter_app;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
  }

  location /admin {
    proxy_pass http://admin_panel;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
  }

  location /api {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Host $host;
  }
}
```

## Production Deployment

### Using Docker & Nginx

```bash
# Update docker-compose for production
COMPOSE_FILE=docker-compose.prod.yml docker-compose up -d

# Enable SSL with Let's Encrypt
docker run -it --rm --name certbot \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  certbot/certbot certonly --standalone \
  -d yourdomain.com
```

### Health Checks

```bash
# Backend health
curl http://localhost:5000/health

# MongoDB health
docker-compose exec mongo mongosh --eval "db.adminCommand('ping')"

# Application status
docker-compose ps
```

## Monitoring & Logs

```bash
# Real-time logs
docker-compose logs -f

# Specific service
docker-compose logs -f backend

# Last 100 lines
docker-compose logs --tail=100 backend

# Filter by timestamp
docker-compose logs --since 2026-05-15 backend
```

## Database Backup

```bash
# Backup MongoDB
docker-compose exec mongo mongodump --out /backup

# Restore MongoDB
docker-compose exec mongo mongorestore /backup
```

## Troubleshooting

### Services Won't Start

```bash
# Check service logs
docker-compose logs backend

# Verify ports are available
lsof -i :5000
lsof -i :27017

# Rebuild containers
docker-compose build --no-cache
docker-compose up -d
```

### Database Connection Issues

```bash
# Check MongoDB is running
docker-compose ps mongo

# Test connection
docker-compose exec backend curl mongodb://mongo:27017

# Check network
docker network ls
docker network inspect abbasi-hotel_abbasi-network
```

### WebSocket Connection Issues

```bash
# Check CORS configuration in .env
echo $SOCKET_IO_CORS

# Test WebSocket
curl -i -N -H "Connection: Upgrade" \
  -H "Upgrade: websocket" \
  http://localhost:5000
```

### High Memory Usage

```bash
# Check container memory
docker stats

# Limit memory (in docker-compose.yml)
services:
  backend:
    mem_limit: 512m
```

## Performance Optimization

### Database Indexing
```bash
# Create indexes for better query performance
docker-compose exec mongo mongosh abbasi-hotel
db.servicerequests.createIndex({ tableId: 1, status: 1 })
db.feedback.createIndex({ requestId: 1, createdAt: -1 })
db.waiters.createIndex({ deviceId: 1 })
```

### Caching Strategy
- Implement Redis for session caching
- Cache frequently accessed data
- Use CDN for static assets

### Load Balancing
- Scale waiter apps horizontally
- Use Nginx as load balancer
- Distribute requests evenly

## Security Checklist

- [ ] Change JWT_SECRET in production
- [ ] Enable HTTPS/SSL
- [ ] Restrict WAITER_ALLOWED_IPS
- [ ] Use strong MongoDB credentials
- [ ] Enable firewall rules
- [ ] Regular security updates
- [ ] Monitor access logs
- [ ] Implement DDoS protection

## Support & Issues

- Check logs: `docker-compose logs -f`
- Review API docs: `/docs/API.md`
- Open GitHub issue
- Contact: support@abbasi-hotel.com

---

**Last Updated:** 2026-05-15  
**Version:** 1.0.0
