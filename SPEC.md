# Calculadora 3D — Product Specification

**Powered by:** Duo3DLab  
**Version:** 1.0  
**Date:** 2026-05-21  
**Type:** Single HTML file, no backend  

---

## 1. Overview

A browser-based 3D printing cost calculator. Given print parameters, machine setup, and pricing goals, it outputs a full cost breakdown per piece, a suggested selling price, production capacity, revenue projection, and a break-even analysis. The page is bilingual (EN/PT), respects the user's dark/light mode preference, and can export results to PDF.

---

## 2. Page Identity

| Property | Value |
|----------|-------|
| Page title | Calculadora 3D |
| Brand line | Powered by Duo3DLab |
e log| Logo | Fetch from Duo3DLab's Instagram profile — [https://www.instagram.com/duo3dlab/](https://www.instagram.com/duo3dlab/). Display the profile image in the header next to the page title. If the image cannot be loaded, fall back to text only. |
| Languages | English (EN) / Portuguese (PT) |
| Default language | Auto-detected from `navigator.language` (pt → PT, else EN) |
| Currency | $ (no label suffix) |
| Color theme | CSS `prefers-color-scheme` auto dark/light |
| Delivery | Single `.html` file, CDN-only external dependencies |

---

## 3. Layout

### 3.1 Desktop (viewport ≥ 900 px)

```
┌──────────────────────────────────────────────────────┐
│  AD BANNER — full width, max 90px height             │
├─────────────────────────────────┬────────────────────┤
│  HEADER                         │                    │
│  [Logo] Calculadora 3D [EN|PT]  │  AD SIDEBAR        │
│  Powered by Duo3DLab            │  300px wide        │
├─────────────────────────────────┤  sticky            │
│  FORM                           │  • Ana Faria Ateliê│
│  ▸ Section 1: Print Parameters  │  • Nothavel        │
│  ▸ Section 2: Machine & Ops     │  • Panda Juju      │
│  ▸ Section 3: Post-Processing   │                    │
│  ▸ Section 4: Accessories &     │                    │
│               Packing           │                    │
│  ▸ Section 5: Pricing           │                    │
│  [  CALCULATE  ]                │                    │
│  RESULTS (shown after calc)     │                    │
│  [  EXPORT PDF  ]               │                    │
├─────────────────────────────────┴────────────────────┤
│  FOOTER: Powered by Duo3DLab                         │
└──────────────────────────────────────────────────────┘
```

### 3.2 Mobile (viewport < 900 px — Android & iOS)

```
┌─────────────────────────┐
│  AD BANNER (full width) │
├─────────────────────────┤
│  HEADER                 │
│  [Logo] Calculadora 3D  │
│  [EN | PT]              │
├─────────────────────────┤
│  Section 1: Print Params│
├─────────────────────────┤
│  ┌─────────────────┐    │
│  │ Ana Faria Ateliê│    │  ← inline ad 1
│  └─────────────────┘    │
├─────────────────────────┤
│  Section 2: Machine&Ops │
├─────────────────────────┤
│  ┌─────────────────┐    │
│  │ Nothavel        │    │  ← inline ad 2
│  └─────────────────┘    │
├─────────────────────────┤
│  Section 3: Post-Proc   │
├─────────────────────────┤
│  ┌─────────────────┐    │
│  │ Panda Juju      │    │  ← inline ad 3
│  └─────────────────┘    │
├─────────────────────────┤
│  Section 4: Packing     │
│  Section 5: Pricing     │
│  [  CALCULATE  ]        │
│  RESULTS                │
│  [  EXPORT PDF  ]       │
├─────────────────────────┤
│  FOOTER                 │
└─────────────────────────┘
```

**Sidebar hidden on mobile.** The three advertiser logos are instead injected as centred inline blocks between form sections, each linking to the respective Instagram profile.

### 3.3 Responsive rules

| Breakpoint | Behaviour |
|-----------|-----------|
| ≥ 900 px (desktop) | Two-column grid (`1fr 300px`); sidebar visible; inline ads hidden |
| < 900 px (mobile/tablet) | Single column; sidebar hidden; inline ads visible between sections |

- The page is fully usable on **Android** and **iOS** mobile browsers (Chrome, Safari, Firefox).
- Touch targets (buttons, inputs, links) are sized ≥ 44 px tall.
- No horizontal scrolling on any viewport width ≥ 320 px.
- Font sizes remain legible without zooming on a standard phone screen.

---

## 4. Input Fields

### Section 1 — Print Parameters

