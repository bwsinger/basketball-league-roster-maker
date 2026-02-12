# Basketball League Roster Maker — Implementation Plan

## Context

A close friend group needs a simple web app to generate balanced basketball league rosters. The app must support file upload (JSON/CSV/YAML), auto-generate balanced teams, allow manual drag-and-drop adjustments, and export results. Generated rosters should be shareable with all players via a public Google Sheet (uploaded to the creator's Google Drive). No data is stored in the app — everything lives in browser memory and disappears when the tab closes. The developer is new to web dev, so the stack must be beginner-friendly, free to host, and require near-zero maintenance.

---

## Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| **Framework** | Svelte 5 (standalone via Vite, not SvelteKit) | Closest to plain HTML/CSS/JS; reactivity via `$state` — no JSX, no hooks, no virtual DOM. No TypeScript pressure. |
| **Package manager** | Bun | Drop-in npm replacement — dramatically faster installs and script running |
| **Build tool** | Vite (via Bun) | Instant dev server + HMR, Svelte compilation via `@sveltejs/vite-plugin-svelte` |
| **Hosting** | Netlify free tier | Drag-and-drop deploy, auto-deploy from GitHub, 100GB bandwidth/mo free |
| **Drag-and-drop** | SortableJS | Framework-agnostic, multi-list drag via `group` option, integrates with Svelte via a simple action wrapper |
| **File parsing** | `papaparse` (CSV), `js-yaml` (YAML), built-in `JSON.parse` | Minimal, well-maintained, simple APIs |
| **File export** | `file-saver` | Cross-browser `saveAs()` in one line |
| **IDs** | `uuid` | Unique keys for players/teams |
| **CSS** | Tailwind CSS | Utility-first classes in markup — fast styling, great docs |
| **State mgmt** | Svelte 5 runes (`$state`, `$derived`) in a shared `.svelte.js` module | Built-in reactivity, no external library needed |
| **Google sharing** | Google Drive API v3 (via browser `gapi` + GIS scripts from CDN) | Upload CSV → auto-convert to Google Sheet → make public → shareable link. No npm deps. |
| **Backend** | None | All client-side; no persistence by design |
| **Routing** | None | Single page, no navigation |
| **Testing** | Vitest + Svelte Testing Library | Vitest is built for Vite (zero config); Svelte Testing Library for component tests |

**Total dependencies: 7 runtime (`sortablejs`, `papaparse`, `js-yaml`, `file-saver`, `uuid`), ~5 dev (Vite, Svelte plugin, Tailwind, Vitest, Svelte Testing Library).** Managed via Bun. Deliberately minimal.

---

## Project Structure

```
basketball-roster-maker/
├── index.html
├── vite.config.js
├── package.json
├── public/
│   └── favicon.ico
├── src/
│   ├── main.js                         # Entry point, mounts App
│   ├── app.css                         # Tailwind directives (@import "tailwindcss")
│   ├── App.svelte                      # Root layout: config sidebar + roster board
│   │
│   ├── lib/
│   │   ├── state/
│   │   │   └── roster.svelte.js        # Central reactive state ($state): players[], teams[], config{}
│   │   │
│   │   ├── components/
│   │   │   ├── FileUpload.svelte       # Upload drop zone
│   │   │   ├── ConfigPanel.svelte      # Team count, reps/team, height thresholds, sort weights
│   │   │   ├── RosterBoard.svelte      # Grid of team columns
│   │   │   ├── TeamColumn.svelte       # Single team: header, lock toggle, player list (droppable)
│   │   │   ├── PlayerCard.svelte       # Draggable card: name, pronouns, height, skill, status
│   │   │   ├── PlayerCardExpanded.svelte # Full detail modal/panel
│   │   │   ├── ExportPanel.svelte      # Export buttons (JSON, CSV, text, Google Sheets)
│   │   │   ├── LinkingModal.svelte     # Create/manage package deal groups
│   │   │   └── UnassignedPool.svelte   # Players not yet assigned
│   │   │
│   │   ├── logic/
│   │   │   ├── parser.js               # Detect file type → parse → normalize
│   │   │   ├── rosterGenerator.js      # Snake draft balancing algorithm
│   │   │   ├── exporter.js             # Convert roster state → downloadable files
│   │   │   └── validation.js           # Validate uploaded data schema
│   │   │
│   │   ├── services/
│   │   │   └── googleDriveExport.js    # Load gapi/GIS, get OAuth token, upload CSV, set public permissions
│   │   │
│   │   ├── actions/
│   │   │   └── sortable.js             # Svelte action wrapper for SortableJS (~10 lines)
│   │   │
│   │   └── utils/
│   │       ├── constants.js            # Default config values, height thresholds
│   │       └── helpers.js              # Height categorization, etc.
│   │
├── tests/
│   ├── parser.test.js                  # File parsing + normalization
│   ├── rosterGenerator.test.js         # Algorithm correctness (balance, locking, linking)
│   └── helpers.test.js                 # Height categorization, utilities
└── README.md
```

---

## Implementation Phases

### Phase 1: Scaffold + Static UI
- Create a **new standalone repo** (not inside Sepsis_ImmunoScore)
- `bun create vite basketball-roster-maker -- --template svelte`
- Install and configure Tailwind CSS (`bun install -D tailwindcss @tailwindcss/vite`, add plugin to `vite.config.js`, add `@import "tailwindcss"` to `app.css`)
- Build static layout in `App.svelte`: header, config sidebar placeholder, team columns grid
- Build `PlayerCard.svelte`, `TeamColumn.svelte`, `RosterBoard.svelte` with hardcoded fake data
- Grid layout: `grid grid-cols-2 lg:grid-cols-4 gap-4`
- **Test:** Page shows 4 team columns with placeholder player cards, Tailwind styles rendering

### Phase 2: Central State + File Upload
- Install `papaparse`, `js-yaml`, `uuid`
- Build `roster.svelte.js` — central state using `$state`:
  ```js
  export const roster = $state({
    players: [],       // all uploaded players
    teams: [],         // { id, name, locked, players[] }
    config: { numTeams: 4, repsPerTeam: 1, heightThresholds: { shortMax: 66, tallMin: 73 }, weights: { skill: true, height: true, repDistribution: true, statusBalance: true } }
  });
  ```
- Build `FileUpload.svelte` with drag-to-upload zone + file picker (`accept=".json,.csv,.yaml,.yml"`)
- Build `parser.js`: detect extension → parse → normalize to common shape:
  ```js
  { id, name, pronouns, height, skill, representative, status, linkedWith }
  ```
- Build `validation.js` for friendly error messages on bad data
- Build `UnassignedPool.svelte` to display parsed players
- **Test:** Upload JSON/CSV/YAML → players appear in the "Unassigned" pool

### Phase 3: Configuration Panel
- Build `ConfigPanel.svelte`: number of teams, reps/team, height thresholds (short/medium/tall boundaries in inches), sorting factor toggles
- Two-way bind inputs to `roster.config` (Svelte's `bind:value` makes this trivial)
- `helpers.js`: `categorizeHeight(inches, thresholds)` → "short"/"medium"/"tall"
- Dynamically create/remove team columns when `numTeams` changes
- **Test:** Change thresholds → card badges update live; change team count → correct number of columns appear

### Phase 4: Roster Generation Algorithm
- Build `rosterGenerator.js` using **snake draft with multi-factor scoring**:
  1. Separate locked teams' players from the pool
  2. Collapse linked groups into single "units" (averaged stats)
  3. Score units: `(skill × weight) + (heightValue × weight) + small random factor`
  4. Snake draft: round 0 → left-to-right, round 1 → right-to-left, etc.
  5. Secondary preferences: balance reps and returning players across teams
- "Generate Rosters" button + "Re-shuffle" button (random tiebreaker for variation)
- **Test:** Upload → configure → generate → see balanced teams; re-shuffle → different but still balanced

### Phase 5: Drag-and-Drop
- Install `sortablejs`
- Build Svelte action `sortable.js` (~10 lines) wrapping SortableJS initialization:
  ```js
  export function sortable(node, options) {
    const instance = new Sortable(node, options);
    return { destroy() { instance.destroy(); } };
  }
  ```
- Apply `use:sortable` to `UnassignedPool.svelte` and each `TeamColumn.svelte` with shared `group`
- On SortableJS `onEnd` event: update `roster.teams` state to reflect the move
- If player is linked, move all linked players together
- If destination team is locked, reject the drop (SortableJS `onMove` → return `false`)
- **Test:** Drag player from Team 1 → Team 3; linked players follow; locked teams reject drops

### Phase 6: Team Locking
- Add `locked` boolean per team in state, toggle button (lock icon) in `TeamColumn.svelte`
- Locked teams: muted visual style (Tailwind `opacity-60`), disable drag in/out via SortableJS config, excluded from reshuffles
- **Test:** Lock Team 2 → reshuffle → Team 2 unchanged, others rebalanced

### Phase 7: Player Linking (Package Deals)
- Build `LinkingModal.svelte`: multi-select players → "Link" → creates group
- Display existing groups with "Unlink" option
- Visual: matching colored left-border + chain icon on linked `PlayerCard`s
- Algorithm (Phase 4) already handles linked groups; this phase adds the UI
- **Test:** Link A+B → generate → always same team; unlink → can separate

### Phase 8: Expanded Player View
- Build `PlayerCardExpanded.svelte`: click card → shows all attributes in a detail panel/modal
- Show: name, pronouns, exact height, height category, skill (as stars or dots), representative team, player status, linked group members
- **Test:** Click card → expanded view; click away → collapses

### Phase 9: Local Export
- Install `file-saver`
- Build `exporter.js`:
  - `exportJSON(teams)` → `JSON.stringify` → `saveAs(blob, 'rosters.json')`
  - `exportCSV(teams)` → `Papa.unparse` → `saveAs(blob, 'rosters.csv')`
  - `exportText(teams)` → formatted plain text → `saveAs(blob, 'rosters.txt')`
- Add download buttons to `ExportPanel.svelte`
- **Test:** Generate rosters → export CSV → valid file downloads with correct data

### Phase 10: Google Drive Export (Shareable Roster Link)

**Approach:** Upload the roster CSV to the user's Google Drive, auto-convert to a native Google Sheet, make it publicly viewable, return the shareable link.

**One-time Google Cloud Console setup (~15 min):**
1. Create project at console.cloud.google.com → enable **Google Drive API**
2. Configure OAuth consent screen (External, Testing status)
3. Add friends' Gmail addresses as test users
4. Create OAuth 2.0 Client ID (Web app) — authorized origins: `https://your-app.netlify.app` + `http://localhost:5173`
5. Create API key → restrict to your domain + Drive API only

**Implementation:**
- Build `googleDriveExport.js` (~100-150 lines):
  - Dynamically load Google `gapi` + GIS scripts from CDN (no npm deps)
  - `getAccessToken()` — wraps `tokenClient.requestAccessToken()` in a Promise
  - `uploadAndShare(filename, csvContent)`:
    1. Construct multipart/related body with metadata (`mimeType: 'application/vnd.google-apps.spreadsheet'` for auto-conversion)
    2. `fetch(upload endpoint)` with OAuth token → creates Google Sheet
    3. `fetch(permissions endpoint)` → set `{type: "anyone", role: "reader"}`
    4. Return `webViewLink` (shareable URL)
- Only 1 OAuth scope: `drive.file` (non-sensitive — friendliest consent screen)
- Only 2 API calls per export
- Add "Share to Google Sheets" button in `ExportPanel.svelte` → OAuth popup → upload → show link + "Copy Link" button

**Gotchas:**
- Users see "unverified app" warning (click Advanced → proceed). Walk friends through once.
- OAuth popup must trigger from a button click or browsers block it
- API key is safe to embed — designed to be public; restrict to your domain

**Test:** Click "Share to Google Sheets" → sign in → public Google Sheet created → link opens a real spreadsheet with roster data

### Phase 11: Automated Tests
- Install `vitest`, `jsdom`, `@testing-library/svelte`
- Add `"test": "vitest run"` to `package.json` scripts
- **`parser.test.js`** — parse JSON/CSV/YAML strings → normalized player objects; validate rejects bad data
- **`rosterGenerator.test.js`** — generated teams balanced (skill totals within threshold); locked teams preserved; linked players on same team
- **`helpers.test.js`** — `categorizeHeight` at boundary values
- **Test:** `bun test` passes all tests

### Phase 12: Polish + Deploy
- Data privacy: verify zero `localStorage`/`sessionStorage`/cookie usage; add `beforeunload` listener as defense-in-depth
- Error handling for invalid files, impossible configs (e.g., more reps/team than total reps)
- Responsive Tailwind classes for laptop screens (columns wrap on narrow windows)
- Deploy to Netlify (see below)

---

## Deployment

**Google Cloud Console (one-time, before deploy):**
1. Create project → enable Drive API → configure OAuth consent (Testing mode) → add friends as test users
2. Create OAuth Client ID → set authorized origins to your Netlify URL + localhost
3. Create API key → restrict to your domain + Drive API
4. Put Client ID and API key in `src/lib/services/googleDriveExport.js` as constants

**Netlify:**
1. `bun run build` → creates `dist/` folder
2. `bun run preview` → verify locally at `localhost:4173`
3. Push to GitHub
4. Netlify: "Add new site" → "Import from GitHub" → select repo → build command `bun run build`, publish dir `dist` → Deploy (Netlify has native Bun support)
5. Rename URL to something like `basketball-roster-maker.netlify.app`
6. Every future push to `main` auto-deploys

**Zero-Git alternative:** Run `bun run build`, drag `dist/` folder onto https://app.netlify.com/drop — live in 10 seconds.

---

## Verification

- Run `bun test` — all automated tests pass
- Upload sample files in all 3 formats (JSON, CSV, YAML) and confirm parsing
- Configure different team counts and height thresholds, verify UI updates
- Generate rosters, verify teams are reasonably balanced by skill/height
- Lock a team, reshuffle, confirm locked team is unchanged
- Link two players, generate, confirm they're always together
- Drag a player between teams, confirm state updates
- Drag onto locked team, confirm rejection
- Export in all 3 formats, confirm file content matches roster state
- Click "Share to Google Sheets" → sign in → public Google Sheet created with roster data → link is copyable
- Close tab, reopen — confirm no data persists
- Deploy to Netlify, repeat key tests on live URL
