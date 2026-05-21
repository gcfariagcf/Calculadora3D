# Calculadora 3D — Implementation Plan

**Target:** Single `index.html` file  
**External CDN:** html2pdf.js only  
**Spec reference:** SPEC.md  

---

## Phase 1 — HTML Structure

**Goal:** Semantic skeleton with all sections, inputs, and result containers in place. No styling or JS yet.

Tasks:
- [ ] `<head>`: charset, viewport, title ("Calculadora 3D"), CDN link for html2pdf.js
- [ ] `<body>` top-level structure:
  - `#ad-top` — top banner placeholder div
  - `<header>` — title, language toggle button, brand line
  - `<main>` — two-column wrapper: `#content` + `#sidebar`
  - `#ad-sidebar` — sidebar ad placeholder div inside `#sidebar`
  - `<footer>` — "Powered by Duo3DLab"
- [ ] Inside `#content`, the form `<form id="calc-form">` with five `<section>` elements:
  - `#s-print` — Print Parameters (weight, print_h, print_min, filament_price)
  - `#s-machine` — Machine & Operations (working_h, working_days, labor_rate, elec_rate, printer_price, failure_rate[default=10])
  - `#s-postproc` — Post-Processing (postproc_hours)
  - `#s-packing` — Accessories & Packing (accessories_cost, piece_h, piece_w, piece_l, read-only box display)
  - `#s-pricing` — Pricing (margin, selling_price)
- [ ] `<button id="btn-calculate">` below the form
- [ ] `#results` div (hidden by default) containing five result cards:
  - `#res-breakdown` — cost breakdown table
  - `#res-suggested` — suggested price + search buttons + reference URL input
  - `#res-capacity` — production capacity
  - `#res-revenue` — revenue projection
  - `#res-breakeven` — break-even analysis
- [ ] `<button id="btn-export">` inside `#results` (hidden until calculation runs)
- [ ] All text-bearing elements carry a `data-i18n="key"` attribute for language binding

**Deliverable:** Valid HTML that renders a bare but complete page structure.

---

## Phase 2 — CSS Styling

**Goal:** Modern, clean visual design that auto-switches dark/light and is responsive.

Tasks:
- [ ] CSS custom properties on `:root` (light palette) and `@media (prefers-color-scheme: dark)` override (dark palette) — see SPEC §10
- [ ] Base reset: `box-sizing: border-box`, font stack (`system-ui, sans-serif`), smooth scroll
- [ ] Layout:
  - Top banner: full-width, 90px min-height, centered content
  - `<header>`: flex row, space-between, sticky top
  - `<main>`: CSS Grid — `1fr 300px` columns on ≥ 900px, single column below
  - `#ad-sidebar`: sticky `top: 0`, height fits content
  - `<footer>`: centered, muted text
- [ ] Form sections: card style (background `--surface`, border-radius 8px, box-shadow subtle, padding 1.5rem, margin-bottom 1rem)
- [ ] Section headings: colored left-border accent using `--primary`
- [ ] Inputs: full width, border `--border`, border-radius 6px, padding 0.5rem, focus ring in `--primary`
- [ ] Input error state: border red, `--error` color text below field
- [ ] Calculate button: full width, background `--primary`, bold, large, hover darken
- [ ] Result cards: same card style as form sections, initially `display: none`
- [ ] Cost breakdown table: striped rows, bold total row
- [ ] Search buttons (Google/Etsy/MakerWorld): inline pill buttons with brand-ish colors
- [ ] Export PDF button: outlined style, aligned right
- [ ] Language toggle: two inline text buttons, active state underlined/bold
- [ ] Ad placeholders (development): dashed border `--border`, `background: --surface`, centered label "Advertisement"
- [ ] Responsive: on < 900px sidebar moves below content; top banner max-width 100%

**Deliverable:** Styled, responsive page that looks correct in both dark and light mode with placeholder content.

---

## Phase 3 — i18n (Bilingual EN/PT)

**Goal:** All visible strings togglable between English and Portuguese without page reload.

Tasks:
- [ ] Define `const STRINGS = { en: {...}, pt: {...} }` JS object covering every `data-i18n` key (see SPEC §9 for the full list — expand to cover all field labels, placeholders, error messages, result labels, button text, tooltips)
- [ ] `let currentLang = detectLang()` — reads `navigator.language`, defaults to `'pt'` if it starts with `pt`, else `'en'`
- [ ] `function applyLang(lang)`:
  - Iterates all `[data-i18n]` elements, sets `textContent` or `placeholder` from `STRINGS[lang]`
  - Updates `document.documentElement.lang`
  - Stores active lang in a module-level variable
  - Updates toggle button active state
