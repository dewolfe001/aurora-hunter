# Aurora Hunter - Product Requirements Document (PRD)

**Version:** 2.0  
**Date:** November 2025  
**Author:** Web321 Marketing Ltd.  
**Status:** Active Development

---

## Executive Summary

Aurora Hunter is a SaaS browser extension that predicts optimal aurora viewing conditions for Canadians by combining real-time aurora forecasts, weather data, and lunar cycle information into a single, actionable viewing score. The service operates on a freemium model with automatic alerts for high-probability aurora events.

### Value Proposition
"Never miss a spectacular aurora display. Get intelligent predictions and real-time alerts for the best aurora viewing nights in your area."

---

## 1. Problem Statement

### Current Pain Points
1. **Fragmented Information**: Aurora watchers must check multiple sources (space weather, local weather, moon phase) separately
2. **Missed Opportunities**: No proactive alerts when viewing conditions become ideal
3. **Location Limitations**: Most aurora apps focus on Arctic regions, not Canadian mid-latitudes (45-60°N)
4. **Poor Timing**: Users don't know which upcoming nights will have the best combined conditions

### Our Solution
A unified platform that:
- Aggregates aurora, weather, and lunar data into a single viewing score (0-20)
- Automatically monitors conditions every 12 hours
- Sends push notifications when high-probability events are detected
- Provides 3-7 day forecasts tailored to user's location
- Works seamlessly across Canada's major population centers

---

## 2. Goals & Objectives

### Business Goals
- **Revenue**: Generate $25,000+ ARR within 12 months
- **User Acquisition**: 5,000+ registered users in Year 1
- **Conversion**: 10% free-to-paid conversion rate
- **Retention**: 80% monthly retention for paid subscribers
- **Market Position**: Become the #1 aurora prediction tool for Canadians

### Product Goals
- **Accuracy**: Achieve 75%+ accuracy in predicting visible aurora events
- **Engagement**: 50%+ of users enable notifications
- **Performance**: API response time <500ms (95th percentile)
- **Reliability**: 99.5% uptime
- **User Satisfaction**: 4.5+ star rating in browser extension stores

---

## 3. Target Users

### Primary Persona: "Aurora Enthusiast Emma"
- **Demographics**: 35-55 years old, lives in Victoria/Calgary/Winnipeg
- **Behavior**: Checks aurora forecasts 2-3x per week, owns DSLR camera
- **Pain Points**: Wastes time checking multiple websites, misses sudden aurora events
- **Needs**: Consolidated data, reliable alerts, multi-day planning
- **Tech Comfort**: High - comfortable installing browser extensions

### Secondary Persona: "Casual Viewer Chris"
- **Demographics**: 25-45 years old, urban Canadian
- **Behavior**: Curious about auroras but not actively seeking them
- **Pain Points**: Doesn't know when/where to look, intimidated by technical data
- **Needs**: Simple yes/no guidance, smartphone notifications
- **Tech Comfort**: Medium - uses popular apps/extensions

### Tertiary Persona: "Professional Photographer Pat"
- **Demographics**: 30-60 years old, photography business owner
- **Behavior**: Plans multi-day trips around aurora forecasts
- **Pain Points**: Needs advance notice for travel planning, wants detailed metrics
- **Needs**: 7+ day forecasts, multiple location monitoring, historical accuracy data
- **Tech Comfort**: Very high - uses multiple tools and APIs

---

## 4. Feature Requirements

### 4.1 Core Features (MVP)

#### F1: User Authentication
**Priority**: P0 (Critical)

**Description**: Secure user registration and login system with JWT-based authentication.

**Requirements**:
- Email + password registration
- Secure password requirements (min 8 characters)
- JWT token authentication with 1-hour expiry
- Token refresh mechanism
- Password reset capability (future)

**Acceptance Criteria**:
- User can register with valid email
- User can login with correct credentials
- Invalid credentials are rejected
- JWT token expires after 1 hour
- Refresh token extends session

---

#### F2: Geolocation Integration
**Priority**: P0 (Critical)

**Description**: Browser-based geolocation to determine user's viewing location.

**Requirements**:
- Request browser geolocation permission
- Cache last known location
- Manual location entry fallback
- Coordinate validation (Canada bounds: 42°N-83°N, -141°W to -52°W)

**Acceptance Criteria**:
- User grants location permission once
- Location cached for future requests
- Invalid coordinates rejected
- Manual override available

---

#### F3: Tiered Forecast System
**Priority**: P0 (Critical)

**Description**: Server-side tier validation and data filtering based on membership level.

**API Request Format**:
```json
{
  "latitude": 48.4284,
  "longitude": -123.3656,
  "membership_id": "usr_abc123" | null
}
```

**API Response Format**:
```json
{
  "success": true,
  "user_tier": "free" | "premium" | "lifetime",
  "days_included": 3 | 7,
  "location": {
    "latitude": 48.4284,
    "longitude": -123.3656,
    "nearest_station": "Victoria, BC"
  },
  "forecast": [...],
  "best_nights": [...],
  "next_update_at": "2025-11-17T12:00:00Z",
  "generated_at": "2025-11-16T23:15:00Z"
}
```

