# Milestone 1: Foundation

## Goal
Create the app skeleton and league setup flow.

## Build
- Scaffold Svelte + Vite app.
- Create league setup form:
- `leagueName`
- `numTeams` (>= 2)
- `teamNames`
- Render empty team columns after setup.

## Acceptance Checklist
- Can create a league with custom team names.
- Form validation prevents invalid team count/name mismatch.
- Team columns render correctly after submission.

## Common Failure Modes
- Team names array length does not match `numTeams`.
- Missing validation makes app state inconsistent.

## Reflection Prompts
- What state belongs at app level?
- Why validate at input boundaries?
