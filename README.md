# Aurora Hunter 🌌

A browser extension that predicts aurora viewing conditions by combining aurora forecasts, lunar cycles, and weather data.

## Features

- **Free Tier**: 3-day aurora predictions based on current location
- **Premium Tier ($4.99/month)**: 7-day predictions + search any Canadian location
- **Lifetime Deal ($99 one-time)**: All premium features forever + ability to gift to others

## Tech Stack

- **Backend**: PHP 8.0+, MySQL 8.0+
- **Payment**: Stripe
- **Browser Extension**: Vanilla JS (Firefox & Chrome compatible)
- **APIs**: Environment Canada, NOAA Aurora, US Naval Observatory (Moon phases)

## Quick Start Deployment

### Prerequisites

- PHP 8.0 or higher
- MySQL 8.0 or higher
- Composer
- SSL certificate (required for Stripe)
- Stripe account

### 1. Backend Setup

```bash
cd backend
composer install
cp .env.example .env
```

Edit `.env` with your credentials

### 2. Database Setup

```bash
mysql -u root -p < database/schema.sql
```

### 3. Configure & Deploy

See full deployment guide in [DEPLOYMENT.md](DEPLOYMENT.md)

### 4. Extension Setup

Update `extension/config.js` with your API URL, then load in browser.

## Project Structure

```
aurora-hunter/
├── backend/           # PHP API backend
├── extension/         # Browser extension
├── database/          # SQL schema
└── docs/             # Documentation
```

## License

Copyright 2025 Web321 Marketing Ltd. All rights reserved.