**Tier Logic**:
- **membership_id = null**: Return 3-day forecast, current location only
- **membership_id = valid**: Validate against database, return 7-day forecast

**Requirements**:
- Server validates membership_id against user database
- Invalid membership_id treated as free tier
- Expired subscriptions automatically downgraded to free
- Response clearly indicates user's tier
- Forecast days limited by tier (3 vs 7)

**Acceptance Criteria**:
- Free users (null membership_id) receive 3-day forecast
- Premium users receive 7-day forecast
- Invalid membership_id returns 3-day forecast + error message
- Response time <500ms for cached data
- Response time <2s for fresh API calls

---

#### F4: Combined Forecast Algorithm
**Priority**: P0 (Critical)

**Description**: Multi-factor scoring algorithm that evaluates aurora viewing conditions.

**Data Sources**:
1. **Aurora Activity** (NOAA/Auroras.live)
   - Kp index (0-9 scale)
   - Geomagnetic storm level
   - 30-minute nowcast
   - 3-day forecast

2. **Weather Conditions** (Environment Canada / Open-Meteo)
   - Cloud cover percentage
   - Precipitation forecast
   - Visibility
   - Sky condition

3. **Lunar Phase** (US Naval Observatory)
   - Moon illumination percentage
   - Phase name (New, Full, etc.)
   - Moonrise/moonset times
   - Darkness score

**Scoring Algorithm**:
```
Total Score (0-20) = Aurora Score + Clarity Score + Darkness Score

Aurora Score (0-10):
  - Kp 0-2: 0-3 points (Poor)
  - Kp 3-4: 4-6 points (Fair)
  - Kp 5-6: 7-8 points (Good)
  - Kp 7-9: 9-10 points (Excellent)

Clarity Score (0-5):
  - 0-20% cloud cover: 5 points
  - 21-40% cloud: 4 points
  - 41-60% cloud: 3 points
  - 61-80% cloud: 2 points
  - 81-100% cloud: 0-1 points
  - Precipitation reduces by 1-2 points

Darkness Score (0-5):
  - 0-20% moon: 5 points
  - 21-40% moon: 4 points
  - 41-60% moon: 3 points
  - 61-80% moon: 2 points
  - 81-100% moon: 1 point
```

**Viewing Recommendations**:
- **16-20**: "Exceptional! Drop everything and go outside!"
- **12-15**: "Excellent conditions - highly recommended"
- **8-11**: "Good chance of viewing - worth checking"
- **5-7**: "Fair conditions - may see faint aurora"
- **0-4**: "Poor conditions - not recommended"

**Requirements**:
- Calculate score for each day in forecast period
- Identify "best nights" (top 3 scores)
- Include human-readable recommendations
- Cache calculations for 1 hour
- Update when new data available

**Acceptance Criteria**:
- Score accurately reflects combined conditions
- Best nights clearly highlighted
- Recommendations match score ranges
- Algorithm processes in <100ms

---

#### F5: Automated Polling System
**Priority**: P0 (Critical)

**Description**: Browser extension automatically checks for forecast updates every 12 hours.

**Polling Mechanism**:
```javascript
// Polling Interval: 12 hours (43,200,000 ms)
const POLLING_INTERVAL = 12 * 60 * 60 * 1000;

// On extension install/startup
chrome.alarms.create('forecastUpdate', {
  periodInMinutes: 720  // 12 hours
});

// Alarm handler
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'forecastUpdate') {
    updateForecast();
  }
});
```

**Update Process**:
1. Extension wakes up every 12 hours
2. Checks if user is logged in
3. Retrieves cached location
4. Sends API request with membership_id
5. Compares new forecast to previous
6. Triggers alert if high-probability event detected
7. Updates badge icon with score
8. Stores forecast in local cache

**Requirements**:
- Polling continues even when browser closed (service worker)
- Respects user's login state
- Handles network failures gracefully
- Implements exponential backoff on errors
- Logs polling activity for debugging
- User can manually trigger refresh
- Rate limited to 1 request per 10 minutes (manual refreshes)

**Acceptance Criteria**:
- Forecast updates every 12 hours ±5 minutes
- Works with browser closed
- Recovers from network errors
- Manual refresh limited to once per 10 minutes
- No unnecessary API calls

---

#### F6: High-Probability Alert System
**Priority**: P0 (Critical)

**Description**: Push notifications when viewing conditions exceed configurable threshold.

**Alert Triggers**:

**Immediate Alert** (within 24 hours):
- Viewing score ≥ 14 AND
- Kp index ≥ 5 AND
- Cloud cover < 50% AND
- No active alerts for same event

**Advance Warning** (24-72 hours):
- Viewing score ≥ 16 AND
- Kp index ≥ 6 AND
- Cloud cover < 30%

