# Basketball League Roster Maker — Implementation Plan

## Context

A close friend group needs a simple web app to generate balanced basketball league rosters. The app must support file upload (JSON/CSV/YAML), auto-generate balanced teams, allow manual drag-and-drop adjustments, and export results. Generated rosters should be shareable with all players via a public Google Sheet (uploaded to the creator's Google Drive). No data is stored in the app — everything lives in browser memory and disappears when the tab closes. The stack should be accessible, free to host, and require near-zero maintenance.

---

## Mission Success Criteria

### Teaching Success
- Developer can explain and implement: file parsing, reactive UI state, a pure roster-generation algorithm, drag-and-drop interactions, API integration, automated tests, and deployment.
- Every phase ships at least one automated test and one short retrospective note ("what was learned", "what was confusing", "what to improve").
- Developer can onboard a second contributor using project docs in under 30 minutes.

### Product Success
- A commissioner can go from raw player file to shareable roster output in under 5 minutes.
- Generated teams are "fair enough" by measurable thresholds (defined in the algorithm section).
- Pilot run with friend group completes without critical failures (broken imports, data loss during session, export failure).
- App remains no-maintenance and free-tier compatible.

---

## MVP Release Plan

### Milestone A: Local MVP (first real usage)
- Upload (JSON/CSV/YAML), validate, normalize.
- Configure teams and generate balanced rosters.
- Manual team edits (drag-and-drop and non-drag fallback controls).
- Local export (CSV/JSON/text).

### Milestone B: Friend Group Pilot
- Run at least one real draft session with actual friend-group data.
- Capture issues and confusing UX points.
- Harden validation and algorithm thresholds from real-world feedback.

### Milestone C: Shareable Cloud Export
- Add Google Sheets export with OAuth.
- Start in OAuth Testing mode for internal validation.
- Promote to Production mode when pilot is stable and consent/privacy text is finalized.

---

## Risk Register

| Risk | Impact | Mitigation | Fallback |
|------|--------|------------|----------|
| OAuth flow blocks users | Sharing feature unusable | Keep local export path first-class; provide setup checklist and troubleshooting | Skip cloud export and share CSV directly |
| Public sheet leaks sensitive fields | Privacy breach | Add explicit "public share" consent checkbox and warning text before publish | Create private sheet only; user manually shares |
| Poor input data quality | Bad rosters or crashes | Strict validation + row-level error reporting + sample templates | Allow partial import with invalid rows excluded |
| DnD-only interaction fails on some devices | Editing blocked | Add non-drag controls (assign/move buttons) and keyboard support | Disable DnD and use button-driven assignment |
| Algorithm perceived as unfair | Low trust, manual rework | Publish fairness metrics and seeded reshuffle option | Manual editing mode after generation |
| Bun ecosystem/tooling hiccups | Developer blocked | Document npm equivalents for all commands | Switch to npm without architecture changes |

---

## Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| **Framework** | Svelte 5 (standalone via Vite, not SvelteKit) | Closest to plain HTML/CSS/JS; reactivity via `$state` — no JSX, no hooks, no virtual DOM. No TypeScript pressure. |
| **Package manager** | Bun (with npm fallback) | Bun is fast, and every command has npm fallback docs for compatibility |
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

**Tooling fallback:** Keep scripts runnable with both Bun and npm (`bun run dev` / `npm run dev`, `bun test` / `npm test`).

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

Each phase includes a Definition of Done (DoD): feature works, at least one targeted automated test exists, manual QA checklist item passes, and docs are updated.

### Phase 1: Scaffold + Static UI
- Create a **new standalone repo** (not inside Sepsis_ImmunoScore)
- `bun create vite basketball-roster-maker -- --template svelte`
- Install and configure Tailwind CSS (`bun install -D tailwindcss @tailwindcss/vite`, add plugin to `vite.config.js`, add `@import "tailwindcss"` to `app.css`)
- Build static layout in `App.svelte`: header, config sidebar placeholder, team columns grid
- Build `PlayerCard.svelte`, `TeamColumn.svelte`, `RosterBoard.svelte` with hardcoded fake data
- Grid layout: `grid grid-cols-2 lg:grid-cols-4 gap-4`
- **Test:** Page shows 4 team columns with placeholder player cards, Tailwind styles rendering
- **DoD:** `dev` build runs, layout matches wireframe on desktop/mobile, first smoke test added.

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
- Add parser/validation tests now (not later): valid parse cases + required-field failures
- **Test:** Upload JSON/CSV/YAML → players appear in the "Unassigned" pool
- **DoD:** Import flow handles success and validation errors with clear messaging.

### Phase 3: Configuration Panel
- Build `ConfigPanel.svelte`: number of teams, reps/team, height thresholds (short/medium/tall boundaries in inches), sorting factor toggles
- Two-way bind inputs to `roster.config` (Svelte's `bind:value` makes this trivial)
- `helpers.js`: `categorizeHeight(inches, thresholds)` → "short"/"medium"/"tall"
- Dynamically create/remove team columns when `numTeams` changes
- Add helper tests for threshold boundaries
- **Test:** Change thresholds → card badges update live; change team count → correct number of columns appear
- **DoD:** Config updates are reactive and validated (no impossible values).

### Phase 4: Roster Generation Algorithm
- Build `rosterGenerator.js` using **snake draft with multi-factor scoring**:
  1. Separate locked teams' players from the pool
  2. Collapse linked groups into single "units" (averaged stats)
  3. Score units: `(skill × weight) + (heightValue × weight) + small random factor`
  4. Snake draft: round 0 → left-to-right, round 1 → right-to-left, etc.
  5. Secondary preferences: balance reps and returning players across teams
- "Generate Rosters" button + "Re-shuffle" button (random tiebreaker for variation)
- Add seeded randomness option (for reproducible results in tests and bug reports)
- Define acceptance thresholds:
  - Total skill delta between strongest and weakest team <= configured threshold
  - Representative-count variance <= 1
  - Height bucket variance <= 1 per bucket where feasible
- Add generator tests now: balance constraints + seeded reproducibility
- **Test:** Upload → configure → generate → see balanced teams; re-shuffle → different but still balanced
- **DoD:** Algorithm outputs meet thresholds and tests cover key edge cases.

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
- Add non-drag fallback actions (Assign to Team, Move Left/Right) for accessibility and touch/device reliability
- **Test:** Drag player from Team 1 → Team 3; linked players follow; locked teams reject drops
- **DoD:** Team reassignment works via drag and via non-drag controls.

### Phase 6: Team Locking
- Add `locked` boolean per team in state, toggle button (lock icon) in `TeamColumn.svelte`
- Locked teams: muted visual style (Tailwind `opacity-60`), disable drag in/out via SortableJS config, excluded from reshuffles
- **Test:** Lock Team 2 → reshuffle → Team 2 unchanged, others rebalanced
- **DoD:** Lock behavior respected by generation and manual moves.

### Phase 7: Player Linking (Package Deals)
- Build `LinkingModal.svelte`: multi-select players → "Link" → creates group
- Display existing groups with "Unlink" option
- Visual: matching colored left-border + chain icon on linked `PlayerCard`s
- Algorithm (Phase 4) already handles linked groups; this phase adds the UI
- **Test:** Link A+B → generate → always same team; unlink → can separate
- **DoD:** Link lifecycle (create/view/remove) is stable and covered by tests.

### Phase 8: Expanded Player View
- Build `PlayerCardExpanded.svelte`: click card → shows all attributes in a detail panel/modal
- Show: name, pronouns, exact height, height category, skill (as stars or dots), representative team, player status, linked group members
- **Test:** Click card → expanded view; click away → collapses
- **DoD:** Detail view is keyboard accessible and mobile-friendly.

### Phase 9: Local Export
- Install `file-saver`
- Build `exporter.js`:
  - `exportJSON(teams)` → `JSON.stringify` → `saveAs(blob, 'rosters.json')`
  - `exportCSV(teams)` → `Papa.unparse` → `saveAs(blob, 'rosters.csv')`
  - `exportText(teams)` → formatted plain text → `saveAs(blob, 'rosters.txt')`
- Add download buttons to `ExportPanel.svelte`
- **Test:** Generate rosters → export CSV → valid file downloads with correct data
- **DoD:** Export outputs are schema-consistent and verified with test fixtures.

### Phase 10: Google Drive Export (Shareable Roster Link)

**Approach:** Upload the roster CSV to the user's Google Drive, auto-convert to a native Google Sheet, make it publicly viewable, return the shareable link.

**One-time Google Cloud Console setup (~15 min):**
1. Create project at console.cloud.google.com → enable **Google Drive API**
2. Configure OAuth consent screen (External, Testing status for pilot)
3. Add pilot users' Gmail addresses as test users
4. Create OAuth 2.0 Client ID (Web app) — authorized origins: `https://your-app.netlify.app` + `http://localhost:5173`
5. Create API key → restrict to your domain + Drive API only
6. After pilot stabilizes, move OAuth app to **Production** and verify branding/privacy text

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
- Add pre-share confirmation:
  - Explicit checkbox: "I understand this creates a public, viewable link"
  - Optional redaction toggle to omit sensitive columns (for example pronouns/status) from shared export

**Gotchas:**
- Users see "unverified app" warning (click Advanced → proceed). Walk friends through once.
- OAuth popup must trigger from a button click or browsers block it
- API key is safe to embed — designed to be public; restrict to your domain

**Test:** Click "Share to Google Sheets" → sign in → public Google Sheet created → link opens a real spreadsheet with roster data

### Phase 11: Expanded Automated Tests + Regression Suite
- Install `vitest`, `jsdom`, `@testing-library/svelte`
- Add `"test": "vitest run"` to `package.json` scripts
- **`parser.test.js`** — parse JSON/CSV/YAML strings → normalized player objects; validate rejects bad data
- **`rosterGenerator.test.js`** — generated teams balanced (skill totals within threshold); locked teams preserved; linked players on same team
- **`helpers.test.js`** — `categorizeHeight` at boundary values
- **`exporter.test.js`** — exported rows/columns match expected schema and optional redaction mode
- **Test:** `bun test` passes all tests
- **DoD:** Core logic has regression coverage and reproducible fixtures.

### Phase 12: Polish + Deploy
- Data privacy: verify zero `localStorage`/`sessionStorage`/cookie usage; add `beforeunload` listener as defense-in-depth
- Error handling for invalid files, impossible configs (e.g., more reps/team than total reps)
- Responsive Tailwind classes for laptop screens (columns wrap on narrow windows)
- Accessibility polish: keyboard navigation, visible focus states, ARIA labels on interactive controls
- Deploy to Netlify (see below)
- **DoD:** Pilot-ready release checklist completed and signed off.

---

## Deployment

**Google Cloud Console (one-time, before deploy):**
1. Create project → enable Drive API → configure OAuth consent (Testing mode) → add friends as test users
2. Create OAuth Client ID → set authorized origins to your Netlify URL + localhost
3. Create API key → restrict to your domain + Drive API
4. Store credentials in environment variables, not hardcoded constants:
   - `VITE_GOOGLE_CLIENT_ID`
   - `VITE_GOOGLE_API_KEY`
5. Restrict credentials to allowed origins and Drive API only

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
- Verify public-share warning and consent checkbox appears before publishing
- Verify optional redaction mode removes sensitive fields from shared output
- Close tab, reopen — confirm no data persists
- Deploy to Netlify, repeat key tests on live URL

---

## Data Contract (Input Schema)

Normalized player object:

```js
{
  id: string,                 // required, UUID if not supplied
  name: string,               // required, non-empty
  pronouns: string | null,    // optional
  height: number | null,      // optional, inches (must be realistic if present)
  skill: number,              // required, integer range 1-10
  representative: boolean,    // required
  status: "new" | "returning" | null, // optional but recommended
  linkedWith: string[]        // optional, ids of linked players
}
```

Validation requirements:
- Reject rows missing `name`, `skill`, or `representative`.
- Reject `skill` outside configured bounds (default 1-10).
- Reject malformed `linkedWith` references that point to unknown IDs.
- Emit row-indexed error messages so users can fix source files quickly.
