# Changelog - Appwrite for Coolify

## Version 3.0 - Critical Production Fixes (2026-07-19)

### 🔴 ALL CRITICAL ISSUES FIXED

#### 1. Gateway Network Added ✅
**Problem:** Services weren't accessible through Coolify proxy  
**Solution:** Added `gateway` network to all exposed services

```yaml
appwrite:
  networks:
    - gateway  # Coolify proxy access
    - appwrite # Internal communication

appwrite-console:
  networks:
    - gateway  # Coolify accessible
    - appwrite

appwrite-realtime:
  networks:
    - gateway  # WebSocket access
    - appwrite
```

#### 2. Healthchecks Implemented ✅
**Problem:** Services started before MariaDB/Redis were ready  
**Solution:** Added proper healthchecks with `service_healthy` conditions

**MariaDB Healthcheck:**
```yaml
healthcheck:
  test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
  interval: 10s
  timeout: 5s
  retries: 10
  start_period: 30s
```

**Redis Healthcheck:**
```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 5s
  timeout: 3s
  retries: 5
  start_period: 10s
```

**Service Dependencies:**
```yaml
depends_on:
  mariadb:
    condition: service_healthy  # ✅ Waits for healthy state
  redis:
    condition: service_healthy  # ✅ Waits for healthy state
```

#### 3. Realtime Environment Complete ✅
**Problem:** Realtime service missing critical environment variables  
**Solution:** Added all required variables via common-variables

```yaml
appwrite-realtime:
  environment:
    <<: *common-variables  # Includes all critical vars
    _APP_USAGE_STATS: ${_APP_USAGE_STATS:-enabled}
  networks:
    - gateway  # WebSocket access
    - appwrite
```

#### 4. Console Environment Added ✅
**Problem:** Console had no environment variables  
**Solution:** Added domain configuration

```yaml
appwrite-console:
  environment:
    _APP_DOMAIN: ${_APP_DOMAIN:?Domain is required}
    _APP_DOMAIN_TARGET: ${_APP_DOMAIN_TARGET:-}
  networks:
    - gateway
    - appwrite
```

#### 5. Service Dependencies Fixed ✅
**Problem:** Race conditions during startup  
**Solution:** Proper dependency chain with health checks

**Startup Order:**
1. MariaDB starts (30s initialization)
2. Redis starts (10s initialization)
3. Appwrite API (waits for healthy DB + Redis)
4. Console + Realtime (waits for healthy DB + Redis)
5. All Workers (wait for healthy DB + Redis)

#### 6. Common Variables Enhanced ✅
**Problem:** Critical variables not in common-variables  
**Solution:** Added all required variables

```yaml
x-common-variables: &common-variables
  _APP_ENV: ${_APP_ENV:-production}
  _APP_OPENSSL_KEY_V1: ${_APP_OPENSSL_KEY_V1:?Encryption key is required}
  _APP_DOMAIN: ${_APP_DOMAIN:?Domain is required}
  _APP_DOMAIN_TARGET: ${_APP_DOMAIN_TARGET:-}
  # ... all critical variables
```

### 📊 Impact

**Before v3.0:**
- ❌ Services not accessible through Coolify proxy
- ❌ Race conditions during startup
- ❌ Workers failing due to DB not ready
- ❌ Realtime missing environment
- ❌ Console not properly configured

**After v3.0:**
- ✅ All services properly exposed via gateway
- ✅ No race conditions - proper startup order
- ✅ Workers wait for healthy DB + Redis
- ✅ Realtime has complete environment
- ✅ Console properly configured
- ✅ WebSocket works correctly

### 🔧 Technical Changes

| Component | Change | Reason |
|-----------|--------|--------|
| Networks | Added `gateway` to exposed services | Coolify proxy access |
| MariaDB | Added healthcheck (30s start) | Ensure DB ready before connections |
| Redis | Added healthcheck (10s start) | Ensure cache ready before workers |
| Dependencies | `service_started` → `service_healthy` | Wait for ready state not just started |
| Realtime | Added complete environment | Missing critical variables |
| Console | Added environment variables | Missing domain config |
| Common vars | Added `_APP_DOMAIN`, `_APP_OPENSSL_KEY_V1` | Required by all services |

### ⚠️ Breaking Changes

1. **Gateway network required:** Must create before deployment
   ```bash
   docker network create gateway
   ```

2. **All required variables must be set:** Deployment fails if missing
   - `_APP_DOMAIN`
   - `_APP_OPENSSL_KEY_V1`
   - `_APP_EXECUTOR_SECRET`
   - `_APP_DB_ROOT_PASS`
   - `_APP_DB_SCHEMA`
   - `_APP_DB_USER`
   - `_APP_DB_PASS`

### 🧪 Validation

- ✅ `docker compose config` passes
- ✅ All healthchecks configured
- ✅ All networks properly assigned
- ✅ All environment variables validated
- ✅ Service startup order correct

### 📝 New Documentation

