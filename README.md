# Roadmap Dashboard

Generate interactive Gantt-style roadmap dashboards for engineering teams. Self-contained HTML, no dependencies, opens in any browser.

## What You Get

A single HTML file per team with:
- **Capacity grid** showing utilisation per iteration (PW planned vs available)
- **Gantt chart** with initiatives, epics, and bars positioned by iteration
- **Release milestone lines** marking key dates vertically through the chart
- **Badges** for release targets and programme milestones
- **Summary cards** with key metrics at a glance
- **Overlay comparison** (optional) to compare current plan vs a baseline
- **Scenario switcher** (optional) to toggle between planning options

## How to Use

### With Claude Code

1. Clone this repo into your project (or reference it)
2. Tell Claude: "Generate a roadmap dashboard for [team name]"
3. Claude follows `SKILL.md` to ask you about iterations, capacity, releases, and initiatives
4. Output: a populated HTML file ready to open in a browser

### Manual

1. Copy `template.html` to your project
2. Follow the commented sections to fill in your team's data
3. Refer to `guide.md` for structure details and maintenance procedures

## Files

| File | Purpose |
|------|---------|
| `template.html` | Bare-bones HTML skeleton with all CSS/JS, no data |
| `SKILL.md` | Onboarding instructions for Claude Code |
| `guide.md` | Maintenance guide: grid structure, bars, badges, scenarios, validation |
| `CLAUDE.md` | Instructions for Claude Code when working in this repo |

## Example

The original dashboard this was extracted from tracks 15+ initiatives across 7 iterations with 3 parallel release trains, scenario comparison, and Jira/Miro overlay toggles. The template supports all of this, but you can start simple with just a basic Gantt and add features as needed.

## Maintenance

Once a dashboard is generated, updates are straightforward. Common operations:
- "Move epic X from iteration 3 to iteration 4"
- "Add a new initiative with 3 epics"
- "Update the release date for milestone Y"

Claude Code can handle all of these using the maintenance procedures in `guide.md`. The validation checklist catches the most common issues (div imbalance, broken positioning).