| Field | Type | Unit | Validation | Default | Notes |
|-------|------|------|------------|---------|-------|
| Piece weight | number | g | > 0 | — | Weight of one printed piece |
| Pieces per printing cycle | number | — | ≥ 1, integer | **1** | How many pieces are printed simultaneously per cycle |
| Print time | number×2 | h / min | hours ≥ 0 integer; minutes 0–59 integer; combined > 0 | — | Single labeled group: two inputs side by side under one "Print time" label, each with an inline unit suffix ("hours" / "minutes") |
| Filament price | number | $/kg | > 0 | — | |

### Section 2 — Machine & Operations

| Field | Type | Unit | Validation | Default |
|-------|------|------|------------|---------|
| Working hours per day | number | h | > 0, ≤ 24 | |
| Working days per month | number | days | > 0, ≤ 31 | |
| Labor cost per hour | number | $ | ≥ 0 | |
| Electricity cost per hour of printing | number | $ | ≥ 0 | |
| Printer purchase price | number | $ | ≥ 0 | Used for depreciation |
| Failure rate | number | % | 0–100 | **10** |

> **Depreciation note:** displayed as read-only below the printer price field after calculation:  
> `Depreciation = printer_price ÷ (3 years × 12 months × working_days × working_hours)`

### Section 3 — Post-Processing

| Field | Type | Unit | Validation |
|-------|------|------|------------|
| Post-processing time per unit | number | **min** | ≥ 0 |

### Section 4 — Accessories & Packing

| Field | Type | Unit | Validation | Default | Notes |
|-------|------|------|------------|---------|-------|
| Accessories cost per unit | number | $ | ≥ 0 | — | Single total |
| Box size | select | — | required | **S** | User selects from predefined list |

**Predefined box sizes (dropdown options):**

| Size | Dimensions (cm) | Cost |
|------|----------------|------|
| XS | 10 × 10 × 10 | $1 |
| S | 20 × 15 × 10 | $2 |
| M | 30 × 20 × 15 | $3 |
| L | 40 × 30 × 20 | $4 |
| XL | 50 × 40 × 30 | $5 |

The dropdown shows size, dimensions, and cost for each option. Selecting a size sets the packing cost directly — no dimension inputs or auto-selection logic required.

### Section 5 — Pricing

| Field | Type | Unit | Validation | Notes |
|-------|------|------|------------|-------|
| Profit margin / Filament | number | % | 0–99 | Margin applied over filament cost (failure-adjusted). Changing this field auto-calculates and fills the selling price field. |
| Profit margin / Accessories | number | % | 0–99 | Margin applied over accessories cost. |
| Your selling price | number | $ | > 0 | Typing here auto-calculates and fills the filament margin field (accessories margin is held fixed). |

The filament margin and the selling price are dynamically linked: editing either one immediately derives and updates the other using the current form values. The accessories margin is independent. The Calculate button is still available for an explicit full recalculation with validation.

---

## 5. Calculations

### 5.1 Derived values

Electricity, machine depreciation, and printing labor are **cycle costs** shared across all pieces in the cycle; filament cost is already per-piece.

```
print_time_h         = print_hours + print_minutes / 60
filament_cost        = (weight_g / 1000) × filament_price_per_kg
electricity_cost     = (print_time_h × electricity_rate) / pieces_per_cycle
depreciation_per_hour= printer_price / (3 × 12 × working_days × working_hours)
depreciation_per_piece = (depreciation_per_hour × print_time_h) / pieces_per_cycle
printing_labor       = (print_time_h × labor_cost_per_hour) / pieces_per_cycle
post_proc_cost       = (post_processing_minutes / 60) × labor_cost_per_hour
failure_factor       = 1 / (1 − failure_rate / 100)
```

### 5.2 Failure rate application

The failure factor is applied only to costs incurred during the print run (material, energy, machine wear, operator supervision during printing). It is **not** applied to post-processing, accessories, or packing, since those costs are only incurred after a successful print.

```
filament_adj    = filament_cost    × failure_factor
electricity_adj = electricity_cost × failure_factor
depreciation_adj= depreciation_per_piece × failure_factor
printing_labor_adj = printing_labor × failure_factor
```

### 5.3 Total cost per piece

```
total_cost = filament_adj + electricity_adj + depreciation_adj
           + printing_labor_adj + post_proc_cost
           + accessories_cost + packing_cost
```

### 5.4 Suggested selling price

Each margin is applied only to its own cost component; all other costs are passed through at face value.

```
filament_component    = filament_adj      / (1 − margin_filament    / 100)
accessories_component = accessories_cost  / (1 − margin_accessories / 100)

suggested_price = filament_component + electricity_adj + depreciation_adj
                + printing_labor_adj + post_proc_cost
                + accessories_component + packing_cost
```

Edge case: if `margin_filament = 100` or `margin_accessories = 100`, show error *"Margin cannot be 100%."*

