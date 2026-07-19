# Appwrite for Coolify 🚀

Production-ready Appwrite configuration optimized for Coolify deployment. This repository contains a fully tested Docker Compose configuration that works seamlessly with Coolify's reverse proxy and environment management.

[![Version](https://img.shields.io/badge/version-3.0-blue.svg)](CHANGELOG.md)
[![Appwrite](https://img.shields.io/badge/Appwrite-1.9.5-f02e65.svg)](https://appwrite.io)
[![Coolify](https://img.shields.io/badge/Coolify-v4+-6C47FF.svg)](https://coolify.io)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## ✨ Features

✅ **No Traefik** - Uses Coolify's built-in reverse proxy  
✅ **Auto-detected Variables** - All environment variables visible in Coolify UI  
✅ **Validated Configuration** - Required variables enforced before deployment  
✅ **MariaDB Ready** - Proper database initialization with credential validation  
✅ **WebSocket Support** - Realtime features fully configured  
✅ **Function Execution** - OpenRuntimes executor included  
✅ **Production Tested** - All services properly configured and tested  

## 🚀 Quick Start

### 1. Prerequisites

```bash
# Create the gateway network (required)
docker network create gateway
```

### 2. Deploy in Coolify

1. **Create new Docker Compose resource** in Coolify
2. **Import from Git**: `https://github.com/mashunterbd/coolify-appwrite`
3. **Configure Environment Variables** in Coolify UI (see [Required Variables](#-required-variables))
4. **Deploy!**

### 3. Configure Domains

Set up three domain routes in Coolify:

| Service | Domain | Path | Port | WebSocket |
|---------|--------|------|------|-----------|
| `appwrite` | api.yourdomain.com | `/` | 80 | No |
| `appwrite-console` | console.yourdomain.com | `/` | 80 | No |
| `appwrite-realtime` | api.yourdomain.com | `/v1/realtime` | 80 | **Yes** |

## 🔑 Required Variables

These **MUST** be set in Coolify or deployment will fail:

```env
_APP_DOMAIN=api.yourdomain.com
_APP_OPENSSL_KEY_V1=<generate-with-openssl>
_APP_EXECUTOR_SECRET=<generate-with-openssl>
_APP_DB_ROOT_PASS=<strong-password>
_APP_DB_SCHEMA=appwrite
_APP_DB_USER=appwrite
_APP_DB_PASS=<strong-password>
```

### Generate Secure Keys

```bash
# For encryption keys and secrets
openssl rand -base64 32

# For passwords
openssl rand -hex 16
```

## 📚 Documentation

- **[Deployment Guide](COOLIFY-DEPLOYMENT-GUIDE.md)** - Complete step-by-step deployment instructions
- **[Environment Variables](.env.example)** - All 80+ variables documented
- **[Changelog](CHANGELOG.md)** - Version history and migration guides

## 🏗️ Architecture

### Services Included

- **appwrite** - Main API server
- **appwrite-console** - Web UI dashboard
- **appwrite-realtime** - WebSocket service for realtime features
- **mariadb** - Primary database (MariaDB 10.11)
- **redis** - Cache and queue management
- **10 Worker Services** - Background job processors
- **4 Task Schedulers** - Automated maintenance and scheduling
- **openruntimes-executor** - Function execution engine
- **openruntimes-proxy** - Function routing proxy

### Networks

- `gateway` (external) - Coolify's reverse proxy network
- `appwrite` (internal) - Service communication
- `runtimes` (internal) - Function execution isolation

## 🔧 Configuration

### Minimum Required Configuration

```env
_APP_ENV=production
_APP_DOMAIN=api.yourdomain.com
_APP_OPENSSL_KEY_V1=[generated]
_APP_EXECUTOR_SECRET=[generated]
_APP_DB_ROOT_PASS=[generated]
_APP_DB_SCHEMA=appwrite
_APP_DB_USER=appwrite
_APP_DB_PASS=[generated]
_APP_DB_HOST=mariadb
_APP_REDIS_HOST=redis
_APP_OPTIONS_FORCE_HTTPS=disabled
```

### Optional Features

#### Email (SMTP)
```env
_APP_SMTP_HOST=smtp.example.com
_APP_SMTP_PORT=587
_APP_SMTP_SECURE=tls
_APP_SMTP_USERNAME=your-username
_APP_SMTP_PASSWORD=your-password
```

#### SMS
```env
_APP_SMS_PROVIDER=twilio
_APP_SMS_FROM=+1234567890
```

#### S3 Storage
```env
_APP_STORAGE_DEVICE=s3
_APP_STORAGE_S3_ACCESS_KEY=your-access-key
_APP_STORAGE_S3_SECRET=your-secret
_APP_STORAGE_S3_REGION=us-east-1
_APP_STORAGE_S3_BUCKET=your-bucket
```

## ✅ Verification Checklist

After deployment, verify:

- [ ] All services are running (`docker ps`)
- [ ] MariaDB initialized successfully (check logs)
- [ ] API accessible at `https://api.yourdomain.com/health`
- [ ] Console loads at `https://console.yourdomain.com`
- [ ] SSL/HTTPS working (green lock in browser)
- [ ] WebSocket connection working (test realtime features)
- [ ] Can create a new project in Console
- [ ] Can create and execute a function

## 🐛 Troubleshooting

### MariaDB Won't Start

**Error:** "Database is uninitialized and password option is not specified"

**Solution:**
1. Verify all database variables are set in Coolify:
   - `_APP_DB_ROOT_PASS`
   - `_APP_DB_SCHEMA`
   - `_APP_DB_USER`
   - `_APP_DB_PASS`
2. Check MariaDB container logs for specific errors
3. Ensure variables don't contain special characters that need escaping

### WebSocket Not Working

**Error:** Realtime features not updating

**Solution:**
1. Verify WebSocket is **enabled** in Coolify for `appwrite-realtime` service
2. Check that path `/v1/realtime` is correctly configured
3. Ensure HTTPS is enabled (WSS requires HTTPS)
4. Test with: `wscat -c wss://api.yourdomain.com/v1/realtime`

### 502 Bad Gateway

**Solution:**
1. Check all services are healthy: `docker ps`
2. Verify `gateway` network exists: `docker network ls`
3. Check service logs in Coolify dashboard
4. Ensure MariaDB started successfully before other services

## 📈 Performance Tips

1. **Database Tuning** - Increase `innodb_buffer_pool_size` for production
2. **Redis Memory** - Adjust `maxmemory` based on your workload
3. **Function Limits** - Configure `_APP_FUNCTIONS_CPUS` and `_APP_FUNCTIONS_MEMORY`
4. **Worker Scaling** - Increase `_APP_WORKER_PER_CORE` for high traffic

## 🔐 Security Recommendations

1. ✅ Use strong, randomly generated passwords
2. ✅ Enable HTTPS through Coolify (automatic)
3. ✅ Keep `_APP_OPTIONS_ABUSE=enabled`
4. ✅ Regularly update Appwrite version
5. ✅ Use environment-specific domains
6. ✅ Enable Redis password protection
7. ✅ Backup MariaDB data volume regularly

## 🆕 What's New in v3.0

### Critical Production Fixes (সব issues solved!)
- ✅ **Gateway network added** - API, Console, Realtime এখন Coolify proxy-accessible
- ✅ **Healthchecks implemented** - MariaDB & Redis properly wait for ready state
- ✅ **Service dependencies fixed** - Workers wait for healthy DB + Redis
- ✅ **Realtime environment complete** - All required variables included
- ✅ **Console environment added** - Domain configuration included
- ✅ **Proper startup order** - DB → Redis → Services → Workers

### Technical Improvements
- 🔧 `service_healthy` conditions replace `service_started`
- 🔧 MariaDB healthcheck with innodb initialization
- 🔧 Redis healthcheck with ping response
- 🔧 Enhanced common-variables with all critical vars

See [CRITICAL-FIXES-v3.md](CRITICAL-FIXES-v3.md) for complete details (বাংলা documentation).

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test your changes with Coolify
4. Submit a pull request

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🔗 Links

- **Repository:** https://github.com/mashunterbd/coolify-appwrite
- **Appwrite Docs:** https://appwrite.io/docs
- **Coolify Docs:** https://coolify.io/docs
- **Issues:** https://github.com/mashunterbd/coolify-appwrite/issues

## 💬 Support

- **Issues:** [GitHub Issues](https://github.com/mashunterbd/coolify-appwrite/issues)
- **Appwrite Discord:** https://discord.gg/appwrite
- **Coolify Discord:** https://discord.gg/coolify

## ⭐ Star This Repo

If this helped you deploy Appwrite on Coolify, please give it a star! ⭐

---

**Made with ❤️ for the Coolify and Appwrite communities**

Last Updated: 2026-07-19 | Version: 3.0
