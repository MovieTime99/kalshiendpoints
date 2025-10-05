# Kalshi API Documentation

Complete documentation of all Kalshi API endpoints used in the trading data collection system.

## Base URL
```
https://api.elections.kalshi.com/v1
```

## Authentication

### Public Endpoints
Most endpoints are public and don't require authentication.

### Private Endpoints
Private endpoints require session authentication with:
- `cookie`: Session token (format: `sessions=...`)
- `x-csrf-token`: CSRF protection token
- `origin`: https://kalshi.com
- `referer`: https://kalshi.com/

## Endpoints

### 1. User Profile Holdings
**Endpoint:** `GET /social/profile/holdings`

**Description:** Fetches a user's current positions and holdings across all markets.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| nickname | string | Yes | Username of the trader |
| limit | integer | No | Maximum holdings to return (default: 100) |

**Response Schema:**
```json
{
  "holdings": [
    {
      "event_ticker": "string",           // Event identifier (e.g., "KXMLBNLWEST-25")
      "total_absolute_position": number,  // Total position size in contracts
      "market_holdings": [
        {
          "market_ticker": "string",       // Specific market (e.g., "KXMLBNLWEST-25-AZ")
          "signed_open_position": number,  // Position size (+long, -short)
          "pnl": number                    // Current profit/loss on position
        }
      ]
    }
  ]
}
```

**Example:**
```
GET https://api.elections.kalshi.com/v1/social/profile/holdings?nickname=Delta0x&limit=100
```

---

### 2. User Profile Metrics
**Endpoint:** `GET /social/profile/metrics`

**Description:** Returns trading metrics for a user over specified time period.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| nickname | string | Yes | Username of the trader |
| since_day_before | integer | Yes | Days back (0=all-time, 1=24hr, 7=week, 30=month) |

**Response Schema:**
```json
{
  "metrics": {
    "pnl": number,                    // Profit/loss in basis points (divide by 10000 for dollars)
    "volume": number,                 // Trading volume in cents
    "num_markets_traded": number,     // Number of unique markets traded
    "portfolio_value": number,        // Current portfolio value (private only)
    "num_trades": number,             // Total number of trades
    "sharpe_ratio": number           // Risk-adjusted return metric
  }
}
```

**Example:**
```
GET https://api.elections.kalshi.com/v1/social/profile/metrics?nickname=Delta0x&since_day_before=0
```

---

### 3. User Social Profile
**Endpoint:** `GET /social/profile`

**Description:** Fetches user profile information including bio, followers, and join date.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| nickname | string | Yes | Username of the trader |

**Response Schema:**
```json
{
  "social_profile": {
    "user_id": "string",              // UUID of the user
    "nickname": "string",             // Username
    "joined_at": "string",            // ISO date (e.g., "2024-10-08T00:00:00Z")
    "description": "string",          // User bio
    "follower_count": number,         // Number of followers
    "following_count": number,        // Number following
    "profile_image_path": "string",   // URL to profile image
    "is_verified": boolean,           // Verification status
    "is_whitelisted": boolean        // Featured/whitelisted status
  }
}
```

**Example:**
```
GET https://api.elections.kalshi.com/v1/social/profile?nickname=Delta0x
```

---

### 4. Leaderboard
**Endpoint:** `GET /social/leaderboard`

**Description:** Returns ranked leaderboard of top traders by various metrics.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| metric_name | string | Yes | Metric type: `projected_pnl`, `volume`, `num_markets_traded` |
| since_day_before | integer | Yes | Time period (0=all-time, 1=24hr, 7=week, 30=month) |
| limit | integer | No | Number of results (default: 100, max: 100) |
| page | integer | No | Page number for pagination (default: 1) |
| category | string | No | Market category filter (see categories below) |

**Categories:**
- Politics
- Sports
- Culture
- Crypto
- Climate
- Economics
- Companies
- Financials
- Science+and+Technology (URL encoded)
- Health
- World
- Elections

**Response Schema:**
```json
{
  "rank_list": [
    {
      "rank": number,                  // Leaderboard position
      "nickname": "string",            // Username
      "value": number,                 // Metric value (PnL in basis points, volume in cents)
      "profile_image_path": "string"   // Profile image URL
    }
  ],
  "total_count": number,              // Total users on leaderboard
  "has_more": boolean                 // More pages available
}
```

**Examples:**
```
# Top PnL traders all-time
GET https://api.elections.kalshi.com/v1/social/leaderboard?metric_name=projected_pnl&since_day_before=0&limit=100

# Top volume traders in Politics last 24 hours
GET https://api.elections.kalshi.com/v1/social/leaderboard?metric_name=volume&since_day_before=1&category=Politics&limit=50

# Most active traders (by markets) this week
GET https://api.elections.kalshi.com/v1/social/leaderboard?metric_name=num_markets_traded&since_day_before=7&limit=100
```

---

### 5. Whitelisted Profiles
**Endpoint:** `GET /social/whitelisted_profiles`

**Description:** Returns list of featured/verified traders curated by Kalshi.

**Parameters:** None

**Response Schema:**
```json
{
  "profiles": [
    {
      "nickname": "string",            // Username
      "user_id": "string",             // UUID
      "description": "string",         // Bio
      "profile_image_path": "string",  // Profile image
      "is_verified": true,            // Always true for whitelisted
      "follower_count": number
    }
  ]
}
```

