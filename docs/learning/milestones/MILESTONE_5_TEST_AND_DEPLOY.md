# Milestone 5: Test and Deploy

## Goal
Make the MVP reliable and live.

## Why This Matters
This milestone teaches shipping discipline:
- writing tests for core logic
- validating full user flow
- deploying and verifying production behavior

## Build
- Add unit tests for:
  - validation rules
  - CSV parsing normalization
  - balancing algorithm behavior
- Use [Vitest docs](https://vitest.dev/guide/) while setting up and writing tests.
  - Why this tool: Vitest is fast, simple, and fits naturally with Vite projects.
- Use [Testing Library guiding principles](https://testing-library.com/docs/guiding-principles/) to keep tests user-focused where applicable.
  - Why this guidance: it helps avoid brittle tests tied to implementation details.
- Run a manual smoke test of full flow.
- Deploy to Netlify using [Netlify deploy docs](https://docs.netlify.com/site-deploys/create-deploys/).
  - Why this platform: Netlify is low maintenance for static SPA hosting.

## Suggested Implementation Steps
1. Add test setup and test command.
2. Write tests for validation edge cases.
3. Write tests for parser normalization behavior.
4. Write tests for balancing invariants (team size diff <= 1, expected assignment count).
5. Run manual end-to-end checklist locally.
6. Deploy and rerun smoke checks on live URL.
7. If behavior differs between local and browser, inspect with [Chrome DevTools docs](https://developer.chrome.com/docs/devtools/).
   - Why this tool: DevTools is the fastest way to inspect runtime errors and network issues in the browser.

## Deliverable
A working deployed MVP with passing tests for core logic.

## Acceptance Checklist
- Tests pass locally.
- Deployed app supports full MVP flow.
- Can run one pilot with real data without blocker bugs.
- At least one bug discovered during testing is fixed before final milestone sign-off.

## Common Failure Modes
- Tests only cover happy path.
- Tests depend on mutable global state.
- Deploy succeeds but runtime behavior differs from local.

## Debugging Checklist
- Run tests before every commit for this milestone.
- Add one regression test for each bug fixed.
- Verify deployed build with fresh browser session.

## Reflection Prompts
- Which tests gave you the most confidence?
- Which critical behavior is still untested?
- What would you harden first after pilot feedback?
