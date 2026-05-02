# BVG Journey Planner — Conversation Summary

## Project
Single-file HTML+JS website (`index.html`) for journey planning using the Berlin BVG transport open API (no auth required). Hostable for free on GitHub Pages / Netlify.

## Key Technical Details

### BVG API Endpoints
- `/locations?query=...` — autocomplete / location search
- `/journeys?from=<stopId>&to=<stopId>` — journey search (requires stop IDs, NOT address IDs)
- `/stops/reachable-from?address=<addr>&latitude=<lat>&longitude=<lon>` — resolve address to nearest stop ID. Response: `data.reachable[0].stations[0].id`
- `/stops/{id}` — stop details

### Address-to-Stop Resolution
- `/journeys` requires stop IDs; address IDs return 404.
- `/stops/reachable-from` needs ALL THREE params: `address`, `latitude`, `longitude`.
- Response structure: `data.reachable[0].stations[0].id`

### Key Functions in index.html
- `locDisplayName(loc)` — `loc.name || loc.address || ''`
- `resolveToStopId(loc)` — returns stop ID directly if type is 'stop'/'station', otherwise calls `/stops/reachable-from`
- `fetchJourneys(fromId, toId, laterThan)` — API call with optional pagination
- `renderJourney(j)` — journey card with line summary, countdown, duration
- `renderLeg(leg)` — individual leg with icon, line+headsign, times+delays, duration, platforms, stopovers
- `legIcon(leg)` — S=green, R=red, U=blue, bus=yellow
- `legDurationSec(leg)` — uses `leg.duration` or computes from departure/arrival time difference (fallback for 0/null)
- `formatTimeWithDelay(plannedIso, actualIso, delaySec)` — planned time with `(+X min)` suffix
- `loadMore()` — pagination trigger via IntersectionObserver on sentinel div
- `getFavorites()` / `saveFavorites(favs)` / `addFavorite(loc, customName)` / `removeFavorite(id)` — cookie-based persistence, 1-year expiry
- `buildSuggestionItem(loc, field, isFav)` — autocomplete dropdown with fav button; custom name via `prompt()`

### CSS Color Coding
- `.leg-icon.train` — red `#e74c3c` (R/RE)
- `.leg-icon.strain` — green `#2ecc71` (S)
- `.leg-icon.subway` — blue `#3498db` (U)
- `.leg-icon.bus` — yellow `#f0b400`
- `.delay` — red text
- `.platform` — grey background badge
- `.line-summary` — 1.15rem bold
- `.journey-time` — 0.9rem grey
- `.journey-duration` — 1.1rem orange bold

### Bugs Resolved
1. `walking` variable used before definition → moved `const walking` before `lineDisplay`
2. Missing `duration` variable → added back `const duration = formatDuration(legDurationSec(leg))`
3. "0 min" duration → `legDurationSec()` fallback to `new Date(arrival) - new Date(departure)`
4. Address locations showing empty → `locDisplayName()` helper
5. Stopover names missing → `s.stop.name` (hafas-client nesting)
6. 404 on journeys with address → `/stops/reachable-from` resolution
7. Missing latitude/address errors → all three params needed
8. Favorites not working for addresses → stored `lat`/`lon` in favorites

## Features Implemented
- Two location inputs (from/to) with autocomplete dropdown
- Favorites stored in cookies with custom naming via prompt
- Journey search with address-to-stop resolution
- Journey display: line summary, times with delays, per-leg duration, platforms, stopovers, walk timing
- Color-coded transit icons (S=green, R=red, U=blue, bus=yellow)
- "In Xd Xh Xm" countdown to departure
- Total journey duration = last arrival minus first departure
- Infinite scroll pagination via IntersectionObserver
- Mobile-friendly (no title, minimal vertical space)
