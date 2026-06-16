# VM Distro Tools — Beta Notes & Refinements Log

> Last updated: 2026-06-05. Owner returns 2026-06-15.

## What's live today

**Site:** https://j6d5s84s6w-eng.github.io/VM-Distro/

Two tools shipped:
- **Signage Distro** — existing, unchanged.
- **Demo Distro** — rebuilt 2026-06-04 → 2026-06-05 to output a wide-format
  workbook that matches the WWRFO forecast shape (Grand Total with Buffer,
  Buffer, Product Zone Tables & Walls, Equipment Lists, Accessories Fixtures,
  Services and Avenues + TOTAL row at the top of the sheet).

## Known discrepancies (vs. real WWRFO)

These are intentional shortcuts for the beta. None are bugs — they're scope
decisions made because we don't currently have the filled-in Input Template
or the WWRFO calc spec.

1. **Cell values are illustrative, not the exact WWRFO calc.**
   Per-product/color cells are computed as `fixture_count × cohort_units ×
   deterministic_presence_flag`. The layout is correct; the numbers won't
   match what WWRFO produces. Safe for "what does the output look like"
   stakeholder previews. NOT safe for real merchandising decisions.

2. **116-column structure is hardcoded from the SP26 iPad reference.**
   Running the tool for iPhone / Mac / Watch resets will still produce the
   iPad-shaped layout (J707/J737 product columns, iPad Asst Table /
   iPad Comp Table fixture bands, etc.). To generalize, the engine needs
   to derive structure from the Input Template's `New Products` and
   `Per-Fixture Quantities` sheets rather than embedding it.

3. **Fixture-name matching is fuzzy but not magic.**
   Lookups are case- and whitespace-tolerant (`iPad Asst Table` matches
   `iPad assortment table`), but if a fixture name in the input isn't
   close to what the engine expects, that label cell and its per-product
   children all go to 0.

4. **Cohort UI no longer reflects the real cohort logic.**
   The cohorts list in Section 4 now drives per-product cell magnitude
   (multiplier on the fixture count). The original "table size cohort →
   units per table" math is gone. Keep the UI for now — it's the only
   knob users have for cell scale — but the labels and the help text
   are out of sync with what it does.

5. **In-Store Buffer field was removed.** Used to be a flat add. The
   new engine doesn't have a place for it.

6. **Fixture Fill file is now optional.** Generation still works
   without it; the Avenue Assignments sheet is skipped from the output.

## Refinement list (priority order)

### P0 — Required before "beta" → "ready"

- [ ] **Get the filled-in Input Template** from the WWRFO team (or a math
      spec / formula doc / Quip page). Without this we can't build the
      real engine — only shape mimics.
- [ ] **Resolve the Mandy issue end-to-end.** The fill file is now
      optional, but verify with her that the rest of her workflow now
      generates correctly with the new wide-format output.
- [ ] **Cross-LOB generalization.** Engine should derive products,
      colors, and fixture types from the Input Template inputs, not from
      a hardcoded SP26 iPad literal. Without this, the tool is iPad-only.

### P1 — UX gaps that hurt usability

- [ ] **Preview before download.** Show the user a small in-browser table
      of what they're about to generate (first 5 rows + column headers)
      so they can sanity-check before saving the xlsx.