**Alert Format**:
```javascript
{
  type: "immediate" | "advance",
  title: "Aurora Alert: Excellent Viewing Tonight!",
  message: "Kp 6.5 expected with clear skies. Score: 17/20",
  icon: "aurora-icon.png",
  date: "2025-11-16",
  score: 17,
  kp_index: 6.5,
  conditions: {
    cloud_cover: 15,
    moon_illumination: 10
  },
  action_url: "popup.html#forecast"
}
```

**Notification Channels**:
1. **Browser Notification** (primary)
   - Native OS notification
   - Clickable to open extension
   - Includes score and Kp index
   - Shows tonight vs tomorrow

2. **Badge Update** (always)
   - Extension icon shows highest score
   - Color coded:
     - Red (16-20): Excellent
     - Orange (12-15): Very Good
     - Yellow (8-11): Good
     - Gray (<8): Poor

3. **Email Alert** (optional, future)
   - For premium users
   - 24-hour advance warnings
   - Includes detailed conditions

**User Controls**:
- Enable/disable notifications (default: ON)
- Minimum alert threshold (default: 14)
- Quiet hours (default: 10 PM - 7 AM)
- Alert frequency cap (max 1 per 6 hours per location)

**Requirements**:
- Request notification permission on first use
- Check permission before sending
- Deduplicate alerts (same event)
- Respect quiet hours
- Include clear call-to-action
- Log all alerts sent
- Track alert effectiveness (open rate)

**Acceptance Criteria**:
- Notification appears within 5 minutes of detection
- User can dismiss notification
- Clicking notification opens extension
- No duplicate alerts for same event
- Respects quiet hours setting
- Works when browser is closed (via service worker)

---

#### F7: Subscription Management
**Priority**: P0 (Critical)

**Description**: Stripe-based payment processing with two tiers.

**Subscription Tiers**:

**Free Tier** (membership_id = null):
- 3-day forecast
- Current location only
- 12-hour update cycle
- Basic alerts (score ≥ 16 only)
- No location search

**Premium Tier** ($4.99 CAD/month):
- 7-day forecast
- Multiple saved locations (up to 10)
- 12-hour update cycle
- Enhanced alerts (score ≥ 14)
- Location search across Canada
- Alert customization (threshold, quiet hours)

**Lifetime Tier** ($99 CAD one-time):
- All Premium features
- Lifetime access
- Giftable to others via email
- Exclusive "Lifetime" badge
- Early access to new features

**Stripe Integration**:
- Payment method: Credit card via Stripe Elements
- Subscription management via Stripe Customer Portal
- Webhook events for subscription lifecycle
- Automatic downgrade on payment failure (grace period: 7 days)
- Prorated refunds for cancellations (within 30 days)

**Gift System**:
```
Purchase Flow:
1. User purchases Lifetime tier ($99)
2. System generates unique gift code: AH-XXXXXX
3. Purchaser receives code + gift form
4. Purchaser enters recipient email
5. Recipient receives email with redemption link
6. Recipient creates account (or logs in)
7. Code applied, lifetime access granted

Code Format: AH-[12 alphanumeric characters]
Example: AH-8F2K9P4M7Q3N
```

**Requirements**:
- Secure Stripe integration (PCI compliant)
- Subscription status checks on every API request
- Automatic tier downgrade on cancellation
- Gift codes are single-use
- Gift codes expire after 1 year
- Email validation before gifting
- Prevent self-gifting

**Acceptance Criteria**:
- Payment succeeds with valid card
- Subscription activates immediately
- Webhook updates subscription status
- Failed payments retry 3 times over 7 days
- Gift codes work for new and existing users
- Downgrade happens automatically

---

### 4.2 Premium Features

#### F8: Location Search & Management
**Priority**: P1 (High)

**Description**: Search and save multiple Canadian locations.

**Requirements**:
- Text search for cities/towns
- Autocomplete with Canadian locations
- Save up to 10 locations (premium)
- Set default location
- Get forecasts for any saved location
- Quick location switcher in extension

**Acceptance Criteria**:
- Search returns relevant results
- Saved locations persist across sessions
- Can get forecast for any saved location
- Free users see upgrade prompt

---

#### F9: Historical Accuracy Tracking
**Priority**: P2 (Medium)

**Description**: Track forecast accuracy and show confidence metrics.

**Requirements**:
- Store actual viewing reports (user submitted)
- Compare predicted score to actual visibility
- Calculate accuracy percentage
- Display confidence intervals
- Show performance over time

**Acceptance Criteria**:
- Accuracy data available for premium users
- Confidence shown with each forecast
- Historical trends visualized

---

### 4.3 Future Enhancements

#### F10: Mobile App
**Priority**: P3 (Low)

**Description**: Native iOS/Android apps with same features.

---

#### F11: Social Features
**Priority**: P3 (Low)

**Description**: Share aurora sightings, photos, and forecasts.

---

#### F12: Aurora Photography Tips
**Priority**: P3 (Low)

**Description**: Camera settings recommendations based on conditions.

---

## 5. Technical Architecture

### 5.1 System Components