- `CRITICAL-FIXES-v3.md` - Detailed Bengali documentation of all fixes

---

## Version 2.0 - Environment Variables Fix (2026-07-19)

### 🔧 Critical Fixes

#### MariaDB Configuration Fixed
- **FIXED:** MariaDB failing to start due to missing environment variables
- Added explicit environment variable definitions with validation
- Used `:?` syntax to make critical variables REQUIRED
- Added both `MARIADB_*` and `MYSQL_*` variables for compatibility

```yaml
environment:
  MARIADB_ROOT_PASSWORD: ${_APP_DB_ROOT_PASS:?Database root password is required}
  MARIADB_DATABASE: ${_APP_DB_SCHEMA:?Database name is required}
  MARIADB_USER: ${_APP_DB_USER:?Database user is required}
  MARIADB_PASSWORD: ${_APP_DB_PASS:?Database password is required}
```

#### Environment Variable Architecture
- **CHANGED:** Removed dependency on `.env` file for all services
- **ADDED:** Explicit environment variables in docker-compose.yaml
- **ADDED:** Common variables anchor (x-common-variables) to reduce duplication
- **IMPROVED:** Coolify now auto-detects all required variables

#### Service Dependency Management
- **IMPROVED:** Added `depends_on` with `condition: service_started` for critical services
- **ENSURES:** MariaDB and Redis start before dependent services
- **VALIDATES:** Required environment variables before container creation

### 📋 New Files

1. **`.env.example`** - Complete environment variable documentation
   - All 80+ environment variables documented
   - Required vs optional variables clearly marked
   - Generation commands for secure keys
   - Coolify-specific deployment notes

### 🔄 Modified Services

All Appwrite services now explicitly declare required environment variables:

- ✅ `appwrite` - Main API (70+ variables)
- ✅ `appwrite-realtime` - WebSocket service
- ✅ `appwrite-console` - Web UI
- ✅ `appwrite-worker-*` - All 10 worker services
- ✅ `appwrite-task-*` - All 4 scheduler services
- ✅ `openruntimes-executor` - Function execution
- ✅ `openruntimes-proxy` - Function proxy
- ✅ `mariadb` - Database with validation
- ✅ `redis` - Cache/queue

### 🎯 Required Variables

Deployment will **FAIL** if these are not set (enforced by `:?` syntax):

```
_APP_OPENSSL_KEY_V1      - Encryption key (REQUIRED)
_APP_EXECUTOR_SECRET     - Executor secret (REQUIRED)
_APP_DOMAIN              - API domain (REQUIRED)
_APP_DB_ROOT_PASS        - Database root password (REQUIRED)
_APP_DB_SCHEMA           - Database name (REQUIRED)
_APP_DB_USER             - Database user (REQUIRED)
_APP_DB_PASS             - Database password (REQUIRED)
```

### 🔑 Key Improvements

1. **Coolify Compatibility**
   - Variables auto-detected from compose file
   - No manual .env file creation needed
   - Full validation before deployment

2. **Error Prevention**
   - Required variables validated at compose parse time
   - Clear error messages if variables missing
   - Prevents partial deployments

3. **Documentation**
   - Complete `.env.example` with all variables
   - Updated deployment guide
   - Troubleshooting section for MariaDB issues

4. **Maintainability**
   - Common variables defined once (x-common-variables)
   - Consistent environment structure across services
   - Easy to add new variables

### 🚀 Deployment Impact

**Before:**
- MariaDB would fail silently
- No validation of required variables
- Confusing "uninitialized database" errors
- Manual .env file creation required

**After:**
- Deployment fails early with clear error messages
- All required variables validated before container creation
- MariaDB starts reliably with proper credentials
- Coolify UI shows all configurable variables automatically

### 📦 Migration Guide

If upgrading from v1.0:

1. Pull the latest `docker-compose.yaml`
2. Remove any existing `.env` file (not needed)
3. Configure variables in Coolify's Environment Variables UI
4. Ensure all REQUIRED variables are set
5. Deploy normally

### ⚠️ Breaking Changes

1. **`.env` file no longer used** - Must configure through Coolify UI
2. **Required variables enforced** - Deployment will fail if not set
3. **Environment format changed** - From array format to object format

### 🧪 Testing

Validated with:
- ✅ Docker Compose config validation
- ✅ Variable substitution testing
- ✅ Required variable enforcement
- ✅ MariaDB initialization
- ✅ Service startup order

---

## Version 1.0 - Initial Coolify Release (2026-07-19)

### Initial Features

- Removed Traefik reverse proxy
- Removed all container names
- Removed all Traefik labels
- Updated networks for Coolify
- Basic environment variable support

### Known Issues (Fixed in v2.0)

- MariaDB could fail to start with environment variable errors
- No validation of required variables
- Relied on manual .env file creation

---

**Latest Version:** 3.0  
**Last Updated:** 2026-07-19  
**Compatibility:** Coolify v4+  
**Appwrite Version:** 1.9.5
