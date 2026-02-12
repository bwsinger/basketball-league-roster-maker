# Milestone 1: Foundation

## Goal
Create the app skeleton and league setup flow.

## Why This Matters
This milestone teaches the baseline architecture of the app:
- how UI state drives rendering
- how forms update application state
- how to create a predictable foundation for later features

## What a Single-Page App Should Look Like
A single-page app (SPA) is one HTML page where JavaScript updates the UI as state changes. For this project, aim for this simple mental model:

1. **State layer** (source of truth)
- league settings
- player list
- teams and assignments

2. **UI layer** (reads state and renders)
- setup form
- unassigned players view
- team columns
- action buttons

3. **Logic layer** (pure functions where possible)
- validation
- CSV parsing (later milestone)
- balancing algorithm (later milestone)

For this milestone, only establish the state + setup UI foundation.

## Why SPA + Svelte + Vite Instead of Vanilla HTML/JS
- **Vanilla HTML/JS is still valid** and great for learning basics.
- For this project, the UI changes often as state changes (league setup, player lists, team columns). In vanilla JS, manual DOM updates can become repetitive and error-prone.
- **Svelte** helps because UI is declared from state, so updates are simpler and easier to reason about.
- **Vite** helps because local development is fast (quick startup and hot reload), which shortens the feedback loop.
- Net effect: less boilerplate DOM code, faster iteration, and lower chance of UI/state mismatch.

## Suggested Initial File Layout
Use this structure as a starting point:

```text
src/
  App.svelte
  app.css
  lib/
    state/
      roster.svelte.js
    logic/
      validation.js
```

- Keep state in one place (`roster.svelte.js` or equivalent).
- Keep validation as small reusable functions.
- Keep `App.svelte` focused on rendering + event wiring.

## UI and State Flow (High Level)
1. User types league settings into form fields.
2. Form submit triggers validation.
3. If valid, state is updated.
4. UI rerenders team columns from state.
5. If invalid, errors are shown without changing committed state.

## Prerequisites
- Node/Bun installed
- Repo cloned locally
- You can run `npm run dev` or `bun run dev`

## Build
- Scaffold Svelte + Vite app (if not already scaffolded) using [Vite getting started](https://vite.dev/guide/).
  - Why this tool: **Vite** gives a fast dev server and quick reloads while learning.
- Review [Svelte tutorial](https://svelte.dev/tutorial) sections on reactivity and forms before building the setup UI.
  - Why this tool: **Svelte** makes state-driven UI updates straightforward.
- Create league setup form fields:
  - `leagueName`
  - `numTeams` (>= 2)
  - `teamNames`
- If form behavior is unclear, use [MDN forms guide](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms).
- Add a submit action that creates league state.
- Render empty team columns after setup.
- Keep league state in one source of truth (top-level state module or `App.svelte`) using the [Svelte docs overview](https://svelte.dev/docs/svelte/overview) as reference.

## Suggested Implementation Steps
1. Define a league state object with safe defaults.
2. Build a controlled form bound to state (see [Svelte bind docs](https://svelte.dev/docs/svelte/bind) if needed).
   - Why this feature: binding keeps form inputs and state in sync without manual DOM querying.
3. Validate `numTeams` and `teamNames.length` before saving.
4. On success, render team columns from state instead of hardcoded markup.
5. Show a clear error message when validation fails.
6. Sanity-check architecture: no duplicated state and no hardcoded team columns.

## Deliverable
A page where a user can enter league settings and immediately see the correct number of team columns with team names.

## Build Order
If you are unsure where to start, follow this exact order:
1. Render static league setup form.
2. Add state object and bind form inputs.
3. Add submit handler and `console.log` submitted values.
4. Add validation and display errors.
5. Replace static team columns with state-driven rendering.

## Acceptance Checklist
- Can create a league with custom team names.
- Form validation prevents invalid team count/name mismatch.
- Team columns render correctly after submission.
- Refreshing the page resets state (no persistence yet).

## Common Failure Modes
- Team names array length does not match `numTeams`.
- Form allows empty team names.
- State updates in multiple places and becomes inconsistent.

## Debugging Checklist
- Log league state after submit.
- Confirm render loops derive directly from state.
- Confirm validation runs before render state updates.

## Reflection Prompts
- What state belongs at app level?
- Why validate at input boundaries?
- What changed when you moved from hardcoded UI to state-driven UI?