```
┌─────────────────┐
│  Browser Ext    │
│  (Frontend)     │
│                 │
│  - Manifest V3  │
│  - Service      │
│    Worker       │
│  - Geolocation  │
│  - Notifications│
└────────┬────────┘
         │
         │ HTTPS/JSON
         │ Every 12h
         │
         ▼
┌─────────────────┐
│   API Server    │
│   (Backend)     │
│                 │
│  - PHP 8.0+     │
│  - REST API     │
│  - JWT Auth     │
│  - Stripe SDK   │
└────────┬────────┘
         │
         │
    ┌────┴─────┐
    │          │
    ▼          ▼
┌────────┐  ┌──────────┐
│ MySQL  │  │  Stripe  │
│   DB   │  │   API    │
└────────┘  └──────────┘
    │
    │ Cache
    │ Queries
    ▼
┌────────────────────┐
│  External APIs     │
│                    │
│  - NOAA Aurora     │
│  - Env. Canada     │
│  - USNO Moon       │
│  - Open-Meteo      │
└────────────────────┘
```

### 5.2 API Endpoint Specification

#### GET /api/forecast

**Request Headers**:
```
Authorization: Bearer {jwt_token}
Content-Type: application/json
```

**Request Body**:
```json
{
  "latitude": 48.4284,
  "longitude": -123.3656,
  "membership_id": "usr_abc123" | null
}
```

**Response (Success - Free Tier)**:
```json
{
  "success": true,
  "user_tier": "free",
  "days_included": 3,
  "location": {
    "latitude": 48.4284,
    "longitude": -123.3656,
    "nearest_station": "Victoria International Airport, BC",
    "distance_km": 12.3
  },
  "forecast": [
    {
      "date": "2025-11-16",
      "timestamp": 1731715200,
      "aurora": {
        "kp_index": 4.5,
        "probability": 65,
        "activity_level": "Active",
        "viewing_quality": "Good"
      },
      "weather": {
        "cloud_cover_percent": 25,
        "sky_condition": "Mostly Clear",
        "precipitation_mm": 0,
        "viewing_conditions": "Excellent"
      },
      "moon": {
        "illumination": 12,
        "phase_name": "Waxing Crescent",
        "age_days": 3.2,
        "darkness_score": 9
      },
      "viewing_score": 14.5,
      "viewing_rating": "Very Good",
      "recommendation": "Great night for aurora viewing. Worth staying up late!",
      "is_recommended_night": true,
      "alert_triggered": true
    }
  ],
  "best_nights": [
    {
      "date": "2025-11-16",
      "score": 14.5,
      "rating": "Very Good"
    }
  ],
  "next_update_at": "2025-11-17T12:00:00Z",
  "generated_at": "2025-11-16T23:15:00Z",
  "cached": false
}
```

**Response (Error - Invalid Membership)**:
```json
{
  "success": false,
  "error": "Invalid membership ID",
  "user_tier": "free",
  "message": "Membership ID not recognized. Returning free tier forecast.",
  "forecast": { /* 3-day forecast */ }
}
```

**Response Codes**:
- 200: Success
- 400: Invalid request (missing lat/lon)
- 401: Unauthorized (invalid/expired JWT)
- 403: Rate limit exceeded
- 500: Server error

---

#### POST /api/alert/acknowledge

**Description**: Record when user acknowledges an alert.

**Request**:
```json
{
  "alert_id": "alert_xyz789",
  "action": "opened" | "dismissed",
  "timestamp": 1731715200
}
```

**Response**:
```json
{
  "success": true,
  "message": "Alert acknowledgment recorded"
}
```

---

### 5.3 Database Schema Updates

**New Table: alerts**
```sql
CREATE TABLE alerts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    alert_type ENUM('immediate', 'advance') NOT NULL,
    forecast_date DATE NOT NULL,
    viewing_score DECIMAL(4,2) NOT NULL,
    kp_index DECIMAL(3,1) NOT NULL,
    location_lat DECIMAL(10, 8) NOT NULL,
    location_lon DECIMAL(11, 8) NOT NULL,
    sent_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    acknowledged_at TIMESTAMP NULL,
    action_taken ENUM('opened', 'dismissed', 'ignored') NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_date (user_id, forecast_date),
    INDEX idx_sent (sent_at)
);
```

**New Table: polling_log**
```sql
CREATE TABLE polling_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NULL,
    polled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    response_time_ms INT NOT NULL,
    cache_hit BOOLEAN NOT NULL,
    error_message TEXT NULL,
    INDEX idx_user (user_id),
    INDEX idx_polled (polled_at),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL
);
```

---

### 5.4 Caching Strategy

**Cache Layers**:

1. **Client-Side Cache** (Browser Extension)
   - Duration: 12 hours
   - Storage: chrome.storage.local
   - Invalidation: On manual refresh or new data from server
   - Size Limit: 5MB

2. **Server-Side Cache** (MySQL)
   - Duration: 1 hour
   - Storage: api_cache table
   - Invalidation: TTL expiry or manual flush
   - Keys:
     - `aurora_forecast_{days}`
     - `weather_{lat}_{lon}_{days}`
     - `moon_phases_{days}`
     - `combined_forecast_{lat}_{lon}_{days}`

