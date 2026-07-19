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
3. **Configure Environment Variables in Coolify UI:**
   - Coolify will automatically detect all required variables
   - Fill in the REQUIRED variables (see section above)
   - You can reference `.env.example` for all available options
   - **DO NOT upload a .env file** - use Coolify's Environment Variables UI instead

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

**CRITICAL:** All environment variables are now explicitly defined in the `docker-compose.yaml` file and will be automatically detected by Coolify.

### ⚠️ REQUIRED Variables (Deployment will FAIL without these)

These variables use the `:?` syntax to ensure they are provided:

```env
# Security (REQUIRED)
_APP_OPENSSL_KEY_V1=your_encryption_key_here
_APP_EXECUTOR_SECRET=your_executor_secret_here
_APP_DOMAIN=api.yourdomain.com

# Database (REQUIRED)
_APP_DB_ROOT_PASS=your_secure_root_password
_APP_DB_SCHEMA=appwrite
_APP_DB_USER=appwrite
_APP_DB_PASS=your_secure_password
```

### 🔑 How to Generate Secure Keys

Run these commands to generate secure random keys:

```bash
# For encryption key and executor secret
openssl rand -base64 32

# For passwords
openssl rand -hex 16
```

### 📋 All Environment Variables

Coolify will automatically show ALL environment variables from the compose file in its UI. You can see the complete list in the `.env.example` file included in this repository.

**Important Configuration Notes:**

1. **No .env file needed:** Configure everything through Coolify's Environment Variables UI
2. **Auto-detection:** Coolify reads variables directly from `docker-compose.yaml`
3. **Validation:** Required variables (marked with `:?`) will prevent deployment if missing
4. **Defaults:** Many variables have sensible defaults and are optional

### Minimum Configuration for Quick Start

```env
_APP_ENV=production
_APP_DOMAIN=api.yourdomain.com
_APP_OPENSSL_KEY_V1=[generate with: openssl rand -base64 32]
_APP_EXECUTOR_SECRET=[generate with: openssl rand -base64 32]
_APP_DB_ROOT_PASS=[generate with: openssl rand -hex 16]
_APP_DB_SCHEMA=appwrite
_APP_DB_USER=appwrite
_APP_DB_PASS=[generate with: openssl rand -hex 16]
_APP_DB_HOST=mariadb
_APP_REDIS_HOST=redis
_APP_OPTIONS_FORCE_HTTPS=disabled
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
1. **Check MariaDB first** - it must start successfully
2. Verify all REQUIRED environment variables are set in Coolify
3. Check that all services are healthy: `docker ps`
4. Verify database and Redis are running
5. Check service logs in Coolify dashboard
6. Ensure the `gateway` network exists and services are connected

### MariaDB Initialization Errors
If MariaDB fails with "Database is uninitialized and password option is not specified":
1. Verify these REQUIRED variables are set in Coolify:
   - `_APP_DB_ROOT_PASS`
   - `_APP_DB_SCHEMA`
   - `_APP_DB_USER`
   - `_APP_DB_PASS`
2. These variables use `:?` syntax and will prevent deployment if missing
3. Check Coolify's Environment Variables page to ensure they're configured
4. MariaDB container logs will show specific missing variables

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
7. **Environment Variables**: All variables are explicitly defined in docker-compose.yaml and auto-detected by Coolify
8. **Required Variables**: Variables marked with `:?` will prevent deployment if not set
9. **No .env File**: Configure all variables through Coolify's Environment Variables UI
10. **MariaDB Dependencies**: All Appwrite services wait for MariaDB to start successfully

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