**Example:**
```
GET https://api.elections.kalshi.com/v1/social/whitelisted_profiles
```

---

### 6. User Search
**Endpoint:** `GET /search/social_profiles`

**Description:** Search for users by username or display name.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| query | string | Yes | Search query (URL encoded) |
| limit | integer | No | Maximum results (default: 100) |

**Response Schema:**
```json
{
  "results": [
    {
      "nickname": "string",            // Username
      "user_id": "string",            // UUID
      "profile_image_path": "string",  // Profile image
      "description": "string",         // Bio snippet
      "follower_count": number
    }
  ],
  "total_matches": number
}
```

**Example:**
```
GET https://api.elections.kalshi.com/v1/search/social_profiles?query=Delta&limit=50
```

---

### 7. User Timeline (Private)
**Endpoint:** `GET /users/{userId}/social/timeline`

**Description:** Fetches user's timeline feed with posts and comments. **Requires authentication.**

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| userId | string | Yes | Your user UUID (in path) |
| limit | integer | No | Posts per request (default: 100) |
| filter | string | No | Feed type: `recent`, `hot` |
| timeframe | string | No | Time period: `daily`, `weekly`, `monthly` |
| include_comments | boolean | No | Include comment threads (default: false) |
| comments_max_depth | integer | No | Comment nesting depth (default: 3) |
| cursor | string | No | Pagination cursor from previous response |

**Response Schema:**
```json
{
  "posts": [
    {
      "post_id": "string",
      "author": {
        "nickname": "string",
        "user_id": "string"
      },
      "content": "string",             // Post text
      "created_at": "string",          // ISO timestamp
      "likes_count": number,
      "comments_count": number,
      "market_ticker": "string",       // Related market (optional)
      "comments": []                   // Nested comment threads
    }
  ],
  "next_cursor": "string"             // For pagination
}
```

**Example:**
```
GET https://api.elections.kalshi.com/v1/users/{userId}/social/timeline?limit=100&filter=hot&timeframe=daily
```

---

## Value Conversions

### PnL (Profit/Loss)
- API returns PnL in **basis points** (1/100 of a cent)
- To convert to dollars: `pnl_dollars = pnl_basis_points / 10000`
- Example: 5000000 basis points = $500.00

### Volume
- API returns volume in **cents**
- To convert to dollars: `volume_dollars = volume_cents / 100`
- Example: 150000 cents = $1,500.00

### Dates
- All dates are in ISO 8601 format
- Example: `2024-10-08T15:30:00Z`

## Rate Limiting

### Recommendations
- Add 1-second delay between API calls
- Use parallel requests with proxies for bulk operations
- Maximum 100 items per request for paginated endpoints

### Error Codes
| Code | Description | Solution |
|------|-------------|----------|
| 200 | Success | - |
| 401 | Unauthorized | Update authentication cookies |
| 403 | Forbidden | Check CSRF token, possible rate limit |
| 404 | Not Found | User/resource doesn't exist |
| 429 | Rate Limited | Reduce request frequency |
| 500 | Server Error | Retry with exponential backoff |

## Usage Examples

### Get Top Traders with Full Profile
```javascript
// 1. Fetch leaderboard
const leaderboard = await fetch(
  'https://api.elections.kalshi.com/v1/social/leaderboard?metric_name=projected_pnl&since_day_before=0&limit=10'
);

// 2. For each user, get detailed profile
for (const user of leaderboard.rank_list) {
  const profile = await fetch(
    `https://api.elections.kalshi.com/v1/social/profile?nickname=${user.nickname}`
  );
  const metrics = await fetch(
    `https://api.elections.kalshi.com/v1/social/profile/metrics?nickname=${user.nickname}&since_day_before=0`
  );
}
```

### Track User PnL Over Time
```javascript
const username = 'Delta0x';

// Get different time period metrics
const allTime = await fetch(`/social/profile/metrics?nickname=${username}&since_day_before=0`);
const monthly = await fetch(`/social/profile/metrics?nickname=${username}&since_day_before=30`);
const weekly = await fetch(`/social/profile/metrics?nickname=${username}&since_day_before=7`);
const daily = await fetch(`/social/profile/metrics?nickname=${username}&since_day_before=1`);

// Calculate period performance
const monthlyPnL = (monthly.metrics.pnl - weekly.metrics.pnl) / 10000; // Last 23 days
const weeklyPnL = (weekly.metrics.pnl - daily.metrics.pnl) / 10000;   // Last 6 days
const dailyPnL = daily.metrics.pnl / 10000;                           // Last 24 hours
```

### Find Category Specialists
```javascript
const categories = ['Politics', 'Sports', 'Crypto', 'Economics'];

for (const category of categories) {
  const specialists = await fetch(
    `/social/leaderboard?metric_name=projected_pnl&since_day_before=30&category=${category}&limit=10`
  );
  console.log(`Top ${category} traders:`, specialists.rank_list);
}
```

## Notes

- User IDs are UUIDs that remain constant even if username changes
- Leaderboards only show positive PnL users; use volume/markets leaderboards to find traders with losses
- Timeline API provides the most comprehensive user discovery but requires authentication
- Whitelisted profiles are Kalshi's featured/verified traders
- All monetary values need conversion from basis points or cents to dollars