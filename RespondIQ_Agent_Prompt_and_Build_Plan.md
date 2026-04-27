# RespondIQ — AI Agent Build Prompt & Plan
### TechFusion 2.0 Ideathon | Resilient Response: Multi-Agency Disaster Coordination

---

## HOW TO USE THIS FILE

1. Open your AI coding agent (Claude Code, Cursor, Bolt, Windsurf, or Lovable)
2. Create a new empty project/directory called `disaster-coord`
3. Paste the AGENT PROMPT (Section 2) as your first message
4. Follow the POST-GENERATION CHECKLIST (Section 3) after files are created
5. Use the DEMO SCRIPT (Section 4) to rehearse before judging

---

---

# SECTION 1 — BUILD PLAN

## Timeline Overview (6 Hours)

```
Hour 1  [0:00–1:00]  Scaffold + Map + Service Worker
Hour 2  [1:00–2:00]  Incident feed + Edge AI triage
Hour 3  [2:00–3:00]  Resource allocation + responder assignment
Hour 4  [3:00–4:00]  Citizen request form + SMS/LoRa fallback UI
Hour 5  [4:00–5:00]  Polish + disaster simulation auto-play
Hour 6  [5:00–6:00]  Rehearse demo — NO new features
```

---

## File Structure the Agent Will Generate

```
disaster-coord/
├── index.html        ←  Full app: UI + all inline JS logic
├── sw.js             ←  Service Worker (cache-first offline shell)
├── db.js             ←  IndexedDB wrapper (incidents, responders, outbox)
├── sim.js            ←  Disaster scenario simulator (auto-play timeline)
└── manifest.json     ←  PWA manifest (installable on Android)
```

> No node_modules. No build step. No server required.
> Open index.html directly in any browser.

---

## Tech Stack Decisions

| Layer            | Choice                        | Reason                                          |
|------------------|-------------------------------|-------------------------------------------------|
| Frontend         | Vanilla JS + single HTML file | Zero build step, works on any device            |
| Offline engine   | Service Worker + IndexedDB    | Native browser, no library needed               |
| Map              | Leaflet.js via CDN            | Free, lightweight, offline-cacheable            |
| AI triage        | Rule-based JS classifier      | Runs on 2GB RAM phones, zero latency            |
| Comms fallback   | Simulated SMS/LoRa feed       | Demo-able without physical hardware             |
| Backend          | None                          | Eliminates all setup/failure risk during demo   |
| Fonts            | system-ui, sans-serif         | Instant load, zero network dependency           |

---

## Evaluation Criteria Mapping

| Criterion        | How the build addresses it                                                |
|------------------|---------------------------------------------------------------------------|
| Innovation       | Edge AI triage in-browser + LoRa/BLE mesh architecture concept            |
| Feasibility      | Single HTML file, runs on ₹5,000 Android phone, zero install              |
| Resilience       | Three fallback layers: internet → SMS → LoRa. Offline-first by design     |
| Execution        | Working demo of exact flow: citizen → assign → status update              |
| Real-world impact| Works with no app store, no towers, ₹1,500/node hardware deployment       |

---

---

# SECTION 2 — AGENT PROMPT

> Copy everything from the START PROMPT marker to the END PROMPT marker
> and paste it as a single message into your AI coding agent.

---

### ============ START PROMPT ============

Build a single-file offline-first disaster coordination PWA demo called "RespondIQ"
for an Indian ideathon. Everything must work with zero internet after first load.

---

## STACK CONSTRAINTS

- Single `index.html` file with all CSS and JS inline — no build step, no npm,
  no frameworks
- `sw.js` for Service Worker (separate file, referenced from index.html)
- `db.js` for IndexedDB wrapper (separate file)
- `sim.js` for disaster simulator (separate file)
- Vanilla JS only — no React, no Vue, no jQuery
- Leaflet.js loaded from CDN (already cached by SW) for the map
- Total JS payload under 200KB excluding Leaflet
- System font stack only (font-family: system-ui, sans-serif) — no Google Fonts
- Mobile-first CSS, works on 360px Android screens up to 1920px desktop
- Disable all Leaflet animations (zoomAnimation: false, fadeAnimation: false,
  markerZoomAnimation: false) for low-end device performance

---

