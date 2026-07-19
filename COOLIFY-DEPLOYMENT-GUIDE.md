# Appwrite on Coolify - Deployment Guide

## ✅ Changes Applied

Your Docker Compose file has been successfully modified for Coolify compatibility. Here's what was changed:

### 1. ✅ Traefik Service Removed
The entire Traefik reverse proxy service has been **completely removed**. Coolify will handle all routing and SSL termination.

### 2. ✅ All Container Names Removed
Removed `container_name` from all services. Coolify automatically generates container names to prevent conflicts.

### 3. ✅ All Traefik Labels Removed
All Traefik-specific labels have been removed from:
- appwrite (main API)
- appwrite-console (web UI)
- appwrite-realtime (WebSocket service)

### 4. ✅ Networks Configuration Updated
```yaml
networks:
  gateway:
    external: true  # Uses Coolify's existing gateway network
  appwrite:         # Internal network for Appwrite services
  runtimes:         # Internal network for function execution
```

### 5. ✅ Internal Ports Preserved
All internal container ports remain unchanged (port 80 inside containers).

---

## 🚀 Coolify Deployment Steps

### Step 1: Create Gateway Network
Before deploying, ensure the `gateway` network exists:
```bash
docker network create gateway
```

### Step 2: Deploy in Coolify
1. In Coolify, create a new **Docker Compose** resource
2. Upload or paste your modified `docker-compose.yaml`
3. Upload your `.env` file with all required environment variables

### Step 3: Configure Domain Routing in Coolify

You need to configure **three separate domains/routes** in Coolify:

#### A) Main API Service (`appwrite`)
- **Service Name:** `appwrite`
- **Internal Port:** `80`
- **Domain:** `api.yourdomain.com` (or your preferred domain)
- **Path:** `/` (root path)
- **Enable HTTPS:** ✅ Yes

#### B) Console UI (`appwrite-console`)
- **Service Name:** `appwrite-console`
- **Internal Port:** `80`
- **Domain:** `console.yourdomain.com` (or subdomain)
- **Path:** `/console` (optional, can be `/` if on separate domain)
- **Enable HTTPS:** ✅ Yes

#### C) Realtime WebSocket (`appwrite-realtime`)
- **Service Name:** `appwrite-realtime`
- **Internal Port:** `80`
- **Domain:** `realtime.yourdomain.com` (or same as API)
- **Path:** `/v1/realtime`
- **Enable HTTPS:** ✅ Yes
- **WebSocket Support:** ✅ **MUST BE ENABLED**

---

## 🔧 Required Environment Variables

Ensure your `.env` file includes these critical variables:

```env
# API Endpoint (your Coolify domain)
_APP_ENDPOINT=https://api.yourdomain.com

# Console URL (your console domain)
_APP_CONSOLE_ENDPOINT=https://console.yourdomain.com

# Database Configuration
_APP_DB_HOST=mariadb
_APP_DB_PORT=3306
_APP_DB_SCHEMA=appwrite
_APP_DB_USER=appwrite
_APP_DB_PASS=your_secure_password
_APP_DB_ROOT_PASS=your_secure_root_password

# Redis Configuration
_APP_REDIS_HOST=redis
_APP_REDIS_PORT=6379

# Security
_APP_EXECUTOR_SECRET=your_secret_key
_APP_OPENSSL_KEY_V1=your_encryption_key

# Functions Configuration
_APP_COMPUTE_RUNTIMES_NETWORK=runtimes

# Environment
_APP_ENV=production
```

---

## ✅ Verification Checklist

After deployment, verify:

- [ ] **Main API is accessible** at `https://api.yourdomain.com`
- [ ] **Console UI loads** at `https://console.yourdomain.com`
- [ ] **HTTPS/SSL is working** (green lock icon in browser)
- [ ] **WebSocket connection works** for realtime features
- [ ] **Database connection is healthy** (check logs)
- [ ] **Redis connection is healthy** (check logs)
- [ ] **Functions can be created and executed**
- [ ] **No port conflicts** in Coolify or Docker
- [ ] **All worker services are running** (check Coolify dashboard)

---

## 🔍 Troubleshooting

### WebSocket Connection Issues
If realtime features don't work:
1. Verify WebSocket support is enabled in Coolify for the `appwrite-realtime` service
2. Check that the path `/v1/realtime` is correctly configured
3. Ensure HTTPS is enabled (WSS protocol requires HTTPS)

### 502 Bad Gateway
If you get 502 errors:
1. Check that all services are healthy: `docker ps`
2. Verify database and Redis are running
3. Check service logs in Coolify dashboard
4. Ensure the `gateway` network exists and services are connected

### Container Name Conflicts
If you see container name conflicts:
- Verify all `container_name` entries have been removed
- Coolify should auto-generate unique names

### Network Issues
If services can't communicate:
1. Verify the `gateway` network exists: `docker network ls`
2. Check that services are on the correct networks
3. Ensure `gateway` network is set to `external: true`

---

## 📝 Important Notes

1. **No Traefik**: Coolify's built-in reverse proxy replaces Traefik completely
2. **Gateway Network**: Must be created before deployment
3. **Domain Configuration**: All routing is configured in Coolify UI, not in docker-compose.yaml
4. **SSL Certificates**: Coolify handles SSL certificate generation and renewal automatically
5. **WebSocket**: Must be explicitly enabled in Coolify for the realtime service
6. **Internal Communication**: Services communicate via service names (e.g., `mariadb`, `redis`)

---

## 🎯 Expected Result

After successful deployment:

✅ No port conflicts  
✅ Works with Coolify Proxy  
✅ HTTPS enabled automatically  
✅ Realtime (WebSocket) working  
✅ Functions working  
✅ Clean Docker deployment without conflicts  

---

## 📞 Need Help?

If you encounter issues:
1. Check Coolify service logs
2. Check Docker container logs: `docker logs <container-name>`
3. Verify environment variables in Coolify
4. Ensure all domains are correctly configured in Coolify

---

**Generated for Coolify Deployment**  
Date: 2026-07-19
