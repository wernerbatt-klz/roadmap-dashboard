# Roadmap Dashboard — Maintenance Guide

This guide covers the structure, conventions, and maintenance procedures for dashboards generated from `template.html`. Each dashboard is a standalone HTML document (no external dependencies) that renders an interactive Gantt-style roadmap.

---

## 1. Grid Structure

The visualisation is a CSS Grid. The default layout has **8 columns** (1 label + 7 iterations), but this is configurable.

| Column | Index | Content |
|--------|-------|---------|
| Row label | 0 | Initiative/epic name, badges, PW |
| Iteration 1 | 1 | First iteration |
| Iteration 2 | 2 | Second iteration |
| ... | ... | ... |
| Iteration N | N | Last iteration |

To change the iteration count, update **three places**:
1. `.capacity-grid` CSS: `grid-template-columns: 280px repeat(N, 1fr);`
2. `.gantt` CSS: `grid-template-columns: 280px repeat(N, 1fr);`
3. `const cols = N+1;` in the column class applicator JS

Every row consists of exactly **N+1 elements**: 1 `.row-label` + N `.cell` divs. The div count must always balance.

### Row Types

| Class | Purpose | Background |
|-------|---------|------------|
| `.row-label.initiative` | Initiative header row | light grey |
| `.row-label` (no `.initiative`) | Epic/detail row | white |
| `.cell.initiative-row` | Empty cells in initiative header rows | light background |
| `.cell` | Data cells containing bars | white |

---

## 2. Bar System

Bars represent work items positioned within cells using percentage-based `left` and `width` (or `right`).

### Bar Colours (CSS classes)

| Class | Colour | Default Meaning |
|-------|--------|-----------------|
| `.green` | #4caf50 | On track / fits in planned iteration |
| `.amber` | #ff9800 | Shifted +1 iteration |
| `.red` | #f44336 | At risk |
| `.blue` | #1976d2 | Hard deadline item |
| `.grey` | #90a4ae | Future / low priority |
| `.teal` | #00897b | Specialist resource (e.g. FE engineer) |
| `.config` | #7e57c2 | Config / ops work |
| `.committed` | #ff9800 | Committed work (distinct from shifted) |
| `.not-counted` | #cfd8dc (60% opacity) | Excluded from capacity count |
| `.at-risk` | dashed red border, no fill | At-risk marker |
| `.slip` | striped orange | Slipping item |
| `.deferred` | dashed grey border | Deferred to a later phase |
| `.overlay-ghost` | dashed border, low opacity | Shows where item was in baseline plan |

Rename or extend these to match your team's conventions. The colours are the vocabulary of the chart.

### Bar Positioning

```html
<!-- Percentage-based within the cell -->
<div class="bar green" style="left:2%;width:45%">6 PW</div>

<!-- Or using right edge -->
<div class="bar amber" style="left:50%;right:5%">~6 PW</div>
```

- `left` = start position within iteration (0% = start, 100% = end)
- `width` or `right` = duration within the iteration
- `title` attribute = tooltip with full details (sizing, rationale, context)

### Spanning Multiple Iterations

For items that span iterations, place a bar in each iteration cell:

```html
<!-- Iteration 3 cell -->
<div class="cell">
  <div class="bar blue" style="left:60%;right:0%">6 PW</div>
</div>
<!-- Iteration 4 cell -->
<div class="cell">
  <div class="bar blue" style="left:0%;width:30%"></div>
</div>
```

---

## 3. Badge System

Badges appear in row labels to show release/programme targets.

### Badge Types

| Class | Purpose | Customise for |
|-------|---------|---------------|
| `.badge.primary` | Primary release classification | Your main release (e.g. R3, v2.0) |
| `.badge.secondary` | Deferred / secondary | Lower priority (e.g. R4, backlog) |
| `.release-badge` | Sub-release target | Specific release milestone (e.g. R3.3, Sprint 5) |
| `.programme-badge` | Cross-programme milestone | External programme targets |
| `.fe-note` | Specialist resource note | FE/BE split, contractor, etc. |

### Adding a Badge

```html
<span class="badge primary">R1</span>
<span class="release-badge r2">v2.1</span>
<span class="programme-badge prog-a" title="Programme A — 30 Jun, on track">PA</span>
```

Always include a `title` attribute with the full date and status for tooltips.

### Defining Badge Colours

Add CSS rules for each release/programme badge:

```css
.release-badge.r1 { background: #e1bee7; color: #6a1b9a; }
.release-badge.r2 { background: #b3e5fc; color: #01579b; }
.programme-badge.prog-a { background: #e8f5e9; color: #2e7d32; border: 1px solid #a5d6a7; }
```

---

## 4. Release Milestone Lines

Vertical lines through the Gantt mark release deadlines.

### Milestone Row

A dedicated row at the top of the Gantt shows release date markers:

```html
<div class="row-label" style="...">Releases</div>
<div class="cell" style="...;position:relative">
  <div class="release-marker m1" style="left:75%">
    <div class="release-label">Release 1 — DD Mon</div>
  </div>
</div>
<!-- one cell per iteration column -->
```

The `.release-marker` positions a vertical line + label at a percentage within the iteration cell.

### Calculating Position Percentages

```
position% = (target_date - iteration_start) / (iteration_end - iteration_start) x 100
```

Example: Release date is 21 Apr. Iteration runs 23 Mar to 1 May (39 days). 21 Apr is day 29. Position = 29/39 x 100 = 75%.

### Vertical Lines Through All Rows

CSS pseudo-elements on column cells draw vertical dashed lines through every data row:

```css
.gantt .cell.col-2-line { position: relative; }
.gantt .cell.col-2-line::before {
  content: ''; position: absolute;
  left: 75%; top: 0; bottom: 0;
  width: 2px; background: #7b1fa2; opacity: 0.3;
  z-index: 3; pointer-events: none;
}
```

The JavaScript at the bottom applies column classes to every data cell:

```javascript
document.querySelectorAll('.gantt').forEach(gantt => {
  const items = Array.from(gantt.children);
  const cols = 8; // total columns
  for (let i = cols; i < items.length; i++) {
    const col = i % cols;
    if (col === 2) items[i].classList.add('col-2-line');
  }
});
```

**Important:** Labels are in the milestone row only. Pseudo-elements draw lines only (no `content` text). This prevents duplicate labels.

---

## 5. Overlay System

The overlay lets you compare the current plan to a previous baseline (e.g. original Miro plan, Jira dates).

### How It Works

1. Toggle button adds/removes `show-overlay` class on `<body>`
2. Ghost bars (`.overlay-ghost`) are hidden by default, shown when overlay active
3. Shifted bars get a highlight shadow when overlay is active
4. "Before" capacity rows are hidden by default, shown when overlay active

### Adding an Overlay Comparison

For each bar that moved from its original position:

```html
<!-- In the ORIGINAL iteration cell -->
<div class="bar overlay-ghost was-green" style="left:2%;width:45%">was here</div>

<!-- In the CURRENT iteration cell -->
<div class="bar green shifted" title="Moved from Iter 2 to Iter 3" style="left:2%;width:45%">6 PW</div>
```

---

## 6. Scenario System (Optional)

For teams that need to compare multiple planning options.

### How It Works

1. Each bar that varies by scenario has **scenario classes**: `s-a`, `s-b`, `s-c`
2. All scenario bars are hidden by default: `.s-bar { display: none !important; }`
3. The active scenario's bars are shown: `body[data-scenario="a"] .s-a { display: flex !important; }`
4. Bars shared across scenarios have multiple classes: `s-a s-b s-c`
5. Bars that are always visible have no `s-bar` class

### Setup Checklist

1. **CSS:** Add `body[data-scenario="X"] .s-X { display: flex !important; }` for each scenario
2. **Button:** Add `<button class="sbtn" data-scenario="X" onclick="setScenario('X')">X: Name</button>`
3. **JS:** Add scenario data to the `SCENARIOS` object
4. **Bars:** Tag each varying bar with `s-bar s-X`
5. **Default:** Call `setScenario('X')` at the bottom of the script

---

## 7. Capacity Grid

The capacity grid shows utilisation per iteration. Standard rows:

| Row | Purpose |
|-----|---------|
| Usable capacity | Total available PW after deductions (support, leave, etc.) |
| Planned load | Sum of all planned epic PW in this iteration |
| Utilisation | Planned / Usable as percentage |
| Headroom | Usable - Planned |

### Colour coding

- `.ok` (green): healthy utilisation (under target)
- `.tight` (orange): near ceiling
- `.over` (red, bold): over-committed
- `.highlight`: current iteration (amber background)
- `.highlight-after`: post-replan improvement (green background)

---

## 8. Notes Section

Below the Gantt, a `<div class="notes">` contains structured HTML with:
- Key decisions and trade-offs
- Release train summary
- Cross-programme milestones
- Risk callouts

Update this whenever the plan changes.

---

## 9. Validation Checklist

Run after every edit:

```python
with open('your-roadmap.html') as f:
    c = f.read()

# 1. Div balance — must match
assert c.count('<div') == c.count('</div>'), "Div mismatch!"

# 2. Row element count — every row must have exactly (cols) elements
# Count row-labels and verify against cell count

# 3. Bar positions — all left/width/right values should be 0-100%
import re
for m in re.finditer(r'left:(\d+)%', c):
    assert 0 <= int(m.group(1)) <= 100

# 4. If using scenarios: every scenario has a button and CSS rule
```

### Quick Bash Check

```bash
python3 -c "
with open('your-roadmap.html') as f:
    c = f.read()
print('divs:', c.count('<div'), '/', c.count('</div>'))
"
```

---

## 10. Common Maintenance Tasks

### Moving an epic from one iteration to another

1. Find the epic's row (search for its Jira key)
2. In the **source** cell: remove or reposition the bar. Optionally add an `overlay-ghost` bar.
3. In the **target** cell: add a new bar
4. Update capacity grid values
5. Update notes section
6. Validate div balance

### Adding a new initiative

1. Add an initiative header row: `.row-label.initiative` + N empty `.cell.initiative-row` divs
2. Add epic rows below (1 `.row-label` + N `.cell` each)
3. Add badges for release targets
4. Place bars in the appropriate iteration cells
5. Update capacity figures
6. Validate div balance

### Updating release dates

1. Recalculate position percentages for milestone markers
2. Update CSS pseudo-element `left` values for vertical lines
3. Update milestone row label text
4. Update notes section

### Adding/removing iterations

1. Update CSS `grid-template-columns` for both `.capacity-grid` and `.gantt`
2. Add/remove header cells
3. Add/remove cells in every existing row (balance must be maintained)
4. Update `const cols` in the JS column class applicator
5. Validate div balance

---

## 11. File Size & Performance

The template is self-contained HTML with inline CSS and JS. No build step. Opens in any browser.

If a dashboard grows large (>1000 divs), consider:
- Collapsing deferred/future items into summary rows
- Archiving old scenarios
- Lazy-rendering distant iterations
