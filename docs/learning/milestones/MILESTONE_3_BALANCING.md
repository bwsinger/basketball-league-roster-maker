# Milestone 3: Balancing Algorithm

## Goal
Auto-assign players to teams using a basic balancing algorithm.

## Build
- Implement snake assignment in `rosterGenerator.js`.
- Sort by skill descending before assignment.
- Optional captain spread before normal assignment.
- Add "Balance Teams" action.
- Show per-team summary: player count and total skill.

## Acceptance Checklist
- Running balance fills teams automatically.
- Team size difference is <= 1.
- Total-skill spread appears reasonable.

## Common Failure Modes
- Off-by-one errors in snake direction switching.
- Mutating source arrays causes hard-to-debug UI issues.

## Reflection Prompts
- Why did you choose your algorithm implementation shape?
- What metric did you use to judge fairness?