- [ ] **Input validation on upload.** When a file is dropped, parse it
      and warn if expected columns / sheets are missing (e.g. "your WW
      Fixture Counts tab doesn't have an `iPad Asst Table` column —
      that section will be empty in the output").
- [ ] **Cohort UI clarity.** Either rename / re-purpose it so users
      understand what it actually does in the new engine, or remove it
      entirely if the wide-format engine doesn't really need it.
- [ ] **Output formatting polish.** Freeze panes (header rows + first 4
      cols), bold TOTAL row, conditional formatting on cells where qty
      ≥ a threshold, color-banding by section.

### P2 — Nice-to-haves

- [ ] **Sustain support.** The current Demo Distro only handles NPI.
      Add the equivalent for sustain (analogous to the Kits Merge
      project's sustain path).
- [ ] **Programs / Program Ratios overrides.** The Input Template has
      a Programs sheet with per-store overrides (Today at Apple
      Highlights, Boardrooms, Flagship+, etc.). Currently ignored.
- [ ] **Per-section toggles in Section 4.** Let user enable / disable
      Equipment Lists, Accessories Fixtures, Services and Avenues
      sections per run.
- [ ] **History / re-run.** Remember the last-used filenames and field
      values across page reloads (localStorage).

## Recent commit log (Demo Distro changes)

```
3c95595  Demo Distro: rebuild output engine to match WWRFO wide format
4d0b167  Demo Distro: dynamic cohorts, free-form table sizes, optional fill file
bb4a04d  Merge Input Template into Demo Distro, remove standalone Recipe tab
9af734f  Add Recipe Distro tool with Input Template download
8d52c57  Add Demo Distro tool and activate on home page
0a3f58d  Initial build: VM Distro Tools site with 2x3 Signage tool
```

## Pressure-test feedback received so far

- **Mandy (2026-06-04):** Fixture Fill file is not applicable for her
  workflow. Generate button was greyed out / showed ✕ cursor because the
  readiness check required all 3 file uploads. → Fixed in `4d0b167`:
  fill file is now optional, generation works without it, output skips
  the Avenue Assignments sheet when absent.

## Where to file new feedback

For now: ping the owner directly. When she's back 2026-06-15, decide
whether to set up a GitHub issues template, a Quip page, or a Slack
channel for ongoing feedback.

---

## 2026-06-16 update — pressure-test response

### Shipped today
- Mode → T Minus in the Input Template's Reset sheet.
- Removed Avenue Assignments + Reference tabs (the engine never reads
  them; they'll come back when the real WWRFO engine is wired up).
- Per-Fixture Quantities range handling: deferred (user wants to revisit).
- Avenue update flow: deferred (user wants to revisit).
- Programs: documented where to add new programs (Input Template →
  Programs sheet, one row per (program, store)).
- Fixture Counts purpose: explained — it's the one sheet the engine
  actually reads today.
- Section 4 — WW Fixture Counts tab is now a dropdown auto-populated
  from the uploaded fixture file.
- Section 4 — Buffer % replaced with editable Buffer Rules list.
  Defaults: 7-8ft +2/color, 10ft +3/color, 12+ +3/color, Watch +5/table.
- Section 4 — Products list (codename + colors) added so per-NPI product
  names are editable. Buffer Rules Product dropdown derives from it.
- Upload zone "Fixture Fill file" → "Accessories GSL".
- Option A wiring: when the Filled Input Template is uploaded, the site
  reads its `Reset` (Date suffix → Forecast Period, Reset name → output
  filename hint) and `New Products` (codes + colors → Products list)
  sheets and pre-fills Section 4. Single source of truth: the xlsx.
- Trace sheet added to every generated output xlsx. Documents per-column
  source (which input it pulls from) and formula. Lets users audit cells
  without reading code.

### Audit — where else there is still hardcoding (item 7 sweep)

Things that should eventually be flexible but currently are not:

1. **Output column structure (116 cols)** — REF_COLUMNS literal in
   `demo/index.html` hardcodes the SP26 iPad section bands, fixture-type
   sub-bands, and product columns. Changing the Products list changes
   the math but not the *column headers* in the output. To make headers
   fully dynamic (M512 Pacific instead of J707 Purple), the engine needs
   to *derive* the column layout from the Input Template's New Products
   + Per-Fixture Quantities sheets, not embed it. Big rewrite, ~2 days.
2. **Watch detection** — engine routes "Watch" buffer based on whether
   the product codename contains "watch" (case-insensitive). If your
   team uses a non-obvious codename for Watch in a future reset, the
   special-case logic stops triggering. Fix: add a Type field to the
   Products list (Phone/Pad/Watch/Mac/etc.) instead of name-sniffing.
3. **Cohort units math** — illustrative formula `fixture_count ×
   cohort_units × presence(product, color)` with a deterministic 60%
   presence rule. Real WWRFO uses per-fixture per-product quantities
   from the Input Template's `Per-Fixture Quantities` sheet. Needs the
   filled template + math spec to wire properly.
4. **Section names** — "Grand Total with Buffer", "Buffer", "Product
   Zone Tables & Walls", "Equipment Lists", "Accessories Fixtures",
   "Services and Avenues" are hardcoded from the iPad reference. If
   future LOBs have different section taxonomies, this needs to come
   from a config sheet in the Input Template.
5. **Fixture name vocabulary** — buffer rule "Applies to" parses
   fixture names by case+whitespace-tolerant substring match against the
   input row's column names. If the WW Fixture Counts file uses
   different naming conventions (abbreviations, codes), the lookup
   silently returns 0. Fix: surface unmatched fixture names in a
   pre-generation validation warning.
6. **Default Buffer Rules** — ship pre-populated with iPad-specific
   defaults (J707/J737 7-8ft / 10ft / 12+, Watch). If a user clones the
   tool for iPhone they have to manually clear and re-add. Fix: have
   defaults derive from the uploaded Input Template's Reset sheet LOB
   field, with per-LOB rule presets.
7. **Output filename pattern** — engine appends `.xlsx` to whatever's
   in the Output File Name input. No template support for `{period}` /
   `{lob}` placeholders. Minor.
8. **Output sheet name** — always "Demo Forecast". Could be parameterized
   per LOB. Minor.
