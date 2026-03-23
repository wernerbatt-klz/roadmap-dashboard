# Roadmap Dashboard — Onboarding Skill

Generate an interactive Gantt-style roadmap dashboard for any engineering team. Produces a self-contained HTML file with capacity utilisation, initiative timeline, release milestones, and optional scenario comparison.

## When to Use

- A team wants a visual roadmap dashboard
- Someone asks to "build a roadmap" or "create a capacity dashboard"
- A PM or EM needs to visualise their team's iteration plan

## Dependencies

- `template.html` — the HTML skeleton (same directory)
- `guide.md` — maintenance reference (same directory)

## Onboarding Flow

Follow these steps in order. Ask questions conversationally, not as a form. Gather what you can from context (Jira, existing docs) before asking.

### Step 1: Team Basics

Ask:
- **Team name** — used in the page title and header
- **Date** — "as of" date for the dashboard

### Step 2: Iteration Structure

Ask:
- **Iteration names and dates** — e.g. "I15: 23 Mar – 1 May", "Sprint 42: 1 Apr – 14 Apr"
- **Iteration duration** — e.g. 6 weeks, 2 weeks
- **How many iterations to show** — typically 6-8 for a rolling view

Update the CSS `grid-template-columns` and JS `cols` variable to match.

### Step 3: Capacity Model

Ask:
- **Engineers** — names, roles, any caveats (part-time, ramping, specialist)
- **Capacity formula** — how do you calculate usable PW per iteration?
  - Common pattern: `(engineers x iteration_weeks) - support_overhead`
  - Any resources excluded from engineering capacity? (FE, QA, contractors)
- **Utilisation targets** — what's the goal vs ceiling?
  - Common: 90% goal, 110% ceiling

Populate the capacity grid with calculated values.

### Step 4: Release Trains

Ask:
- **What release trains does this team ship through?** — e.g. "quarterly releases", "monthly deploys", "programme milestones"
- **For each train: milestone names and dates**
- **Which is the primary? Are there cross-programme dependencies?**

For each release milestone:
1. Calculate position percentage within the iteration it falls in
2. Add a `.release-marker` in the milestone row
3. Add CSS pseudo-element rules for vertical lines
4. Add badge CSS rules (`.release-badge.rX`)
5. Add legend items

### Step 5: Initiatives and Epics

Ask:
- **What are the initiatives?** — name, Jira key (if applicable), release target, programme badges
- **For each initiative, what are the epics?** — name, Jira key, size (PW), which iteration, confidence

For each initiative:
1. Add an initiative header row with badges
2. Add epic rows with bars positioned in the correct iteration cells
3. Choose bar colours based on status (green = on track, amber = shifted, blue = hard deadline, etc.)
4. Add tooltips with sizing rationale

### Step 6: Summary Cards

Based on the capacity data, create 3-4 summary cards:
- One per "interesting" iteration (the ones with pressure)
- One for key risks or slips
- One for overall status (reds/ambers count)

Use `ok` (green), `warn` (amber), or `bad` (red) classes.

### Step 7: Notes Section

Generate a structured notes section covering:
- Key decisions and trade-offs
- Release target analysis
- Cross-programme milestones (if any)
- Current plan summary

### Step 8: Polish

1. **Legend** — ensure every visual element used in the chart has a legend entry
2. **Overlay** — if there's a previous plan to compare against, set up ghost bars
3. **Scenarios** — if the team needs to compare options, set up the scenario system (see guide.md section 6)
4. **Validate** — run the div balance check:
   ```bash
   python3 -c "
   with open('OUTPUT_FILE.html') as f:
       c = f.read()
   print('divs:', c.count('<div'), '/', c.count('</div>'))
   "
   ```

### Step 9: Review (optional, recommended)

After the dashboard is generated and validated, offer to do a critical review. If the user accepts, open the HTML in a browser (or reason about the structure) and assess:

**Data quality**
- Are any initiatives missing that should be on the roadmap? Cross-check against the Jira project and any planning docs provided.
- Are sizing estimates present for all epics, or are some marked with "?" or "TBD"? Flag these.
- Do the capacity grid totals match the sum of bars in each iteration? Recount if unsure.

**Visual clarity**
- Can you scan the chart and immediately see which iterations are under pressure? If not, are the summary cards doing that job?
- Are there too many bar colours? More than 6-7 active colours becomes hard to read. Suggest consolidating if needed.
- Are initiative rows clearly separated from epic rows? If the chart is dense, suggest adding background colour to initiative rows.
- Are bar labels readable, or are some bars too narrow for their text? Suggest moving text to tooltips for narrow bars.

**Completeness**
- Does every initiative have at least one release badge or target? If not, flag it as "untagged, needs a release home."
- Are there iterations with no headroom and no risks flagged? That's a red flag worth calling out.
- Is the notes section substantive, or just a placeholder? It should capture the "why" behind the plan, not just restate what the chart shows.

**Risks surfaced**
- Any single engineer on the critical path for >50% of an iteration's work? Flag as single point of failure.
- Any items squeezed right up against a release deadline with zero buffer? Flag as zero-margin.
- Any initiatives spanning 3+ iterations? These tend to slip. Flag for attention.

Present the review as a numbered list of **suggested improvements**, categorised as:
- **Data gaps** — missing or uncertain information that should be filled in
- **Visual improvements** — layout, colour, readability changes
- **Risk flags** — things the team should be aware of or discuss

### Step 10: Improvement PR (follows Step 9)

After presenting the review, offer to implement the improvements:

1. Create a new branch (e.g. `dashboard-improvements`)
2. Apply the suggested changes (data fixes, visual tweaks, risk annotations)
3. Re-validate div balance
4. Create a PR with:
   - Title summarising the changes
   - Body listing each improvement made, grouped by category
   - Any remaining open items that need human input (e.g. missing sizing data)

This gives the team a clean diff to review and a natural checkpoint before merging changes into their dashboard.

## Output

Write the populated HTML file to the team's chosen location. Name it descriptively, e.g. `payments-roadmap.html`, `platform-roadmap.html`.

Provide a brief summary of what was generated:
- How many initiatives and epics
- Key capacity figures
- Any risks or flags surfaced during population

## Maintenance

After generating the initial dashboard, point the team to `guide.md` for ongoing maintenance. Key sections:
- Moving epics between iterations (guide section 10)
- Adding new initiatives (guide section 10)
- Updating release dates (guide section 10)
- Validation checklist (guide section 9)

The guide is designed to be read by both humans and Claude Code. Future updates can be done conversationally: "move PROJ-1234 from Iter 3 to Iter 4" and Claude can follow the guide to make the change correctly.

## Customisation Points

These are the things that vary most between teams:

| What | Where | Notes |
|------|-------|-------|
| Column count | CSS `grid-template-columns`, JS `cols` | Label + N iterations |
| Bar colours | CSS `.bar.X` rules | Rename to match team vocabulary |
| Badge types | CSS `.release-badge.X`, `.programme-badge.X` | One set per release train |
| Capacity formula | Capacity grid values, subtitle text | Team-specific deductions |
| Release milestone colours | CSS `.release-marker.X` | One per release milestone |
| Jira project key | Row label links | Team's Jira project |

## Tips

- **Start simple.** Don't add scenarios or overlays on the first pass. Get the basic Gantt right, then layer complexity.
- **Bar text should be scannable.** Keep it to "X PW" or a short status. Details go in the tooltip.
- **Validate after every batch of changes.** Div imbalance is the most common bug and the hardest to debug visually.
- **The capacity grid and Gantt must agree.** If you add 6 PW to an iteration's bars, the capacity grid's planned load must increase by 6.