- [ ] Language toggle button click handler calls `applyLang()`
- [ ] `applyLang()` also re-renders any currently-displayed results (re-calls result rendering with stored last result data)
- [ ] Call `applyLang(currentLang)` on page load

**Deliverable:** Page language switches instantly on button click; all labels, placeholders, and results update.

---

## Phase 4 — Box Size Auto-Selection

**Goal:** Given piece H, W, L, select smallest fitting predefined box and return its cost.

Tasks:
- [ ] Define box data:
  ```js
  const BOXES = [
    { name: 'XS', dims: [10,10,10], cost: 1 },
    { name: 'S',  dims: [20,15,10], cost: 2 },
    { name: 'M',  dims: [30,20,15], cost: 3 },
    { name: 'L',  dims: [40,30,20], cost: 4 },
    { name: 'XL', dims: [50,40,30], cost: 5 },
  ]
  ```
- [ ] `function selectBox(h, w, l)`:
  - Sort piece dims descending: `piece = [max, mid, min]`
  - For each box (smallest first), sort box dims descending
  - Return first box where `piece[0] ≤ box[0] && piece[1] ≤ box[1] && piece[2] ≤ box[2]`
  - Return `null` if none fit (triggers manual cost override)
- [ ] In the packing section: show auto-selected box name and cost as read-only text below dimension inputs, updated on Calculate
- [ ] If `selectBox` returns `null`: show warning string (i18n key `warn_box_overflow`) and reveal a manual packing cost input `#packing-override`

**Deliverable:** Box is selected and displayed correctly for any piece dimensions entered.

---

## Phase 5 — Core Calculations

**Goal:** Implement all formulas from SPEC §5 and produce a result data object.

Tasks:
- [ ] `function readInputs()` — reads and parses all form values, returns a plain object; returns `null` if validation fails (see Phase 6 for validation)
- [ ] `function calculate(inputs)` — pure function, takes the inputs object, returns a `results` object with every derived value:
  ```
  print_time_h, filament_adj, electricity_adj, depreciation_adj,
  printing_labor_adj, post_proc_cost, packing_cost, box_name,
  total_cost, suggested_price, pieces_per_day, pieces_per_month,
  daily_revenue, monthly_revenue, monthly_profit, actual_margin,
  fixed_monthly_costs, variable_cost_per_piece, contribution_margin,
  break_even_units, break_even_days, depreciation_per_hour
  ```
- [ ] Handle all edge cases:
  - `failure_rate = 100` → error
  - `profit_margin ≥ 100` → error
  - `print_time_h > working_hours` → `pieces_per_day = 0`, flag `warn_no_capacity`
  - `contribution_margin ≤ 0` → flag `warn_no_breakeven`
  - `selectBox` returns null → flag `warn_box_overflow`, use manual packing cost
- [ ] Store last result in module-level `let lastResult = null` for re-render on lang switch
- [ ] Calculate button click handler: `readInputs()` → `calculate()` → `renderResults()`

**Deliverable:** Clicking Calculate produces a correct result object for any valid input set.

---

## Phase 6 — Input Validation

**Goal:** Prevent calculation with missing or invalid inputs; show inline errors.

Tasks:
- [ ] `function validateInputs(raw)` — checks each field:
  - Required fields (weight, filament_price, working_h, working_days, labor_rate, elec_rate, selling_price): must be present and > 0
  - print_h + print_min: combined print time must be > 0
  - print_min: 0–59
  - failure_rate: 0–99
  - profit_margin: 0–99
  - piece dims: all > 0 if any one is filled (or all three required)
- [ ] For each error: add CSS error class to input, insert `<span class="field-error" data-i18n="...">` below it
- [ ] `clearErrors()` removes all error states before each validation pass
- [ ] Scroll to first error field if errors exist
- [ ] Return `true` if valid, `false` if not

**Deliverable:** Invalid inputs are clearly marked; Calculate does not proceed until form is valid.

---

## Phase 7 — Results Rendering

**Goal:** Populate all result cards with formatted values and show them.

