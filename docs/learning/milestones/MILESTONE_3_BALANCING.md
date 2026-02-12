# Milestone 3: Balancing Algorithm

## Goal
Auto-assign players to teams using a basic balancing algorithm.

## Why This Matters
This is the core product value. It also teaches:
- algorithm design
- array/data transformation
- separating pure logic from UI

## Build
- Implement snake assignment in `rosterGenerator.js`.
  - Why this approach: a simple snake draft is easy to understand, implement, and debug.
- Sort players by `skill` descending before assignment using [MDN `Array.sort`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort).
- Optional captain spread before normal assignment.
- Add "Balance Teams" action.
- Show per-team summary: player count and total skill (helpful with [MDN Array methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)).

## Suggested Implementation Steps
1. Write a pure function that accepts players + team count and returns teams.
2. Add sorting by descending `skill`.
3. Implement snake direction switching logic (forward, then reverse).
4. Add optional captain pre-assignment pass.
5. Wire function to UI button and render results.
6. Compute and display team-level metrics.
7. If complexity/performance questions come up, use this gentle [Big-O intro](https://www.geeksforgeeks.org/understanding-time-complexity-simple-examples/).
   - Why this concept: Big-O helps you judge whether the algorithm stays practical as player count grows.

## Deliverable
Clicking "Balance Teams" assigns unassigned players into teams with reasonable fairness.

## Acceptance Checklist
- Running balance fills teams automatically.
- Team size difference is <= 1.
- Total-skill spread appears reasonable.
- Running balance twice with same input gives predictable behavior (unless intentional randomness is added).

## Common Failure Modes
- Off-by-one errors in snake direction switching.
- Mutating source arrays causes hard-to-debug UI issues.
- Team index goes out of bounds at direction transitions.

## Debugging Checklist
- Test algorithm with small fixed datasets (4, 6, 8 players).
- Print assignment order and final team totals.
- Write one tiny test for direction switching specifically using [Vitest](https://vitest.dev/guide/).
  - Why this tool: Vitest is lightweight and integrates cleanly with Vite-based projects.

## Reflection Prompts
- Why did you choose this algorithm shape?
- What fairness metric is most practical for this app?
- What tradeoff did you make between simplicity and perfect balance?
