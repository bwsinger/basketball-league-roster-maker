# Milestone 2: Player Input

## Goal
Support player entry and CSV import.

## Build
- Manual player form with:
- `name` (required)
- `skill` (required 1-10)
- `height` (optional)
- `isCaptain` (optional)
- CSV parser for the same schema.
- Inline error display for invalid rows.

## Acceptance Checklist
- Can add a player manually and see them in the unassigned list.
- Can import CSV and load players.
- Invalid rows are rejected with clear messages.

## Common Failure Modes
- Skill parsed as string and not normalized to number.
- CSV import silently skips errors.

## Reflection Prompts
- What data normalization did you add?
- What input errors were hardest to handle?
