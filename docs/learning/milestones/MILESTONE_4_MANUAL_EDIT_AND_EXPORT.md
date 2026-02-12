# Milestone 4: Manual Edit and Export

## Goal
Let the commissioner finalize rosters and export results.

## Build
- Add manual reassignment controls (no drag-and-drop required):
- Move a player to another team.
- Add CSV export of final team rosters.

## Acceptance Checklist
- Can move a player between teams and state updates correctly.
- Exported CSV opens and reflects current roster state.

## Common Failure Modes
- UI updates but internal state remains stale.
- Export includes old data after edits.

## Reflection Prompts
- How did you keep state as a single source of truth?
- What export format decisions did you make?
