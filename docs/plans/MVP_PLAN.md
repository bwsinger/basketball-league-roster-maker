# Basketball League Roster Maker — Simplified Implementation Plan

## Mission

Build a small, practical web app that helps a commissioner:
1. Add players to rosters and form teams
2. Create a league with those teams
3. Auto-balance teams using a basic player-attribute algorithm

This project is both a real tool and a teaching project, so the plan prioritizes simplicity and completion over feature breadth.

---

## Scope (MVP Only)

### In Scope
- Add players manually in the UI
- Optional CSV import for players
- Create league settings (league name, number of teams, team names)
- View/edit team rosters
- Run a simple balance algorithm to distribute players
- Manually adjust assignments after auto-balance
- Export final teams to CSV

### Out of Scope (for now)
- Google Drive / Google Sheets integration
- OAuth and API key setup
- Drag-and-drop dependency integrations
- Player linking/package deals
- Locking teams during rebalance
- Public sharing links
- Multiple advanced balancing heuristics

---

## Success Criteria

### Product
- Commissioner can build a usable league from raw player info in under 10 minutes.
- Balanced output is acceptable for real friend-group use.
- No blocking bugs during a pilot run.

### Teaching
- Developer ships a complete app end-to-end (input -> algorithm -> output).
- Developer can explain each core part: state, forms, algorithm, validation, tests, deploy.

---

## Tech Stack (Minimal)

- Framework: Svelte + Vite
- Styling: Plain CSS (or light utility classes already in project)
- Parsing: `papaparse` (CSV only)
- IDs: `uuid`
- Testing: Vitest (logic-focused tests)
- Hosting: Netlify

Notes:
- Keep Bun if preferred, but document npm equivalents.
- Avoid adding dependencies unless they remove significant complexity.

---

## Data Model

Player:

```js
{
  id: string,
  name: string,
  skill: number,       // 1-10
  height: number|null, // inches, optional
  isCaptain: boolean   // optional balancing signal
}
```

League:

```js
{
  leagueName: string,
  numTeams: number,
  teamNames: string[]
}
```

Team:

```js
{
  id: string,
  name: string,
  players: Player[]
}
```

Validation rules:
- `name` required
- `skill` required and must be 1-10
- `numTeams` must be >= 2
- `teamNames.length` must equal `numTeams`

---

## Balancing Algorithm (Basic)

Goal: evenly distribute player strength and avoid obvious team stacking.

1. Sort players by `skill` descending.
2. Use snake assignment across teams:
- Round 1: Team 1 -> Team N
- Round 2: Team N -> Team 1
- Repeat until all players assigned
3. If `isCaptain` is used, spread captains first (one per team where possible).
4. Return assigned teams; allow manual edits afterward.

Acceptance checks:
- Max total-skill gap between strongest and weakest team <= agreed threshold (start with 5-8 points, tune during pilot).
- Team size difference <= 1 player.

---

## Implementation Phases

### Phase 1: App Skeleton + League Setup
- Scaffold app structure
- Build league setup form (league name, team count, team names)
- Render empty team columns
- DoD: user can create a league and see teams

### Phase 2: Player Entry + CSV Import
- Build player form (name, skill, optional height, optional captain)
- Add CSV import for the same schema
- Add validation with clear inline errors
- DoD: players can be added/imported reliably

### Phase 3: Auto-Balance
- Implement snake-based balance function in `rosterGenerator.js`
- Add "Balance Teams" action
- Show team summary metrics (count + total skill)
- DoD: teams auto-populate and pass acceptance checks

### Phase 4: Manual Adjustment + Export
- Allow moving a player from one team to another (simple select controls/buttons)
- Add CSV export of final rosters
- DoD: commissioner can finalize and export teams

### Phase 5: Tests + Deploy
- Unit tests for parser/validation and balancing algorithm
- Smoke test key UI flow manually
- Deploy to Netlify
- DoD: live MVP usable by friend group

---

## Project Structure

```
basketball-roster-maker/
├── src/
│   ├── App.svelte
│   ├── lib/
│   │   ├── state/roster.svelte.js
│   │   ├── logic/parser.js
│   │   ├── logic/rosterGenerator.js
│   │   ├── logic/exporter.js
│   │   └── logic/validation.js
│   └── app.css
├── tests/
│   ├── parser.test.js
│   ├── rosterGenerator.test.js
│   └── validation.test.js
└── README.md
```

---

## Verification Checklist

- Create a league with custom team names
- Add 20+ players manually
- Import players via CSV
- Run balance algorithm and verify team-size and skill-gap checks
- Manually move players and confirm updates
- Export CSV and verify content
- Run tests successfully
- Verify deployed app works on desktop and mobile

---

## Future Enhancements (Post-MVP)

Only consider after MVP is stable in real use:
- Drag-and-drop UX
- Google Sheets sharing
- Team locking and linked-player constraints
- More advanced balancing heuristics and weighting controls