3. **API Response Cache** (in-memory)
   - Duration: 5 minutes
   - Storage: PHP APCu or Redis
   - Purpose: Prevent duplicate API calls during high traffic

**Cache Invalidation**:
- Automatic TTL expiry
- Manual refresh button (respects rate limit)
- Webhook from external API (future)
- Cron job clears expired entries hourly

---

### 5.5 Polling Implementation

**Service Worker (background.js)**:

```javascript
// Constants
const POLLING_INTERVAL = 12 * 60 * 60 * 1000; // 12 hours in ms
const MIN_ALERT_SCORE = 14;
const ALERT_COOLDOWN = 6 * 60 * 60 * 1000; // 6 hours

// Initialize polling on install
chrome.runtime.onInstalled.addListener(() => {
  setupPolling();
});

// Setup alarm for 12-hour polling
function setupPolling() {
  chrome.alarms.create('forecastUpdate', {
    periodInMinutes: 720,  // 12 hours
    when: Date.now() + POLLING_INTERVAL
  });
  
  // Also check on browser startup
  checkForecast();
}

// Alarm listener
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'forecastUpdate') {
    checkForecast();
  }
});

// Main forecast check function
async function checkForecast() {
  try {
    // Get user data
    const user = await getUserData();
    if (!user || !user.token) {
      console.log('User not logged in, skipping forecast check');
      return;
    }
    
    // Get location
    const location = await getStoredLocation();
    if (!location) {
      console.log('No location available, skipping forecast check');
      return;
    }
    
    // Fetch forecast from API
    const forecast = await fetchForecast(
      location.latitude,
      location.longitude,
      user.membership_id
    );
    
    // Check for high-probability events
    checkForAlerts(forecast);
    
    // Update badge
    updateBadge(forecast);
    
    // Store forecast
    await storeForecast(forecast);
    
    // Log polling activity
    logPolling({
      success: true,
      user_id: user.id,
      location: location,
      timestamp: Date.now()
    });
    
  } catch (error) {
    console.error('Forecast check failed:', error);
    logPolling({
      success: false,
      error: error.message,
      timestamp: Date.now()
    });
  }
}

// Check if alerts should be triggered
function checkForAlerts(forecast) {
  const now = Date.now();
  const lastAlert = getLastAlertTime();
  
  // Respect cooldown period
  if (lastAlert && (now - lastAlert) < ALERT_COOLDOWN) {
    return;
  }
  
  // Check each forecast day
  for (const day of forecast.forecast) {
    if (day.viewing_score >= MIN_ALERT_SCORE && 
        day.is_recommended_night) {
      
      // Determine alert type
      const daysUntil = getDaysUntil(day.date);
      const alertType = daysUntil === 0 ? 'immediate' : 'advance';
      
      // Send notification
      sendNotification({
        type: alertType,
        date: day.date,
        score: day.viewing_score,
        kp_index: day.aurora.kp_index,
        recommendation: day.recommendation
      });
      
      // Update last alert time
      setLastAlertTime(now);
      
      // Only send one alert per check
      break;
    }
  }
}

// Send browser notification
function sendNotification(alert) {
  const title = alert.type === 'immediate' 
    ? '🌌 Aurora Alert: Excellent Viewing Tonight!'
    : '🌠 Aurora Forecast: Great Conditions Coming!';
  
  const message = `Viewing Score: ${alert.score}/20 | Kp ${alert.kp_index}\n${alert.recommendation}`;
  
  chrome.notifications.create({
    type: 'basic',
    iconUrl: 'icons/icon-128.png',
    title: title,
    message: message,
    priority: 2,
    requireInteraction: true,
    buttons: [
      { title: 'View Forecast' }
    ]
  });
  
  // Log alert sent
  logAlert(alert);
}

// Update extension badge
function updateBadge(forecast) {
  const bestScore = Math.max(...forecast.best_nights.map(n => n.score));
  
  // Set badge text
  chrome.action.setBadgeText({ 
    text: bestScore.toString() 
  });
  
  // Set badge color based on score
  let color;
  if (bestScore >= 16) color = '#ef4444';      // Red - Excellent
  else if (bestScore >= 12) color = '#f97316'; // Orange - Very Good
  else if (bestScore >= 8) color = '#eab308';  // Yellow - Good
  else color = '#6b7280';                      // Gray - Poor
  
  chrome.action.setBadgeBackgroundColor({ color });
}
```

---

### 5.6 Performance Requirements

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| API Response Time (cached) | <200ms | <500ms |
| API Response Time (fresh) | <1.5s | <3s |
| Database Query Time | <50ms | <200ms |
| Extension Load Time | <100ms | <300ms |
| Notification Delivery | <30s | <2min |
| Polling Accuracy | ±5min | ±15min |
| Concurrent Users | 1,000 | 5,000 |
| Cache Hit Rate | >80% | >60% |

---

## 6. User Stories

### Epic 1: User Onboarding

