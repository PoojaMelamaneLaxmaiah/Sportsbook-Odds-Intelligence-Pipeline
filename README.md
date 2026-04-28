# Sportsbook Odds Intelligence Pipeline

> Automated pipeline that fetches, classifies, and logs live sportsbook
> odds data every 15 minutes — built with Make.com, PHP/Laravel, MySQL,
> and R for time-series analysis.

## What it does

- Fetches live market data from a sportsbook odds API via HTTP module
- Classifies each market event by odds value using a Router (high / standard / low)
- Logs structured rows to a MySQL database and Google Sheets with timestamp
- Monitors odds movement over time using pulled_at timestamps
- Sends email alerts automatically if the pipeline fails (3x retry + fallback)

## Architecture

```
Schedule (15 min)
    → HTTP GET: Odds API
    → Iterator: split array into one bundle per market
    → Filter: active markets only
    → Router
        → Branch A: odds > 2.5  → Log to "high_value" table
        → Branch B: odds 1.5–2.5 → Log to "standard" table
        → Branch C: odds < 1.5   → Log to "favourite" table
    → Error handler: Retry × 3 → Email alert → Error log table
```

## Tech stack

| Layer | Technology |
|---|---|
| Automation | Make.com (Integromat) |
| Backend | PHP 8.1 / Laravel 10 |
| Database | MySQL 8.0 |
| Analysis | R (dplyr, ggplot2) |
| Reporting | Google Sheets, Power BI |

## Key SQL queries

```sql
-- Consolidate market, event, and line data
SELECT
    m.market_id,
    m.market_name,
    e.event_id,
    e.event_name,
    o.side,
    o.odds_value,
    o.pulled_at
FROM markets m
JOIN events e ON e.market_id = m.market_id
JOIN odds o   ON o.event_id  = e.event_id
WHERE m.status = 'active'
ORDER BY o.pulled_at DESC;

-- Pivot odds by side (Home / Away / Draw)
SELECT
    event_id,
    MAX(CASE WHEN side = 'Home' THEN odds_value END) AS home_odds,
    MAX(CASE WHEN side = 'Away' THEN odds_value END) AS away_odds,
    MAX(CASE WHEN side = 'Draw' THEN odds_value END) AS draw_odds,
    MAX(pulled_at) AS last_updated
FROM odds
GROUP BY event_id;
```

## Setup

```bash
# 1. Clone the repository
git clone https://github.com/poojamelamanelaxmaiah/Betting-Automation-Solutions.git
cd Betting-Automation-Solutions

# 2. Install dependencies
composer install

# 3. Configure environment
cp .env.example .env
# Edit .env with your database credentials and API key

# 4. Run database migrations
php artisan migrate

# 5. Import sample data (anonymised)
php artisan db:seed --class=OddsSeeder
```

## Environment variables

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=odds_pipeline
DB_USERNAME=your_username
DB_PASSWORD=your_password

ODDS_API_KEY=your_api_key_here
ODDS_API_URL=https://api.example.com/v1/odds
```

## What I learned building this

- How to structure multi-table relational schemas for time-series data
- Iterator vs collection mode in Make.com for different API response formats
- SQL PIVOT pattern using conditional aggregation (CASE WHEN)
- Error handling design: retry logic + fallback logging + alerting

## Author

**Pooja Melamane** — Data Analyst & Automation Engineer, Malta
[linkedin.com/in/poojaml](https://linkedin.com/in/poojaml) · [pooja.melamane@proton.me](mailto:pooja.melamane@proton.me)
