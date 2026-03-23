# CLAUDE.md

## What This Repo Is

A reusable toolkit for generating interactive Gantt-style roadmap dashboards. Any engineering team can use it to create a self-contained HTML dashboard showing their capacity, initiatives, and release targets.

## Key Files

- `template.html` — the HTML skeleton. All CSS and JS, no data. Commented insertion points.
- `SKILL.md` — the onboarding flow. Follow this when generating a new dashboard.
- `guide.md` — maintenance reference. How the grid, bars, badges, scenarios, and validation work.

## When Generating a Dashboard

Follow `SKILL.md` step by step. Key rules:

1. **Every row must have exactly N+1 elements** (1 label + N iteration cells). Div imbalance breaks the grid silently.
2. **Capacity grid and Gantt must agree.** If you place 6 PW of bars in an iteration, the capacity grid's planned load must reflect it.
3. **Validate after generation.** Run the div balance check before delivering.
4. **Start simple.** Basic Gantt first. Overlays, scenarios, and programme badges can be added later.

## When Maintaining a Dashboard

Follow `guide.md` section 10 for common operations. Always validate div balance after changes.

## Style

- Bar colours are semantic. Don't use colour arbitrarily.
- Tooltips (`title` attribute) carry the detail. Bar text should be scannable (e.g. "6 PW", not a sentence).
- Release milestone positions are calculated as percentages within iteration cells. Document the calculation in a comment.
