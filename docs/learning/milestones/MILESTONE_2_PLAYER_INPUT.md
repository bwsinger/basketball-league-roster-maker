# Milestone 2: Player Input

## Goal
Support player entry and CSV import.

## Why This Matters
This milestone teaches data modeling and validation:
- converting user input into consistent internal objects
- handling invalid input safely
- importing structured data from files

## Build
- Manual player form with:
  - `name` (required)
  - `skill` (required 1-10)
  - `height` (optional)
  - `isCaptain` (optional)
- Use [Svelte bindings/events](https://svelte.dev/docs/svelte/bind) when wiring the form.
  - Why this tool: Svelte bindings reduce manual input-handling code and keep form state predictable.
- CSV parser for the same schema using [Papa Parse docs](https://www.papaparse.com/docs).
  - Why this tool: Papa Parse handles CSV edge cases (headers, empty lines, quoting) better than hand-rolled parsing.
- Use [MDN File API guide](https://developer.mozilla.org/en-US/docs/Web/API/File_API/Using_files_from_web_applications) for upload flow.
  - Why this API: the browser File API is the standard way to safely read uploaded files client-side.
- Normalization logic so all players share one object shape.
- Inline error display for invalid rows/inputs.

## Suggested Implementation Steps
1. Define a canonical `Player` object shape.
2. Build manual add-player form with validation.
3. Add CSV upload input (`.csv`) and parse with `papaparse`.
4. Normalize parsed row values (trim strings, convert numbers, booleans) using [MDN Number reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) for numeric conversion behavior.
   - Why this step: normalization prevents subtle bugs later in balancing/export logic.
5. Reject invalid rows with clear messages that include row numbers.
6. Merge valid players into the unassigned list.

## Deliverable
User can add players manually or via CSV and see them in one unassigned roster list.

## Acceptance Checklist
- Can add a player manually and see them in the unassigned list.
- Can import CSV and load players.
- Invalid rows are rejected with clear messages.
- Player objects have consistent shape after manual add and CSV import.

## CSV Contract (Minimum)
Expected headers:
- `name`
- `skill`
- `height` (optional)
- `isCaptain` (optional)

## Common Failure Modes
- `skill` parsed as a string and never converted to number.
- Empty rows get treated as valid players.
- CSV import silently skips errors.

## Debugging Checklist
- Console log one parsed row before normalization.
- Console log one normalized player object.
- Add a temporary count: total rows, valid rows, invalid rows.

## Reflection Prompts
- What normalization rules did you add?
- Where should validation happen: input layer, parser layer, or both?
- Which error messages were most useful to you during debugging?
