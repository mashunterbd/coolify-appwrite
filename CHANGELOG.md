# Changelog - Appwrite for Coolify

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

**Latest Version:** 2.0  
**Last Updated:** 2026-07-19  
**Compatibility:** Coolify v4+  
**Appwrite Version:** 1.9.5