## FILE STRUCTURE TO GENERATE

```
disaster-coord/
├── index.html      ← full app UI + logic
├── sw.js           ← service worker
├── db.js           ← IndexedDB wrapper
├── manifest.json   ← PWA manifest
└── sim.js          ← disaster scenario simulator
```

---

## APP SHELL AND LAYOUT

- Dark theme (#0f0f1a background, #e2e8f0 text)
- Three-panel layout on desktop:
    - Left sidebar: agency/resource panel (280px fixed width)
    - Center: Leaflet map (flex-grow, fills remaining space)
    - Right sidebar: incident queue (320px fixed width)
- On mobile (under 768px):
    - Single column layout
    - Map takes 45vh
    - Panels stack below as tabs: Map / Incidents / Resources
- Top bar contains:
    - App name "RespondIQ" in bold
    - Connectivity status indicator (green dot = online, red dot = offline)
    - Live clock (updates every second)
    - Active incident count badge
- Stats bar below top bar — four cards:
    - Active Incidents
    - Responders Deployed
    - Resolved
    - Avg Response Time

---

## MAP REQUIREMENTS

- Leaflet.js map centred on Bengaluru, India (12.9716, 77.5946), zoom level 12
- OpenStreetMap tiles
- Five marker types with distinct colours:
    - Red pulsing CSS circle marker: CRITICAL incidents (severity 8-10)
    - Orange circle marker: HIGH incidents (severity 5-7)
    - Yellow circle marker: MEDIUM incidents (severity 2-4)
    - Green circle marker: RESOLVED incidents
    - Blue marker: Responder/resource current positions
- Clicking any incident marker opens a Leaflet popup containing:
    - Incident ID, type badge, severity score
    - Original message text
    - Citizen name and timestamp received
    - Source badge (LoRa / SMS / Manual)
    - Assigned responder name (or "Unassigned")
    - Current status
    - "Assign Nearest Responder" button (disabled if already assigned)
- When an assignment is made, draw a dashed polyline from the responder
  marker to the incident marker
- Polyline is removed when incident is resolved

---

## INDEXEDDB SCHEMA (db.js)

Implement three object stores and expose the following API:

Object stores:

```
incidents — keyPath: 'id'
  Fields: id (string), type (string), severity (int 1-10),
          message (string), citizenName (string), phone (string),
          lat (float), lng (float),
          status ('open' | 'assigned' | 'resolved'),
          assignedResponder (string|null), timestamp (int),
          source ('lora' | 'sms' | 'manual')

responders — keyPath: 'id'
  Fields: id (string), name (string),
          agency ('fire' | 'police' | 'medical' | 'ndrf'),
          lat (float), lng (float),
          status ('available' | 'busy' | 'offline'),
          currentIncident (string|null)

outbox — keyPath: 'id', autoIncrement: true
  Fields: type (string), payload (object), timestamp (int)
```

Exported functions from db.js:

```javascript
initDB()                          // opens/upgrades the DB, returns promise
saveIncident(incident)            // put into incidents store
getIncidents()                    // getAll from incidents store
updateIncident(id, changes)       // get, merge changes, put back
saveResponder(responder)          // put into responders store
getResponders()                   // getAll from responders store
updateResponder(id, changes)      // get, merge changes, put back
addToOutbox(item)                 // add to outbox store
flushOutbox(syncFn)               // iterate outbox, call syncFn(item) for each,
                                  // delete on success, return count flushed
```

Do not use any external IndexedDB library — implement with raw IDBOpenRequest API
wrapped in Promises.

---

## EDGE AI CLASSIFIER (inline in index.html)

Implement `classifyMessage(text)` returning `{ type, severity, keywords, agency }`.

Classification types: FLOOD, FIRE, MEDICAL, RESCUE, STRUCTURAL, RELIEF, UNKNOWN

Severity is an integer 1-10.

Keyword rules must include common Indian disaster phrases and transliterated
Hindi/Kannada words including at minimum:
- Flood: flood, water, paani, baarish, submerged, drowning, knee deep, waist deep
- Fire: fire, aag, burning, smoke, blast, explosion
- Medical: injured, hurt, bleeding, unconscious, heart, breathe, hospital, doctor
- Rescue: trapped, stuck, collapse, debris, building, bachao, help, madad
- Structural: crack, collapse, wall, roof, building, fallen
- Relief: food, hungry, shelter, water (non-flood context), medicine, blanket

Agency suggestion logic:
- FLOOD/RESCUE → ndrf
- FIRE → fire
- MEDICAL → medical
- STRUCTURAL → ndrf
- RELIEF → police
- UNKNOWN → police

Display a live "AI Triage" panel in the right sidebar that shows each
incoming message being classified:
- Raw message text
- Detected keywords (highlighted in amber)
- Classified type badge (colour-coded)
- Severity score bar (1-10)
- Suggested agency badge
- Each new classification entry flashes with a 300ms highlight animation
- Panel shows last 10 classifications, scrollable

---

## RESOURCE ALLOCATION LOGIC (inline in index.html)

Implement `assignNearestResponder(incidentId)` as follows:

1. Load all responders from IndexedDB
2. Filter to status === 'available'
3. Filter by matching agency type:
   - MEDICAL incidents → agency 'medical' first, fallback any available
   - FIRE incidents → agency 'fire' first, fallback any available
   - FLOOD, RESCUE, STRUCTURAL → agency 'ndrf' first, fallback police
   - RELIEF, UNKNOWN → agency 'police' first, fallback any available
4. Calculate Haversine distance from each candidate to incident coordinates
5. Select the nearest available responder
6. Update incident: status = 'assigned', assignedResponder = responder.id
7. Update responder: status = 'busy', currentIncident = incident.id
8. Persist both changes to IndexedDB
9. Update map: change incident marker colour, draw dashed polyline,
   show responder marker movement
10. If navigator.onLine is false: add assignment to outbox instead
11. Log event to the Event Log panel
12. Return { responder, distance, estimatedArrival (distance/40km/h in minutes) }

Also implement `resolveIncident(incidentId)`:
1. Update incident status to 'resolved'
2. Free the assigned responder (status = 'available', currentIncident = null)
3. Change map marker to green
4. Remove polyline
5. Update stats bar
6. Log to Event Log

---

## DISASTER SIMULATION ENGINE (sim.js)

Export `startSimulation(app)` and `stopSimulation()`.

The `app` parameter is an object with methods:
- `app.injectIncident(incident)` — adds incident to DB and map
- `app.injectMessage(source, text, citizenName)` — triggers AI triage
- `app.triggerOffline()` — simulates network failure
- `app.triggerOnline()` — simulates network restore
- `app.autoAssign(incidentId)` — triggers auto assignment

Simulation timeline (all timeouts stored in an array for cleanup):

```
T+0s   Inject FLOOD incident: lat 12.9352, lng 77.6245 (Koramangala)
       citizen: "Meera Sharma", message: "Water level rising fast,
       knee deep in ground floor, 4 people stuck please help"

T+2s   Inject FIRE incident: lat 12.9698, lng 77.7499 (Whitefield)
       citizen: "Suresh Kumar", message: "Aag lag gayi factory mein,
       smoke everywhere, workers trapped inside"

T+5s   Inject SMS message from citizen "Ravi Nair":
       "My father fell unconscious, need ambulance urgently,
       near Indiranagar 100ft road"
       Show this in AI triage panel with SMS badge

T+8s   Inject RESCUE incident: lat 12.9121, lng 77.6446 (HSR Layout)
       citizen: "Kavya Reddy", message: "Building wall collapsed,
       3 people under debris, bachao please"

T+10s  Inject MEDICAL incident at T+5s citizen's location
       lat 12.9784, lng 77.6408

T+12s  Inject LoRa packet message (show as hex decode):
       Raw: "464c4f4f44 313220 39372e3535"
       Decoded: "FLOOD L12 97.55" — inject as FLOOD alert, BTM Layout
       lat 12.9166, lng 77.6101

T+15s  Inject RELIEF incident: lat 12.9308, lng 77.5833 (Jayanagar)
       citizen: "Anjali Singh", message: "No food or water for
       2 days, 8 people in shelter, need madad"

T+20s  Auto-assign the two oldest unassigned incidents

T+25s  Auto-assign two more incidents

T+30s  Resolve the first assigned incident (FLOOD in Koramangala)
       Show green marker, log "Incident resolved by Arjun Singh"

T+45s  Trigger network failure (app.triggerOffline())
       Show red banner, inject one more citizen request to demonstrate
       offline queueing

T+60s  Trigger network restore (app.triggerOnline())
       Flush outbox, show toast "3 queued updates synced"

T+75s  Auto-assign remaining open incidents
```

All interval/timeout IDs must be stored in an array.
`stopSimulation()` must call clearTimeout/clearInterval on all of them.

---

## CITIZEN REQUEST PANEL

Collapsible form panel at bottom of right sidebar.

Fields:
- Name (text input, required)
- Phone (tel input, pattern: Indian mobile)
- Location (text input — address or "Use my GPS" button that fills
  lat/lng from navigator.geolocation)
- Need type (select: Medical Emergency / Fire / Flood / Trapped / Food+Water / Other)
- Description (textarea, max 280 chars, live char counter)
- Submit button

On submit:
1. Validate required fields
2. Run classifyMessage on the description
3. Create incident object with generated ID (INC-timestamp)
4. Save to IndexedDB
5. If offline: add to outbox, show "Request queued — will sync when connected"
6. If online: show "Request received — ID: INC-XXXX"
7. Add pin to map immediately regardless of connectivity
8. Inject into AI triage panel
9. Clear form

---

## SERVICE WORKER (sw.js)

```javascript
const CACHE_NAME = 'respondiq-v1';

// Cache these on install:
const APP_SHELL = [
  '/',
  '/index.html',
  '/sw.js',
  '/db.js',
  '/sim.js',
  '/manifest.json',
  'https://unpkg.com/leaflet@1.9.4/dist/leaflet.css',
  'https://unpkg.com/leaflet@1.9.4/dist/leaflet.js'
];

// Install: cache all app shell files
// Activate: delete all caches not matching CACHE_NAME
// Fetch strategy:
//   - App shell files (index.html, js, css): Cache-first
//   - Map tiles (tile.openstreetmap.org): Cache-first with network fallback
//   - Everything else: Network-first with cache fallback
// Message handler: respond to { type: 'SKIP_WAITING' } to activate immediately
```

---

## OFFLINE / ONLINE HANDLING

In index.html:

- On `window.addEventListener('online')`:
  1. Update connectivity indicator to green
  2. Hide offline banner
  3. Call `db.flushOutbox(syncToServer)` where syncToServer is a mock
     function that logs "synced: " + JSON.stringify(item)
  4. Show toast notification: "Connected — X updates synced" for 3 seconds
  5. Re-enable any disabled buttons

- On `window.addEventListener('offline')`:
  1. Update connectivity indicator to red dot
  2. Show persistent red banner at top: "Offline mode — all actions
     queued locally"
  3. Disable fetch calls (route to IndexedDB only)

- Check `navigator.onLine` before every fetch call and every form submit

- Add a manual "Toggle Offline Mode" button in the top bar for demo purposes
  that overrides navigator.onLine with a local `isOnline` boolean — this
  allows the demo to simulate offline even when internet is available

---

## SMS / LORA SIMULATION PANEL

In the Event Log panel, show a dedicated section "Communication Feed".

Each entry must show:
- Source badge: LoRa (purple) / SMS (green) / Manual (blue) / System (gray)
- Timestamp (HH:MM:SS)
- For SMS: masked sender number (+91 98XXX XXXXX), message text, triage result
- For LoRa: raw hex payload, decoded message, triage result
- For System: event description

Two demo buttons visible in this panel:
1. "Send Test SMS" — picks a random message from a pre-written list of 8
   Indian disaster scenario messages and injects it through the triage pipeline
2. "Simulate LoRa Packet" — injects a pre-defined hex payload, shows decode
   animation (hex → ASCII → structured data → triage result)

Pre-written SMS test messages (pick randomly):
```
"Paani aa gaya ghar mein, first floor tak water, 6 log phanse hain"
"Aag lagi hai building mein, 3rd floor, smoke bahut hai, help karo"
"My mother is unconscious not breathing, need ambulance Koramangala"
"Wall girne wali hai, cracks aa gaye hain, bachao please"
"No food for children since yesterday, need help Jayanagar shelter"
"Flood water entered shop, goods damaged, need rescue boats"
"Man fell from construction site, leg broken, bleeding, need help"
"Gas leak ho raha hai, smell aa rahi hai, kya karein"
```

---

## PWA MANIFEST (manifest.json)

```json
{
  "name": "RespondIQ — Disaster Coordination",
  "short_name": "RespondIQ",
  "description": "Offline-first multi-agency disaster coordination system",
  "start_url": "/index.html",
  "display": "standalone",
  "orientation": "any",
  "background_color": "#0f0f1a",
  "theme_color": "#3b82f6",
  "icons": [
    {
      "src": "data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><rect width='100' height='100' rx='20' fill='%233b82f6'/><text y='.9em' font-size='70' x='15'>🚨</text></svg>",
      "sizes": "192x192",
      "type": "image/svg+xml"
    }
  ]
}
```

---

## SEED DATA (pre-populate on initDB() first run)

8 responders to seed into IndexedDB on first load:

```
R1: Arjun Singh    | medical | 12.9352, 77.6245 | available
R2: Priya Nair     | fire    | 12.9698, 77.7499 | available
R3: Karan Mehta    | ndrf    | 12.9121, 77.6446 | available
R4: Sunita Rao     | police  | 12.9784, 77.6408 | available
R5: Vikram Das     | medical | 12.9166, 77.6101 | available
R6: Lakshmi Iyer   | fire    | 12.9591, 77.7007 | available
R7: Raju Patil     | ndrf    | 12.9308, 77.5833 | available
R8: Deepa Menon    | police  | 12.9757, 77.6011 | available
```

Check if responders store is empty on startup. If empty, seed all 8.
Show responders on map as blue markers with name tooltips.

---

## PERFORMANCE REQUIREMENTS

- IndexedDB writes debounced at 300ms — never block UI thread
- Map marker updates: never redraw all markers — only update changed ones,
  store marker references in a Map() keyed by incident/responder ID
- Event log capped at 100 entries — splice oldest when over limit
- No document.write anywhere
- No innerHTML on any user-supplied input — use textContent or createElement
- All setInterval and setTimeout IDs stored in window._simTimers array
- stopSimulation() must clearTimeout/clearInterval on all entries in that array
- Leaflet map: use L.circleMarker for incidents (lighter than L.marker),
  L.marker only for responders
- CSS: use will-change: transform only on the pulsing critical marker animation

---

## WHAT JUDGES MUST BE ABLE TO DO IN THE DEMO

1. See the Bengaluru map load with 8 responder positions marked in blue
2. Click "Start Simulation" and watch incidents appear sequentially on the map
3. Watch the AI Triage panel classify each incoming message in real time,
   showing keywords highlighted and severity scored
4. Click any incident marker and see the popup with full details
5. Click "Assign Nearest Responder" and watch the polyline draw from
   responder to incident
6. Click "Toggle Offline" button — red banner appears
7. Submit a citizen request form while offline — see confirmation with INC ID
8. Click "Restore Connection" — see outbox flush toast with count
9. See the full Event Log with LoRa / SMS / System colour-coded badges
10. Optionally: click "Send Test SMS" and "Simulate LoRa Packet" buttons

---

## FINAL INSTRUCTION TO AGENT

Generate all five files completely.
No placeholders. No TODO comments. No "add your logic here".
Every function must be fully implemented and working.

The app must:
- Work by opening index.html directly in Chrome/Firefox/Edge with no server
- Work when served from `python3 -m http.server 8080`
- Work when served from a Raspberry Pi on local LAN
- Pass a manual offline test: load in browser, go to DevTools → Network →
  Offline, reload page — full app must load from Service Worker cache

### ============ END PROMPT ============

---

---

# SECTION 3 — POST-GENERATION CHECKLIST

Complete these steps immediately after the agent generates the files.

## Step 1 — Fix Service Worker cache URLs (CRITICAL)

Open `sw.js` and find the `APP_SHELL` array.
Verify these exact Leaflet CDN URLs are present:

```
https://unpkg.com/leaflet@1.9.4/dist/leaflet.css
https://unpkg.com/leaflet@1.9.4/dist/leaflet.js
```

If the agent used different CDN URLs in index.html, update APP_SHELL
to match exactly. Mismatched URLs = map fails offline.

## Step 2 — Test offline mode

1. Open index.html in Chrome
2. Open DevTools (F12) → Application → Service Workers
3. Confirm SW is registered and status is "activated"
4. DevTools → Network tab → check "Offline"
5. Reload the page
6. Map and full UI must load — if it fails, the SW cache list is wrong

## Step 3 — Test on mobile

1. Serve with: `python3 -m http.server 8080`
2. Find your laptop's LAN IP: `ifconfig` or `ipconfig`
3. On your Android phone (same Wi-Fi): open `http://192.168.X.X:8080`
4. Verify layout is usable on mobile screen
5. Check that map renders and markers are tappable

## Step 4 — Run full simulation

1. Open index.html
2. Click "Start Simulation"
3. Confirm all 10+ steps of the timeline execute correctly
4. Confirm the offline/online toggle works and outbox flushes
5. Time the simulation — it should run ~90 seconds total

## Step 5 — Edge cases to verify

- Submit citizen form while offline → incident appears on map + queued
- Click "Assign" on an already-assigned incident → button should be disabled
- Resolve an incident → polyline disappears, marker turns green, stats update
- Event log → scroll works, entries don't bleed outside panel

---

---

# SECTION 4 — DEMO SCRIPT (3 Minutes for Judges)

Rehearse this exact sequence in Hour 6. Do not improvise.

---

**[0:00] Open the app**
> "This is RespondIQ — a disaster coordination system designed to work
> with zero internet. Notice the connectivity indicator — green, online.
> Eight responders are already deployed across Bengaluru."

**[0:20] Kill internet visibly**
> Click "Toggle Offline" on the top bar.
> "Internet is now off. Red banner appears. The app is running entirely
> from the browser's offline cache — Service Worker technology."

**[0:35] Start simulation**
> Click "Start Simulation".
> "A flood in Koramangala. A factory fire in Whitefield. Watch the AI
> triage panel — it's classifying every incoming message locally,
> no cloud API, no internet. Keyword detection, severity scoring,
> agency routing — all running on this device."

**[1:00] Show AI triage**
> Point to triage panel as SMS arrives.
> "This SMS came in — paani, phanse — the classifier identifies FLOOD,
> severity 8, routes to NDRF. The coordinator sees a structured card,
> not raw text."

**[1:20] Manual assignment**
> Click on the flood incident marker on the map.
> "Here's the incident detail. I'll assign the nearest available
> responder." Click "Assign Nearest Responder".
> "Haversine distance calculated locally. Arjun Singh, 1.2km away.
> Polyline drawn. ETA: 2 minutes. All computed on-device."

**[1:45] Offline citizen request**
> Scroll to citizen form. Fill name "Demo Citizen", select Medical,
> type "Need help urgently" in description.
> "Submitting while offline. Incident created locally, queued in
> IndexedDB outbox. The citizen gets a confirmation immediately."

**[2:05] Restore connection**
> Click "Restore Connection".
> "Connection restored. Outbox flushes automatically — 3 queued
> updates synced. No data was lost during the outage."

**[2:20] LoRa simulation**
> Click "Simulate LoRa Packet".
> "This is what a LoRa mesh message looks like — raw hex from a
> field node. Decoded to a flood alert, triaged, pinned on the map.
> No cell tower. No internet. Just radio."

**[2:40] Close**
> "Three fallback layers: internet, SMS, LoRa mesh. The app is a
> PWA — it installs on any Android phone, works offline permanently,
> and runs on hardware costing ₹1,500 per node. Deployable today,
> anywhere in India."

---

---

# SECTION 5 — AGENT-SPECIFIC NOTES

## Claude Code
Paste prompt into terminal after running `claude` in empty directory.
Works as-is. Will generate all 5 files in sequence.

## Cursor / Windsurf
Open a new project folder. Paste into the AI chat panel (not a file).
Let it complete before asking for any edits.

## Bolt.new / Lovable
Prepend this line to the prompt before pasting:
> "Do not use React, Vue, or any JS framework. Vanilla JS only."
These agents default to React — the prepended line overrides that.

## v0 (Vercel)
v0 will try to use Next.js. Add to the start of the prompt:
> "Generate plain HTML/CSS/JS files only. No Next.js. No npm.
> No package.json. Static files that open directly in a browser."

---

*RespondIQ | TechFusion 2.0 | Built for India's most resilient response*
