# Aurora Hunter - Deployment Guide

## Prerequisites

Before deploying Aurora Hunter, ensure you have:

- **Server**: Linux server (Ubuntu 20.04+ recommended) with SSH access
- **PHP**: Version 8.0 or higher
- **MySQL**: Version 8.0 or higher  
- **Web Server**: Apache 2.4+ or Nginx
- **SSL Certificate**: Required for Stripe (Let's Encrypt recommended)
- **Composer**: For PHP dependencies
- **Stripe Account**: For payment processing
- **Domain**: Pointed to your server

## Step 1: Server Setup

### Install Required Software

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install PHP 8.0 and extensions
sudo apt install -y php8.0 php8.0-cli php8.0-fpm php8.0-mysql php8.0-mbstring \
    php8.0-xml php8.0-curl php8.0-zip php8.0-gd

# Install MySQL
sudo apt install -y mysql-server

# Install Apache (or Nginx)
sudo apt install -y apache2

# Enable required Apache modules
sudo a2enmod rewrite
sudo a2enmod ssl
sudo a2enmod headers

# Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Install Certbot for SSL
sudo apt install -y certbot python3-certbot-apache
```

## Step 2: Upload Files

```bash
# Create directory
sudo mkdir -p /var/www/aurora-hunter
cd /var/www/aurora-hunter

# Upload files (use SCP, SFTP, or Git)
# Option 1: SCP from local machine
scp -r aurora-hunter/* user@yourserver:/var/www/aurora-hunter/

# Option 2: Git
git clone your-repo-url .

# Set permissions
sudo chown -R www-data:www-data /var/www/aurora-hunter
sudo chmod -R 755 /var/www/aurora-hunter
```

## Step 3: Database Setup

```bash
# Secure MySQL installation
sudo mysql_secure_installation

# Create database and user
sudo mysql -u root -p

# In MySQL prompt:
CREATE DATABASE aurora_hunter CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'aurora_user'@'localhost' IDENTIFIED BY 'your_secure_password';
GRANT ALL PRIVILEGES ON aurora_hunter.* TO 'aurora_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;

# Import schema
cd /var/www/aurora-hunter
mysql -u aurora_user -p aurora_hunter < database/schema.sql
```

## Step 4: Backend Configuration

```bash
cd /var/www/aurora-hunter/backend

# Install PHP dependencies
composer install --no-dev --optimize-autoloader

# Create environment file
cp .env.example .env
nano .env
```

### Configure .env file:

```env
# Environment
ENVIRONMENT=production

# Database
DB_HOST=localhost
DB_PORT=3306
DB_NAME=aurora_hunter
DB_USER=aurora_user
DB_PASS=your_secure_password

# Stripe
STRIPE_SECRET_KEY=sk_live_your_key
STRIPE_PUBLISHABLE_KEY=pk_live_your_key
STRIPE_WEBHOOK_SECRET=whsec_your_secret
STRIPE_PRICE_MONTHLY=price_your_monthly_id
STRIPE_PRICE_LIFETIME=price_your_lifetime_id

# JWT
JWT_SECRET=generate_a_very_long_random_string_here
JWT_EXPIRY=3600

# API URLs
API_BASE_URL=https://yourdomain.com/api
FRONTEND_URL=https://yourdomain.com

# CORS (your domain only in production)
ALLOWED_ORIGINS=https://yourdomain.com

# Cache
CACHE_ENABLED=true
CACHE_TTL=3600

# Email (optional - for gift notifications)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_FROM=noreply@yourdomain.com
SMTP_FROM_NAME=Aurora Hunter
```

### Generate JWT Secret:

```bash
php -r "echo bin2hex(random_bytes(32)) . PHP_EOL;"
```

## Step 5: Apache Configuration

```bash
sudo nano /etc/apache2/sites-available/aurora-hunter.conf
```

```apache
<VirtualHost *:80>
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com
    
    DocumentRoot /var/www/aurora-hunter/backend/public
    
    <Directory /var/www/aurora-hunter/backend/public>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/aurora-hunter-error.log
    CustomLog ${APACHE_LOG_DIR}/aurora-hunter-access.log combined
</VirtualHost>
```

```bash
# Enable site
sudo a2ensite aurora-hunter.conf

# Disable default site
sudo a2dissite 000-default.conf

# Test configuration
sudo apache2ctl configtest

# Restart Apache
sudo systemctl restart apache2
```

## Step 6: SSL Certificate

```bash
# Get Let's Encrypt SSL certificate
sudo certbot --apache -d yourdomain.com -d www.yourdomain.com

# Certbot will:
# 1. Verify domain ownership
# 2. Issue certificate
# 3. Configure Apache for HTTPS
# 4. Set up auto-renewal

# Test auto-renewal
sudo certbot renew --dry-run
```

## Step 7: Stripe Setup

1. **Log into Stripe Dashboard** (https://dashboard.stripe.com)

2. **Create Products**:
   - Go to Products → Add Product
   - **Monthly Subscription**:
     - Name: "Aurora Hunter Premium"
     - Price: $4.99 CAD / month recurring
     - Copy the Price ID (starts with `price_...`)
   - **Lifetime Deal**:
     - Name: "Aurora Hunter Lifetime"
     - Price: $99.00 CAD / one-time
     - Copy the Price ID

3. **Update .env** with Price IDs

4. **Configure Webhook**:
   - Go to Developers → Webhooks
   - Add endpoint: `https://yourdomain.com/api/webhooks/stripe`
   - Select events:
     - `customer.subscription.created`
     - `customer.subscription.updated`
     - `customer.subscription.deleted`
     - `invoice.payment_succeeded`
     - `invoice.payment_failed`
   - Copy Signing Secret to `.env`

5. **Test Mode**: Use test keys first, then switch to live

## Step 8: Cron Jobs

```bash
# Edit crontab
crontab -e

# Add cleanup job (runs hourly)
0 * * * * php /var/www/aurora-hunter/backend/scripts/cleanup-cache.php >> /var/log/aurora-cache-cleanup.log 2>&1
```

## Step 9: Browser Extension Configuration

```bash
cd /var/www/aurora-hunter/extension

# Update config.js
nano config.js
```

Change:
```javascript
API_BASE: 'https://yourdomain.com/api',
STRIPE_PUBLISHABLE_KEY: 'pk_live_your_key',
```

### Package Extension:

```bash
# Create zip for submission
zip -r aurora-hunter-extension.zip * -x "*.git*"
```

## Step 10: Submit Extensions

### Firefox:
1. Go to https://addons.mozilla.org/developers/
2. Submit New Add-on
3. Upload `aurora-hunter-extension.zip`
4. Fill in listing details
5. Wait for review (typically 1-7 days)

### Chrome:
1. Go to https://chrome.google.com/webstore/devconsole
2. Pay $5 developer fee (one-time)
3. Upload `aurora-hunter-extension.zip`
4. Fill in listing details
5. Submit for review (typically 1-3 days)

## Step 11: Testing

### Test Backend API:

```bash
# Health check
curl https://yourdomain.com/api/health

# Test registration
curl -X POST https://yourdomain.com/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"testpass123"}'
```

### Test Extension:

1. Load extension in browser (developer mode)
2. Test registration/login
3. Test geolocation
4. Test forecast retrieval
5. Test upgrade flow with Stripe test card: `4242 4242 4242 4242`

### Test Webhooks:

```bash
# Install Stripe CLI
stripe login

# Forward webhooks to local
stripe listen --forward-to https://yourdomain.com/api/webhooks/stripe

# Trigger test event
stripe trigger payment_intent.succeeded
```

## Step 12: Monitoring

### Set up logging:

```bash
# Create log directory
sudo mkdir -p /var/log/aurora-hunter
sudo chown www-data:www-data /var/log/aurora-hunter

# Configure PHP logging
sudo nano /etc/php/8.0/apache2/php.ini
```

Find and update:
```ini
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
log_errors = On
error_log = /var/log/aurora-hunter/php-error.log
```

### Monitor logs:

```bash
# API errors
tail -f /var/log/aurora-hunter/php-error.log

# Apache access
tail -f /var/log/apache2/aurora-hunter-access.log

# Apache errors
tail -f /var/log/apache2/aurora-hunter-error.log

# Cache cleanup
tail -f /var/log/aurora-cache-cleanup.log
```

## Security Checklist

- [ ] SSL certificate installed and auto-renewing
- [ ] `.env` file not publicly accessible
- [ ] Database user has minimal required permissions
- [ ] Strong passwords for all accounts
- [ ] Stripe webhook signature verification enabled
- [ ] CORS restricted to your domain
- [ ] File permissions set correctly (755 for directories, 644 for files)
- [ ] Error messages don't expose sensitive info in production
- [ ] Regular backups configured

## Backup Strategy

```bash
# Database backup script
sudo nano /usr/local/bin/backup-aurora-db.sh
```

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
mysqldump -u aurora_user -p'your_password' aurora_hunter > /backups/aurora_hunter_$DATE.sql
find /backups -name "aurora_hunter_*.sql" -mtime +7 -delete
```

```bash
sudo chmod +x /usr/local/bin/backup-aurora-db.sh

# Add to crontab (daily at 2 AM)
0 2 * * * /usr/local/bin/backup-aurora-db.sh
```

## Troubleshooting

### "Unable to connect to database"
- Check MySQL is running: `sudo systemctl status mysql`
- Verify credentials in `.env`
- Test connection: `mysql -u aurora_user -p aurora_hunter`

### "Permission denied" errors
```bash
sudo chown -R www-data:www-data /var/www/aurora-hunter
sudo chmod -R 755 /var/www/aurora-hunter
```

### "500 Internal Server Error"
- Check PHP error log: `tail -f /var/log/aurora-hunter/php-error.log`
- Verify `.htaccess` is being read
- Check Apache error log

### Stripe webhook failures
- Verify webhook URL is accessible publicly
- Check webhook secret in `.env` matches Stripe dashboard
- Review Stripe dashboard webhook logs

### Extension can't connect to API
- Verify CORS settings in `public/api/index.php`
- Check SSL certificate is valid
- Verify API_BASE in extension `config.js`

## Support

For issues: https://github.com/yourusername/aurora-hunter/issues

## License

Copyright 2025 Web321 Marketing Ltd. All rights reserved.