**US1.1**: As a new user, I want to create an account with my email so I can access aurora forecasts.
- **Acceptance**: Can register with valid email, receive confirmation, and login

**US1.2**: As a user, I want to grant location permission so the extension knows where I am.
- **Acceptance**: Browser prompts for location, saves preference, retrieves coordinates

**US1.3**: As a free user, I want to see a 3-day forecast so I can plan aurora viewing.
- **Acceptance**: Forecast displays 3 days with scores, recommendations, and best nights

---

### Epic 2: Automated Monitoring

**US2.1**: As a user, I want automatic forecast updates every 12 hours so I have current information.
- **Acceptance**: Extension polls API every 12 hours, updates even when browser closed

**US2.2**: As a user, I want a notification when viewing conditions are excellent so I don't miss aurora events.
- **Acceptance**: Browser notification appears when score ≥14, includes score and recommendation

**US2.3**: As a user, I want the extension badge to show the best viewing score so I can see conditions at a glance.
- **Acceptance**: Badge displays highest score, color indicates quality (red=excellent, gray=poor)

---

### Epic 3: Subscription Management

**US3.1**: As a free user, I want to see what premium features are available so I can decide to upgrade.
- **Acceptance**: Upgrade banner shows in extension, lists premium benefits

**US3.2**: As a user, I want to purchase a monthly subscription for $4.99 so I get 7-day forecasts.
- **Acceptance**: Stripe payment succeeds, subscription activates immediately, forecasts extend to 7 days

**US3.3**: As a user, I want to purchase lifetime access for $99 so I never pay again.
- **Acceptance**: Payment succeeds, lifetime tier activated, receive gift code

**US3.4**: As a lifetime user, I want to gift my purchase to a friend so they can enjoy it too.
- **Acceptance**: Enter friend's email, they receive redemption link, code works once

---

### Epic 4: Premium Features

**US4.1**: As a premium user, I want to search for any Canadian city so I can check conditions there.
- **Acceptance**: Search box autocompletes Canadian locations, can view forecast for any result

**US4.2**: As a premium user, I want to save favorite locations so I can quickly switch between them.
- **Acceptance**: Can save up to 10 locations, quick switcher in extension, default location setting

**US4.3**: As a premium user, I want to customize alert thresholds so I only get notified for conditions I care about.
- **Acceptance**: Settings page allows adjusting minimum score (12-18), respects setting in future alerts

---

## 7. Non-Functional Requirements

### 7.1 Security
- **Authentication**: JWT tokens with 1-hour expiry, secure refresh mechanism
- **API Security**: Rate limiting (100 requests/hour per user), input validation, SQL injection prevention
- **Payment Security**: PCI-DSS compliant (handled by Stripe), no card data stored
- **Data Privacy**: PIPEDA compliant, user data encrypted at rest, GDPR-ready
- **HTTPS**: All communications over TLS 1.3+
- **CORS**: Restricted to extension origin only

### 7.2 Performance
- See section 5.6 Performance Requirements

### 7.3 Reliability
- **Uptime**: 99.5% availability (max 3.65 hours downtime/month)
- **Data Backup**: Daily automated MySQL backups, 30-day retention
- **Disaster Recovery**: 4-hour RTO (Recovery Time Objective), 1-hour RPO (Recovery Point Objective)
- **Error Handling**: Graceful degradation, user-friendly error messages, automatic retry with exponential backoff

### 7.4 Scalability
- **Horizontal Scaling**: API server can scale to multiple instances behind load balancer
- **Database**: Master-slave replication ready, connection pooling
- **Caching**: Redis/Memcached ready for distributed cache
- **CDN**: Static assets served via CDN

### 7.5 Compatibility
- **Browsers**: Firefox 109+, Chrome 109+, Edge 109+ (Manifest V3)
- **Operating Systems**: Windows 10+, macOS 11+, Linux (Ubuntu 20.04+)
- **Mobile**: Not supported in v1.0 (future)

### 7.6 Accessibility
- **WCAG**: Level AA compliance
- **Keyboard Navigation**: Full keyboard support
- **Screen Readers**: ARIA labels, semantic HTML
- **Color Contrast**: 4.5:1 minimum ratio

---

## 8. Data Models

### 8.1 User
```
User {
  id: integer (PK)
  email: string (unique, indexed)
  password_hash: string
  subscription_tier: enum('free', 'premium', 'lifetime')
  stripe_customer_id: string (nullable)
  membership_id: string (unique, generated)
  created_at: timestamp
  updated_at: timestamp
  last_login: timestamp
}
```

### 8.2 Subscription
```
Subscription {
  id: integer (PK)
  user_id: integer (FK)
  stripe_subscription_id: string
  status: enum('active', 'canceled', 'past_due', 'lifetime')
  plan_type: enum('monthly', 'lifetime')
  current_period_start: timestamp
  current_period_end: timestamp
  created_at: timestamp
}
```