Tasks:
- [ ] `function renderResults(results, lang)`:
  - Show `#results` div (remove `display:none`)
  - **Cost Breakdown** (`#res-breakdown`): populate table rows with formatted dollar amounts; highlight total row
  - **Suggested Price** (`#res-suggested`): set suggested price text; update search button hrefs with piece weight interpolated into query strings
  - **Production Capacity** (`#res-capacity`): set pieces/day and pieces/month
  - **Revenue Projection** (`#res-revenue`): set all revenue/profit fields; color monthly_profit green if positive, red if negative
  - **Break-Even** (`#res-breakeven`): set all break-even fields; if `warn_no_breakeven` flag, replace content with error message
  - Show `#btn-export`
- [ ] `function fmt(value, lang)` — formats numbers with 2 decimal places using `toLocaleString` with active locale
- [ ] Scroll page to `#results` smoothly after render
- [ ] Show depreciation per hour as read-only note in Section 2 after calculation

**Deliverable:** All result cards display correct, locale-formatted values after Calculate is pressed.

---

## Phase 8 — Internet Price Reference

**Goal:** Let users search for market prices and save a reference URL.

Tasks:
- [ ] In `#res-suggested`, three buttons with `target="_blank"` and `rel="noopener"`:
  - **Google**: href built as `https://www.google.com/search?q=3d+print+cost+{weight}g` (weight interpolated at render time)
  - **Etsy**: href built as `https://www.etsy.com/search?q=3d+printed+{weight}g`
  - **MakerWorld**: href fixed to `https://makerworld.com/en/search/models?keyword=3d+print`
- [ ] Reference URL text input (`#ref-url-input`) below buttons
- [ ] On `input` event of `#ref-url-input`: if non-empty, render a `<a href="..." target="_blank">` link below; if empty, hide the link
- [ ] No URL validation (user knows what they're pasting)

**Deliverable:** Users can open market searches and record a reference URL in the results area.

---

## Phase 9 — PDF Export

**Goal:** Export the Results section as a clean, dated PDF.

Tasks:
- [ ] Load html2pdf.js from CDN in `<head>` (defer):
  ```html
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js" defer></script>
  ```
- [ ] `#btn-export` click handler:
  ```js
  const date = new Date().toISOString().slice(0,10)
  html2pdf()
    .set({
      margin: 10,
      filename: `calculadora3d-${date}.pdf`,
      html2canvas: { scale: 2 },
      jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
    })
    .from(document.getElementById('results'))
    .save()
  ```
- [ ] Hide `#btn-export`, `#ad-top`, `#ad-sidebar`, and search buttons from the PDF using a `print-hide` CSS class and applying it before export, then removing after
- [ ] Add page header to PDF via html2pdf's `before-page` option (or a hidden-except-print `#pdf-header` div with brand name and date)

**Deliverable:** Clicking Export PDF downloads a well-formatted A4 PDF of the results.

---

## Phase 10 — Ad Space Setup & Final Polish

**Goal:** Production-ready ad slots, accessibility, and final QA.

Tasks:
- [ ] Add HTML comments inside `#ad-top` and `#ad-sidebar` explaining where to paste ad network code
- [ ] Remove development placeholder border/label from ad divs (or keep as an opt-in `data-dev` toggle)
- [ ] Add `aria-label` to all icon-only buttons
- [ ] Add `<label for="...">` correctly linked to every input
- [ ] Add `title` attributes to inputs that benefit from a tooltip (e.g., failure rate, depreciation formula)
- [ ] Test on Chrome, Firefox, Edge (latest)
- [ ] Test dark mode (toggle OS preference)
- [ ] Test language switch with results visible
- [ ] Test PDF export in both languages
- [ ] Test responsive layout at 375px, 768px, 1280px breakpoints
- [ ] Verify all calculations against hand-calculated examples
- [ ] Minify inline CSS and JS (optional, single-file goal)

**Deliverable:** Shippable `index.html` ready to upload to any static host.

---

## Phase Summary

| Phase | Focus | Output |
|-------|-------|--------|
| 1 | HTML structure | Complete skeleton |
| 2 | CSS + theme | Styled, responsive, dark/light |
| 3 | i18n | EN/PT language toggle |
| 4 | Box selection | Auto packing cost |
| 5 | Calculations | Core result object |
| 6 | Validation | Error states on inputs |
| 7 | Result rendering | All result cards populated |
| 8 | Price reference | Search links + URL field |
| 9 | PDF export | Downloadable results PDF |
| 10 | Polish + QA | Ship-ready file |

---

## File Deliverable

```
index.html        ← single file, everything inlined
SPEC.md           ← this project's specification
PLAN.md           ← this document
```

No build step. No package manager. Deploy by copying `index.html` to any static host (GitHub Pages, Netlify, Vercel, or a plain web server).
