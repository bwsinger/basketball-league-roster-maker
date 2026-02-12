# Milestone 4: Manual Edit and Export

## Goal
Let the commissioner finalize rosters and export results.

## Why This Matters
Auto-balance is never perfect. This milestone teaches:
- controlled state updates from UI actions
- keeping exported output consistent with current app state
- closing the loop from input to actionable output

## Build
- Add manual reassignment controls (no drag-and-drop required):
  - Move a player to another team.
- Add CSV export of final team rosters.
- Ensure export reads from current in-memory state only.
- Use [MDN Blob API](https://developer.mozilla.org/en-US/docs/Web/API/Blob) and [MDN `URL.createObjectURL`](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL_static) when implementing browser download behavior.
  - Why these APIs: they are the standard browser-native way to generate and download files without a backend.

## Suggested Implementation Steps
1. Add "Move to team" control per player.
2. Implement a single state update function for reassignment.
3. Add export function that flattens team/player state into CSV rows (review [CSV basics](https://en.wikipedia.org/wiki/Comma-separated_values) if needed).
   - Why CSV: easy for commissioners to open/share in spreadsheet tools.
4. Trigger file download from browser.
5. Validate exported file manually in spreadsheet software.
6. If accidental mutation bugs appear, review [immutability patterns](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze).

## Deliverable
User can make final roster edits and export clean CSV output.

## Acceptance Checklist
- Can move a player between teams and state updates correctly.
- Exported CSV opens and reflects current roster state.
- Export includes all teams and assigned players.

## Common Failure Modes
- UI updates but internal state remains stale.
- Export includes old data after edits.
- Duplicate players appear after reassignment.

## Debugging Checklist
- Verify there is exactly one source of truth for team assignments.
- Log state right before export.
- Compare UI counts vs exported row counts.

## Reflection Prompts
- How did you keep state as a single source of truth?
- What export schema did you choose and why?
- What is the smallest change that would make export more robust?
