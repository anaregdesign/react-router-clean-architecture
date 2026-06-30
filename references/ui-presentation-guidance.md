# UI Presentation Guidance

Covers chart/data-visualization choices and responsive/mobile behavior for the
app's presentational layer. For component file shape, component-library usage,
and CSS Modules, see
[`component-file-and-css-module-rules.md`](component-file-and-css-module-rules.md).
Verify rendered results with
[`playwright-ui-verification.md`](playwright-ui-verification.md).

## Charts And Data Visualization

### Goal

Use the simplest accessible chart that answers the product question without
overloading the canvas.

### Choose The Chart Deliberately

- Use line charts for continuous trends over time.
- Use bar or column charts to compare values across discrete categories or
  distinct periods.
- Use combo charts only when line and bar marks truly share the same X-axis
  and the comparison would be weaker as separate views.
- Add a secondary Y-axis only when measures genuinely have different scales,
  and label both axes explicitly.

### Keep The Visual Quiet

- Keep data ink as the primary focus. Avoid 3D effects, heavy shadows,
  decorative gradients, and non-informational chrome.
- Keep axis labels readable and keep gridlines visually secondary.
- Use theme-aware colors instead of ad hoc hardcoded palettes.
- Keep chart titles, legends, and units short and easy to scan.
- Match legend markers to the actual chart geometry so users do not decode
  shapes twice.

### Labels, Tooltip, And Interaction

- Keep key context visible: chart title, timeframe, units, series names,
  thresholds, and major status cues.
- Use Tooltip for supplemental inspection, not as the only place where users
  can discover the chart's meaning.
- Keep interactions simple and direct. Add filtering, drilldown, or zoom only
  when they materially improve exploration.
- If a chart becomes crowded, reduce the number of visible series or split the
  view instead of stacking more colors, legends, and labels into one canvas.

### Motion

- Animate only meaningful state changes.
- If both axes and marks change, transition the axes first and the data
  second.
- Avoid independent animation of many marks at the same time.
- Prefer subtle motion that helps users track change instead of decorative
  animation.

### Chart Accessibility

- Never rely on color alone. Add labels, shape differences, annotations, or
  patterns when distinctions matter.
- Keep keyboard access and focus treatment intentional for interactive chart
  controls.
- For important charts, provide a nearby text summary and, when exact values
  matter, an accessible table or equivalent non-hover path to the data.
- If the chart behaves like a complex image, connect it to a visible text
  description with standard accessible markup.
- Do not hide critical insight, error state, or threshold meaning only inside
  a hover Tooltip.

## Responsive And Mobile UI

### Goal

Keep the UI usable, readable, and touch-friendly from narrow mobile screens up
through desktop without forking the product into separate architectures.

### Core Approach

- Design responsive layouts that stay capable on both mobile and desktop.
  Start from the most constrained layout when it helps, but do not treat a
  rigid mobile-first process as mandatory.
- Use fluid layout primitives such as CSS Grid, Flexbox, intrinsic sizing, and
  percentage or `clamp()`-based sizing before adding hard breakpoints.
- Choose breakpoints from content pressure, not from device marketing names.
- Keep media, cards, panels, and data containers able to shrink or wrap
  instead of assuming desktop width.

### Document And Viewport Basics

- Keep the standard mobile viewport meta tag in the document head:
  `width=device-width, initial-scale=1`.
- Add `viewport-fit=cover` only when the layout intentionally handles
  safe-area insets.
- Treat ordinary app content as needing reflow at narrow widths rather than
  requiring two-dimensional scrolling.
- Do not lock orientation unless one orientation is genuinely essential to the
  task.

### Interaction On Touch Devices

- Do not make hover the only way to discover required actions or meaning.
- Keep a touch path and keyboard path for the same important task.
- Meet at least WCAG 2.2 minimum target size expectations or provide
  equivalent spacing between targets.
- Prefer larger touch targets when practical, especially for primary actions,
  icon-only buttons, and dismiss controls.
- Be careful with sticky headers, sticky footers, and bottom action bars so
  they do not hide inputs or validation on small screens.

### Data-Dense UI On Small Screens

- If a table, chart, or filter bar becomes unreadable on mobile, adapt the
  presentation instead of shrinking everything uniformly.
- Prefer stacked summaries, progressive disclosure, card views, or drill-in
  patterns before forcing dense desktop layouts onto mobile.
- Use horizontal scrolling for dense data only when the structure truly
  requires it, and keep labels or context understandable when scrolling.
- Keep chart labels, legends, and summaries readable on small screens, and
  provide a non-hover path to key values.

## Architecture Placement

- Put DTO-to-series mapping, filters, grouping, sorting, and drill state in
  `app/lib/client/usecase/<feature>/`, and keep chart components focused on
  rendering, theming, and accessibility wiring.
- Keep shared chart shells, wrappers, and common chart presentation primitives
  in `app/components/shared/`, and keep feature-specific chart views near the
  owning feature component.
- Treat chart library adapters as client-side UI infrastructure only when they
  need a reusable wrapper or abstraction boundary.
- Keep responsive layout behavior in component-scoped styles — the component's
  own CSS Module or the component library's styling solution — and in
  presentational components by default. Do not introduce global media queries
  for feature-specific layout.
- Move viewport-aware state into `app/lib/client/usecase/<feature>/` only when
  it affects interaction flow, data loading, or feature behavior rather than
  pure presentation.
- Keep device detection and browser capability checks in client infrastructure
  when they are truly required.

## Verification

Verify rendered charts and responsive layouts in a real browser per
[`playwright-ui-verification.md`](playwright-ui-verification.md) and
[`verification-gates.md`](verification-gates.md). For this guidance,
specifically confirm:

- at least one desktop viewport and one narrow mobile viewport for
  UI-affecting changes, and both portrait and landscape when orientation
  matters
- no accidental horizontal scrolling, clipped primary action, unreadable text,
  or hidden validation on the touched flow
- Tooltip, Popover, chart, table, and dialog behavior on touch-sized layouts,
  not just desktop
- chart labels, legend clarity, summary text, and a non-hover path to key
  values
