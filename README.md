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
5. Detects the element that actually scrolls the conversation (see below)
6. Scrolls to the very end of the chat and keeps re-asserting the bottom until the thread stops growing
7. Looks for "Appointment [Lead Name] created" text (also tries first-name-only and generic "Appointment"); if it is not at the bottom, scans upwards to confirm it exists, then returns to the bottom
8. Re-confirms the bottom immediately before each screenshot attempt, then captures and returns the public URL

### Scrolling to the end of the chat

The screenshot is always taken at the bottom of the conversation. Getting there reliably needs
three things, each of which used to fail intermittently:

- **Finding the real scroller.** The scroll container is detected by probing for elements that
  genuinely overflow (`overflow-y: auto/scroll` and `scrollHeight > clientHeight`), scored by size
  and message-shaped children, and restricted to the main content column. Matching a selector like
  `[class*="conversation"]` is not enough — that matches non-scrollable wrappers and the
  `hl_conversations--list` contacts rail, and scrolling those moves the chat nowhere.
- **Not stopping early.** Messages load lazily, so one `scrollTop = scrollHeight` lands at what was
  the bottom a moment ago. The actor re-asserts the bottom until both the height and the position
  hold steady for three consecutive rounds, falling back to real wheel/`End` input if programmatic
  scrolling stalls.
- **Staying at the bottom.** The bottom is re-confirmed right before every screenshot attempt, so
  late-arriving content cannot leave the capture parked mid-thread.

A conversation short enough to fit on screen has no scroll container; that is treated as already
being at the end rather than as a failure.

## Config

- **RAM:** 2048MB
- **Timeout:** 3600s
