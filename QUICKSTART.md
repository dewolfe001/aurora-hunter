# Aurora Hunter - Quick Start Guide

## What You Have

A complete, production-ready Aurora Hunter system including:

✅ **Backend API** (PHP 8.0+)
- User authentication with JWT
- Stripe payment integration (monthly + lifetime)
- Aurora, weather, and moon phase forecasting
- 3-day free / 7-day premium tiers
- Gift code system for lifetime deals

✅ **Browser Extension** (Firefox & Chrome compatible)
- User registration/login
- Geolocation integration
- Combined forecast display
- Upgrade flow
- Responsive UI

✅ **Database Schema** (MySQL 8.0+)
- Complete table structure
- Sample weather station data
- Optimized indexes

## 30-Minute Setup (Local Development)

### 1. Install Prerequisites (10 min)

```bash
# macOS
brew install php@8.0 mysql composer

# Ubuntu/Debian
sudo apt install php8.0 mysql-server composer

# Windows
# Download XAMPP from apachefriends.org
```

### 2. Setup Database (5 min)

```bash
mysql -u root -p

# In MySQL:
CREATE DATABASE aurora_hunter;
USE aurora_hunter;
SOURCE /path/to/aurora-hunter/database/schema.sql;
EXIT;
```

### 3. Configure Backend (5 min)

```bash
cd aurora-hunter/backend
composer install
cp .env.example .env

# Edit .env with your database credentials
# For testing, use Stripe test keys
```

### 4. Start Local Server (2 min)

```bash
cd aurora-hunter/backend/public
php -S localhost:8000
```

Visit: http://localhost:8000/api/health
Should return: `{"status":"healthy","timestamp":...}`

### 5. Load Extension (5 min)

**Firefox**:
1. Go to `about:debugging#/runtime/this-firefox`
2. Click "Load Temporary Add-on"
3. Select `aurora-hunter/extension/manifest.json`

**Chrome**:
1. Go to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select `aurora-hunter/extension` folder

### 6. Test It! (3 min)

1. Click extension icon
2. Register new account
3. Allow location access
4. View your 3-day aurora forecast!

## Production Deployment

See [DEPLOYMENT.md](DEPLOYMENT.md) for complete production deployment instructions.

## Key Configuration Files

- `backend/.env` - All secrets and config
- `extension/config.js` - API URL and Stripe public key
- `backend/public/api/index.php` - Main API router

## Stripe Test Cards

- Success: `4242 4242 4242 4242`
- Decline: `4000 0000 0000 0002`
- Requires 3D Secure: `4000 0027 6000 3184`

## Common Tasks

### Add a Weather Station:

```sql
INSERT INTO weather_stations (station_code, name, province_code, latitude, longitude) 
VALUES ('NEW', 'New Station', 'BC', 50.0, -120.0);
```

### Test API Endpoints:

```bash
# Register
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"test1234"}'

# Login  
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"test1234"}'
```

### Clear Cache:

```bash
php backend/scripts/cleanup-cache.php
```

## File Structure

```
aurora-hunter/
├── backend/              # PHP API
│   ├── src/             # Application code
│   ├── public/api/      # Entry point
│   ├── scripts/         # Utility scripts
│   └── .env.example     # Config template
├── extension/           # Browser extension
│   ├── manifest.json    # Extension manifest
│   ├── popup.html/js/css
│   ├── background.js    # Service worker
│   └── config.js        # Configuration
├── database/            # SQL schema
└── README.md           # This file
```

## Support & Resources

- **Documentation**: See [DEPLOYMENT.md](DEPLOYMENT.md)
- **Environment Canada API**: https://api.weather.gc.ca/
- **NOAA Space Weather**: https://www.swpc.noaa.gov/
- **Stripe Docs**: https://stripe.com/docs
- **Browser Extension Docs**: 
  - Firefox: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions
  - Chrome: https://developer.chrome.com/docs/extensions/

## Troubleshooting

**"Composer not found"**
```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

**"Class not found" errors**
```bash
cd backend
composer dump-autoload
```

**Extension won't load**
- Check manifest.json syntax
- Ensure all referenced files exist
- Check browser console for errors

**API returns 500 error**
- Check PHP error log
- Verify database connection
- Ensure .env file exists and is configured

## Next Steps

1. ✅ Get it running locally (30 min)
2. 📝 Customize branding and copy
3. 🎨 Create extension icons
4. 🔐 Set up Stripe account and products
5. 🚀 Deploy to production server
6. 📱 Submit extensions to stores
7. 🎉 Launch!

## License

Copyright 2025 Web321 Marketing Ltd. All rights reserved.

---

**Made with ❤️ in British Columbia, Canada** 🇨🇦
