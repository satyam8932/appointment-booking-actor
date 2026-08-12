# Appointment Booking Verification - GHL

Verifies appointment booking in GoHighLevel by finding "Appointment [Lead Name] created" message in conversation and taking a screenshot.

## Actor IDs

- **Actor ID:** `43L8lc3WSCJ88Frz5`
- **Platform:** Apify Cloud

## Setup

```bash
cd appointment-booking-actor
npm install
```

## Refresh Cookies (do this every few days)

```bash
# Set INPUT.json to login mode
echo '{"loginMode": true, "subAccountUrl": "https://app.tjbdigitalservices.com/v2/location/ANY_LOCATION_ID/dashboard", "leadName": "test"}' > storage/key_value_stores/default/INPUT.json

# Run - browser opens, log in, reach any dashboard page
npx tsx src/main.ts

# Cookies saved to storage-state.json
# OR just copy from unscheduled-call-complete-actor:
cp ../unscheduled-call-complete-actor/storage-state.json ./storage-state.json
```

Both actors share the same GHL session so you only need to login once from either actor.

## Run Locally

```bash
# Set input
echo '{"subAccountUrl": "https://app.tjbdigitalservices.com/v2/location/LOCATION_ID/dashboard", "leadName": "Lead Name"}' > storage/key_value_stores/default/INPUT.json

npx tsx src/main.ts
```

## Run on Cloud (API call)

```json
{
  "subAccountUrl": "https://app.tjbdigitalservices.com/v2/location/LOCATION_ID/dashboard",
  "leadName": "Lead Name",
  "storageState": { /* contents of storage-state.json */ }
}
```

Cloud requires `storageState` field with full cookie JSON.

## Deploy

```bash
APIFY_TOKEN=your_token apify push --force
apify builds add-tag -b BUILD_ID -t latest
```

## Input

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| subAccountUrl | string | yes | GHL sub-account dashboard URL |
| leadName | string | yes | Lead name to search |
| storageState | object | cloud only | Browser session JSON |
| loginMode | boolean | no | Opens browser for manual login |

## Output

```json
{
  "appointmentFound": true,
  "leadFound": true,
  "leadName": "Warren Vann",
  "screenshotUrl": "https://api.apify.com/v2/key-value-stores/STORE_ID/records/appointment-screenshot",
  "error": null
}
```

## How it works

1. Navigates to sub-account dashboard
2. Searches lead via global search (smart name matching, skips duplicates with "(2)" suffix)
3. Opens lead conversation
4. Closes activity sidebar panel
5. Looks for "Appointment [Lead Name] created" text (also tries first-name-only and generic "Appointment")
6. Scrolls up/down if not immediately visible
7. Positions text at 40% from top of viewport for clean framing
8. Takes screenshot, returns public URL

## Config

- **RAM:** 2048MB
- **Timeout:** 3600s