### 8.3 Alert
```
Alert {
  id: integer (PK)
  user_id: integer (FK)
  alert_type: enum('immediate', 'advance')
  forecast_date: date
  viewing_score: decimal
  kp_index: decimal
  location_lat: decimal
  location_lon: decimal
  sent_at: timestamp
  acknowledged_at: timestamp (nullable)
  action_taken: enum('opened', 'dismissed', 'ignored')
}
```

### 8.4 Forecast (Cached)
```
CachedForecast {
  cache_key: string (PK)
  latitude: decimal
  longitude: decimal
  days: integer
  forecast_data: json
  expires_at: timestamp
  created_at: timestamp
}
```

---

## 9. Success Metrics

### 9.1 User Metrics
| Metric | Target (Month 3) | Target (Month 6) | Target (Month 12) |
|--------|------------------|------------------|-------------------|
| Total Registered Users | 500 | 1,500 | 5,000 |
| Daily Active Users (DAU) | 150 | 500 | 1,500 |
| Monthly Active Users (MAU) | 400 | 1,200 | 4,000 |
| DAU/MAU Ratio | 38% | 42% | 38% |

### 9.2 Engagement Metrics
| Metric | Target |
|--------|--------|
| Notifications Enabled | 60% of users |
| Notification Open Rate | 40% |
| Average Session Duration | 2-5 minutes |
| Sessions Per Week | 3-5 |
| Manual Refresh Rate | <10% of forecasts |

### 9.3 Conversion Metrics
| Metric | Target |
|--------|--------|
| Free → Premium Conversion | 10% |
| Free → Lifetime Conversion | 2% |
| Trial Abandonment | <20% |
| Monthly Churn Rate | <5% |

### 9.4 Revenue Metrics
| Metric | Month 3 | Month 6 | Month 12 |
|--------|---------|---------|----------|
| MRR (Monthly Recurring) | $500 | $1,500 | $5,000 |
| Lifetime Sales | $1,000 | $3,000 | $10,000 |
| Total ARR | $15,000 | $30,000 | $70,000 |

### 9.5 Technical Metrics
| Metric | Target |
|--------|--------|
| API Uptime | 99.5% |
| P95 Response Time | <500ms |
| Error Rate | <0.5% |
| Cache Hit Rate | >80% |
| Alert Delivery Success | >98% |
| Forecast Accuracy | >75% |

---

## 10. Development Roadmap

### Phase 1: MVP (Weeks 1-4) - COMPLETE ✅
- ✅ User authentication (JWT)
- ✅ Basic 3/7-day forecasts
- ✅ Stripe integration (monthly + lifetime)
- ✅ Database schema
- ✅ Browser extension UI
- ✅ Initial deployment

### Phase 2: Polling & Alerts (Weeks 5-6) - IN PROGRESS 🔨
- 🔨 Implement 12-hour polling system
- 🔨 Build alert detection logic
- 🔨 Integrate browser notifications
- 🔨 Add badge updates
- 🔨 Implement alert acknowledgment tracking
- 🔨 Create polling/alert logs

**Deliverables**:
- Service worker with alarm-based polling
- Alert trigger algorithm
- Notification UI
- Badge display logic
- Admin dashboard for alert monitoring

### Phase 3: Premium Features (Weeks 7-9)
- Location search functionality
- Saved locations management
- Alert customization settings
- Historical accuracy tracking
- Enhanced forecast details

**Deliverables**:
- Location search API
- Settings page in extension
- Accuracy dashboard
- Premium feature gates

### Phase 4: Polish & Launch (Weeks 10-12)
- Performance optimization
- Bug fixes from beta testing
- Documentation
- Marketing materials
- Browser store submissions
- Public launch

**Deliverables**:
- Store listings
- User documentation
- Marketing website
- Press kit

### Phase 5: Post-Launch (Months 4-6)
- User feedback implementation
- Mobile app development
- Social features
- Partnership integrations (tourism boards, photography groups)
- API for third-party developers

---

## 11. Risks & Mitigation

### R1: External API Reliability
**Risk**: NOAA, Environment Canada, or USNO APIs go down
**Impact**: High - No forecast data available
**Mitigation**: 
- Implement fallback APIs (multiple weather sources)
- Cache last known good data (up to 24 hours)
- Display cached data with disclaimer
- Set up API monitoring and alerts

### R2: Alert Fatigue
**Risk**: Users disable notifications due to too many false positives
**Impact**: Medium - Reduced engagement
**Mitigation**:
- Conservative alert thresholds (≥14 score)
- 6-hour cooldown between alerts
- User-customizable settings
- A/B test alert frequency

### R3: Low Conversion Rate
**Risk**: Free users don't see value in premium
**Impact**: High - Revenue target miss
**Mitigation**:
- Clear value proposition in upgrade prompts
- Limited-time promotional pricing
- Referral program
- Showcase premium features during high-activity events

### R4: Browser Extension Rejection
**Risk**: Firefox/Chrome stores reject submission
**Impact**: Medium - Delayed launch
**Mitigation**:
- Follow all store guidelines strictly
- Test on multiple browsers
- Prepare appeals documentation
- Have web app alternative ready