**Reverse calculation (selling price → filament margin):**
```
margin_filament = (1 − filament_adj / (selling_price − other_costs)) × 100
```
where `other_costs = electricity_adj + depreciation_adj + printing_labor_adj + post_proc_cost + accessories_component + packing_cost`.

### 5.5 Production capacity

```
cycles_per_day   = floor(working_hours / print_time_h)
pieces_per_day   = cycles_per_day × pieces_per_cycle
pieces_per_month = pieces_per_day × working_days
```

Edge case: if `print_time_h > working_hours`, set `pieces_per_day = 0` and show warning.

### 5.6 Revenue projection

Uses the user-defined `selling_price`.

```
daily_revenue    = pieces_per_day   × selling_price
monthly_revenue  = pieces_per_month × selling_price
monthly_cost     = pieces_per_month × total_cost
monthly_profit   = monthly_revenue  − monthly_cost
actual_margin_%  = (monthly_profit / monthly_revenue) × 100   [if monthly_revenue > 0]
```

### 5.7 Break-even analysis

```
fixed_monthly_costs = (working_hours × working_days × labor_cost_per_hour)
                    + (printer_price / 36)

variable_cost_per_piece = filament_adj + electricity_adj + accessories_cost
                        + packing_cost + post_proc_cost

contribution_margin = selling_price − variable_cost_per_piece
break_even_units    = ceil(fixed_monthly_costs / contribution_margin)
break_even_days     = ceil(break_even_units / pieces_per_day)   [if pieces_per_day > 0]
```

Edge case: if `contribution_margin ≤ 0`, show error *"Selling price does not cover variable costs."*

---

## 6. Results Display

Results are hidden on page load and shown only after the Calculate button is pressed. Each sub-section is a distinct card.

### 6.1 Cost Breakdown

Displayed as a table with two columns: item name and amount ($X.XX).

| Line item | Formula source |
|-----------|---------------|
| Filament (failure-adjusted) | `filament_adj` |
| Electricity (failure-adjusted) | `electricity_adj` |
| Machine depreciation (failure-adjusted) | `depreciation_adj` |
| Printing labor (failure-adjusted) | `printing_labor_adj` |
| Post-processing | `post_proc_cost` |
| Accessories | `accessories_cost` |
| Packing (box size label shown) | `packing_cost` |
| **Total cost per piece** | **`total_cost`** |

### 6.2 Suggested Price & Internet Reference

- Display: `Suggested price: $XX.XX` (based on entered margin %)
- Three search buttons (open in new tab):
  - **Google** → `https://www.google.com/search?q=3d+print+cost+[weight]g`
  - **Etsy** → `https://www.etsy.com/search?q=3d+printed+[weight]g`
  - **MakerWorld** → `https://makerworld.com/search?keyword=3d+print`
- A text input field labeled *"Paste reference price URL"* — when filled, renders a clickable hyperlink below the field so the user can revisit their reference.

### 6.3 Production Capacity

| Label | Value |
|-------|-------|
| Pieces per day | `pieces_per_day` |
| Pieces per month | `pieces_per_month` |

### 6.4 Revenue Projection

| Label | Value |
|-------|-------|
| Your selling price | `$selling_price` |
| Daily revenue | `$daily_revenue` |
| Monthly revenue | `$monthly_revenue` |
| Monthly profit | `$monthly_profit` |
| Profit margin | `actual_margin_%` |

### 6.5 Break-Even Analysis

| Label | Value |
|-------|-------|
| Fixed monthly costs | `$fixed_monthly_costs` |
| Variable cost per piece | `$variable_cost_per_piece` |
| Contribution margin per piece | `$contribution_margin` |
| Break-even units/month | `break_even_units` pieces |
| Break-even days | `break_even_days` days |

---

## 7. Ad Spaces

| Slot | ID | Position | Sizing behaviour |
|------|----|----------|-----------------|
| Top banner | `ad-top` | Full-width, above header | Image is constrained to **max-height 90 px**; width scales automatically (`width: auto; max-width: 100%`); vertically centred in the bar |
| Sidebar | `ad-sidebar` | Right column, sticky | Each ad image fills 100 % of the 300 px column; height follows the image's natural aspect ratio |

**Responsive sizing rules:**
- No fixed pixel dimensions are imposed on either slot.
- Images use `width: 100%; height: auto` so they scale fluidly with the available space.
- On narrow viewports (< 900 px) the sidebar collapses below the main content; sidebar images continue to fill the full column width at that size.
- Hover effects (subtle opacity fade) are preserved on both slots.

Each slot contains real linked logo images (not placeholder divs). The top banner links to Duo3DLab's Instagram; the sidebar contains three stacked advertiser logos, each linking to the respective Instagram profile.

---

## 8. PDF Export

