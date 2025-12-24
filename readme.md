# n8n Weather Automation Workflow

Daily weather automation that fetches data from OpenWeatherMap, saves it to Supabase, and sends email notifications.

## What It Does
- Runs automatically at 8:00 AM every day
- Gets weather data for a city
- Checks for rain, extreme heat, or freezing temperatures
- Saves everything to a database
- Sends a formatted email with the weather report

## How It Works

```
Schedule (8 AM daily)
    ↓
Get Weather (OpenWeatherMap API)
    ↓
Process Data (JavaScript logic)
    ↓
Save to Database (Supabase)
    ↓
Send Email (SMTP)
```

## What You Need

1. **n8n account** - https://n8n.io/ (free)
2. **OpenWeatherMap API key** - https://openweathermap.org/api (free)
3. **Supabase account** - https://supabase.com (free)
4. **Gmail account** - For sending emails

## Setup

### 1. Create Supabase Database

Log into Supabase, go to SQL Editor, and run this:

```sql
CREATE TABLE weather_logs (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  run_at TIMESTAMP DEFAULT NOW(),
  city TEXT NOT NULL,
  temperature FLOAT,
  temperature_unit TEXT,
  condition TEXT,
  humidity INT,
  wind_speed FLOAT,
  alert_type TEXT,
  raw_response JSONB
);

CREATE INDEX idx_weather_logs_city ON weather_logs(city);
CREATE INDEX idx_weather_logs_run_at ON weather_logs(run_at DESC);
```

Then get your credentials from Project Settings → API:
- Copy the **Project URL**
- Copy the **service_role key** (not the anon key)

### 2. Get OpenWeatherMap API Key

1. Sign up at https://openweathermap.org/api
2. Go to API Keys section
3. Copy your API key

### 3. Setup Gmail

Gmail needs an app password, not your regular password:

1. Enable 2-Factor Authentication on your Google account
2. Go to https://myaccount.google.com/apppasswords
3. Create app password for "Mail"
4. Copy the 16-character password

You'll use these settings:
- Host: smtp.gmail.com
- Port: 465
- SSL: Enabled
- User: your email
- Password: the app password you just created

### 4. Import to n8n

1. Log into n8n
2. Click Workflows → Add workflow
3. Click the three dots (⋯) → Import from File
4. Upload `Daily Weather Automation.json`

### 5. Add Your Credentials

**OpenWeatherMap:**
1. Click "Fetch Weather Data" node
2. Create new credential
3. Paste your API key

**Supabase:**
1. Click "Save to Supabase" node
2. Create new credential
3. Enter your Project URL and service_role key

**Email:**
1. Click "Send Weather Email" node
2. Create new SMTP credential
3. Enter Gmail settings from step 3

### 6. Update Settings

**Change email addresses:**
- Click "Send Weather Email" node
- Update "From Email" and "To Email"

**Change city (optional):**
- Click "Fetch Weather Data" node
- Change the `q` parameter from "London" to your city

**Change schedule (optional):**
- Click "Schedule Trigger" node
- Current: `0 8 * * *` (8 AM daily)
- Change to something like `0 7 * * *` for 7 AM

## Testing

1. Click "Execute Workflow" button
2. Check that all nodes turn green
3. Look at your email - you should get a weather report
4. Check Supabase table - you should see a new row

If something fails, click on the red node to see the error.

## Activate It

Click the **Publish** button to make it run automatically every day at 8 AM.

## Weather Alerts

The workflow checks for:
- **Rain/Snow/Storm** - alerts if these words appear in the weather condition
- **Heat** - alerts if temperature > 32°C
- **Frost** - alerts if temperature < 0°C

## Database Schema

```sql
weather_logs (
  id              UUID PRIMARY KEY,
  run_at          TIMESTAMP,
  city            TEXT,
  temperature     FLOAT,
  temperature_unit TEXT,
  condition       TEXT,
  humidity        INT,
  wind_speed      FLOAT,
  alert_type      TEXT,
  raw_response    JSONB
)
```

## Email Format

The workflow sends HTML-formatted emails with:
- Current temperature with unit
- Weather condition description
- Humidity percentage
- Wind speed in km/h
- Alert notifications (when applicable)
- Clean, professional styling

## Data Flow

1. **Schedule Trigger**: Fires at 8:00 AM daily (cron: `0 8 * * *`)
2. **HTTP Request**: Calls OpenWeatherMap API with city parameter
3. **Code Node**: 
   - Parses JSON response
   - Extracts weather metrics
   - Converts wind speed from m/s to km/h
   - Determines alert type based on conditions
   - Formats summary message
4. **Supabase Node**: Inserts structured data into `weather_logs` table
5. **Email Node**: Sends HTML email with weather summary and alerts

## Troubleshooting

**Weather API not working?**
- Check your API key is correct
- Make sure city name is spelled right

**Supabase errors?**
- Verify you're using the service_role key, not anon key
- Check the table exists

**Email not sending?**
- Use the app password, not your regular Gmail password
- Make sure port is 465 and SSL is enabled
- Check spam folder

**Still stuck?**
- Click on the failed node to see the error message
- Check the Executions tab for details

## View Your Data

Go to Supabase → Table Editor → weather_logs to see all your weather history.

Some example queries:

```sql
-- Recent weather
SELECT * FROM weather_logs 
ORDER BY run_at DESC 
LIMIT 10;

-- All heat alerts
SELECT city, temperature, run_at 
FROM weather_logs 
WHERE alert_type = 'heat';

-- Average temperature
SELECT city, AVG(temperature) as avg_temp
FROM weather_logs
GROUP BY city;
```

## What's Included

- Complete n8n workflow with 5 nodes
- Database schema for storing weather data
- HTML email template with weather info
- Alert logic for severe weather
- All setup instructions

## Tech Used

- n8n for workflow automation
- OpenWeatherMap API for weather data
- Supabase (PostgreSQL) for storage
- Gmail SMTP for email delivery
- JavaScript for processing logic