### R5: Forecast Inaccuracy
**Risk**: Predictions don't match actual conditions
**Impact**: High - Loss of trust
**Mitigation**:
- Display confidence intervals
- Collect user feedback on accuracy
- Continuously tune algorithm
- Set appropriate expectations (not 100% accurate)

### R6: Server Costs Exceed Budget
**Risk**: API calls and storage costs grow faster than revenue
**Impact**: Medium - Unsustainable business
**Mitigation**:
- Aggressive caching strategy
- Rate limiting on free tier
- Monitor costs weekly
- Scale horizontally only when needed

---

## 12. Open Questions

### Technical
- Q1: Should polling happen even when user is offline? 
  - **Decision needed**: Queue forecasts for when user comes back online?
  
- Q2: How to handle timezone differences for alerts?
  - **Decision needed**: Use user's local timezone or UTC?
  
- Q3: Should we store full forecast history?
  - **Decision needed**: How much historical data to retain? (30 days? 1 year?)

### Product
- Q4: What's the minimum score for "good" viewing?
  - **Recommendation**: A/B test 12 vs 14 as alert threshold

- Q5: Should free users get any alerts at all?
  - **Recommendation**: Yes, but only for score ≥16 (exceptional events)

- Q6: How to prevent gaming the gift system?
  - **Decision needed**: IP tracking? Email domain restrictions?

### Business
- Q7: Should we offer annual subscription discount?
  - **Recommendation**: $49.99/year (17% savings over monthly)

- Q8: Partnership opportunities with camera manufacturers?
  - **To explore**: Canon, Nikon, Sony co-marketing

- Q9: Should we build API for B2B customers (tour operators)?
  - **Recommendation**: Phase 5+ feature

---

## 13. Glossary

**Aurora Borealis**: Northern Lights; natural light display in Earth's sky, predominantly seen in high-latitude regions

**Kp Index**: Quasi-logarithmic scale (0-9) measuring global geomagnetic activity. Higher = more intense aurora.

**Geomagnetic Storm**: Disturbance in Earth's magnetosphere caused by solar wind. Produces aurora.

**JWT (JSON Web Token)**: Compact, URL-safe token for authentication

**Manifest V3**: Latest version of Chrome extension manifest specification

**Service Worker**: Background script that runs independently of web pages

**Stripe**: Payment processing platform for online businesses

**PIPEDA**: Personal Information Protection and Electronic Documents Act (Canadian privacy law)

**TLS**: Transport Layer Security - protocol for encrypted communications

**RTO/RPO**: Recovery Time/Point Objective - disaster recovery metrics

---

## Appendix A: API Rate Limits

| Tier | Requests/Hour | Requests/Day | Manual Refresh Limit |
|------|---------------|--------------|---------------------|
| Free | 100 | 500 | 1 per 10 minutes |
| Premium | 500 | 2,000 | 1 per 5 minutes |
| Lifetime | 1,000 | 5,000 | 1 per 2 minutes |

Rate limit headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1731715200
```

---

## Appendix B: Notification Templates

**Immediate Alert (Tonight)**:
```
Title: 🌌 Aurora Alert: Excellent Viewing Tonight!
Body: Viewing Score: 16/20 | Kp 6.2
Great conditions with clear skies and new moon. 
Best viewing: 11 PM - 2 AM
[View Forecast]
```

**Advance Alert (Tomorrow)**:
```
Title: 🌠 Aurora Forecast: Tomorrow Night Looks Great!
Body: Viewing Score: 17/20 | Kp 7.0
Strong aurora activity expected with clear weather.
[View Forecast]
```

**Weekly Summary (Premium)**:
```
Title: 📊 Your Weekly Aurora Forecast
Body: 3 excellent nights coming up:
• Tomorrow: 16/20
• Thursday: 15/20  
• Saturday: 14/20
[View Details]
```

---

## Appendix C: Error Messages

**User-Facing Errors**:
- "Unable to get your location. Please enable location permissions."
- "Forecast temporarily unavailable. Showing cached data from [time]."
- "Your subscription has expired. Upgrade to continue receiving 7-day forecasts."
- "Rate limit exceeded. Please try again in [minutes]."
- "We're having trouble connecting. Please check your internet connection."

**Developer Errors** (logged only):
- "NOAA API timeout after 10s"
- "JWT token expired: [token_id]"
- "Database connection pool exhausted"
- "Stripe webhook signature verification failed"
- "Cache write failed: disk full"

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Nov 2025 | Web321 | Initial MVP spec |
| 2.0 | Nov 2025 | Web321 | Added polling, alerts, membership_id request format |

**Approval**:
- Product Owner: Shawn, Web321 Marketing Ltd.
- Technical Lead: _Pending_
- Stakeholders: _Pending_

**Next Review Date**: Dec 2025

---

## Contact

**Product Owner**: Shawn  
**Company**: Web321 Marketing Ltd.  
**Location**: Saanichton, British Columbia, Canada  
**Email**: shawn@web321.co  

---

*Aurora Hunter: Perfect nights for perfect lights* 🌌

END OF DOCUMENT