- Library: **html2pdf.js** loaded from CDN (`https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/...`)
- Exports the **Results section only** (form and ads are excluded)
- Filename pattern: `calculadora3d-YYYY-MM-DD.pdf`
- Button label: *Export PDF / Exportar PDF*
- Button is shown only after a successful calculation

---

## 9. Language Toggle

- Button in header: `EN | PT`
- Active language is highlighted
- All user-visible strings (labels, placeholders, tooltips, error messages, result labels, button text) have translations defined in a JS dictionary object
- Numbers use `toLocaleString()` with the active locale (`en-US` / `pt-BR`)
- Switching language re-renders all visible text and results without recalculating

### Key string pairs (sample)

| Key | EN | PT |
|-----|----|----|
| `title` | Calculadora 3D | Calculadora 3D |
| `powered_by` | Powered by Duo3DLab | Desenvolvido por Duo3DLab |
| `section_print` | Print Parameters | Parâmetros de Impressão |
| `section_machine` | Machine & Operations | Máquina e Operações |
| `section_postproc` | Post-Processing | Pós-Processamento |
| `section_packing` | Accessories & Packing | Acessórios e Embalagem |
| `section_pricing` | Pricing | Precificação |
| `btn_calculate` | Calculate | Calcular |
| `btn_export` | Export PDF | Exportar PDF |
| `lbl_weight` | Piece weight (g) | Peso da peça (g) |
| `lbl_filament` | Filament price ($/kg) | Preço do filamento ($/kg) |
| `lbl_failure` | Failure rate (%) | Taxa de falha (%) |
| `lbl_print_time` | Print time | Tempo de impressão |
| `lbl_hours` | hours | horas |
| `lbl_minutes` | minutes | minutos |
| `lbl_margin_filament` | Profit margin / Filament (%) | Margem de lucro/filamento (%) |
| `lbl_margin_accessories` | Profit margin / Accessories (%) | Margem de lucro/Acessórios (%) |
| `lbl_selling` | Your selling price ($) | Seu preço de venda ($) |
| `result_total_cost` | Total cost per piece | Custo total por peça |
| `result_suggested` | Suggested price | Preço sugerido |
| `result_pieces_day` | Pieces per day | Peças por dia |
| `result_revenue_month` | Monthly revenue | Receita mensal |
| `result_breakeven` | Break-even units/month | Ponto de equilíbrio (peças/mês) |

---

## 10. Theme

- CSS custom properties set on `:root` for light mode
- `@media (prefers-color-scheme: dark)` overrides them for dark mode
- No manual theme toggle (follows OS/browser preference automatically)

**Light mode palette (reference):**

| Token | Value |
|-------|-------|
| `--bg` | #f5f5f5 |
| `--surface` | #ffffff |
| `--primary` | #1a73e8 |
| `--text` | #212121 |
| `--text-muted` | #757575 |
| `--border` | #e0e0e0 |

**Dark mode palette (reference):**

| Token | Value |
|-------|-------|
| `--bg` | #121212 |
| `--surface` | #1e1e1e |
| `--primary` | #4da3ff |
| `--text` | #e0e0e0 |
| `--text-muted` | #9e9e9e |
| `--border` | #333333 |

---

## 11. Validation & Error Handling

- All required fields highlighted in red on Calculate if empty or invalid
- Error messages appear inline below the offending field
- Results section is not shown if there are validation errors
- If `pieces_per_day = 0`: display warning in capacity section, skip revenue and break-even
- If `contribution_margin ≤ 0`: display error in break-even section
- Packing cost is always resolved from the dropdown selection — no overflow warnings needed

---

## 12. Session Persistence — Machine & Operations

The six fields in the **Machine & Operations** section are saved to `localStorage` automatically and restored on every page load. All other form sections remain ephemeral (cleared on reload).

| Field | localStorage key (inside object) |
|-------|----------------------------------|
| Working hours per day | `working_h` |
| Working days per month | `working_days` |
| Labor cost per hour | `labor_rate` |
| Electricity cost per hour | `electricity_rate` |
| Printer purchase price | `printer_price` |
| Failure rate | `failure_rate` |

**Storage key:** `calc3d_machine` (single JSON object).

**Behaviour:**
- Values are written to `localStorage` on every `input` event in Section 2.
- On `DOMContentLoaded`, stored values are read and applied to the fields before any other init logic.
- If no stored data exists, fields keep their HTML defaults (failure rate = 10, others = 0).
- If `localStorage` is unavailable (e.g. private browsing restrictions), the page degrades silently — no error is shown.

---

## 13. Out of Scope (v1.0)

- Multiple printers
- Saving/loading sessions for sections other than Machine & Operations
- User accounts
- Backend or database
- Email sending
- Real-time internet price lookup
- Multi-currency conversion
