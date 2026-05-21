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
| Logo | Fetch from Duo3DLab's Instagram profile — [https://www.instagram.com/duo3dlab/](https://www.instagram.com/duo3dlab/). Display the profile image in the header next to the page title. If the image cannot be loaded, fall back to text only. |
| Languages | English (EN) / Portuguese (PT) |
| Default language | Auto-detected from `navigator.language` (pt → PT, else EN) |
| Currency | $ (no label suffix) |
| Color theme | CSS `prefers-color-scheme` auto dark/light |
| Delivery | Single `.html` file, CDN-only external dependencies |

---

## 3. Layout

```
┌──────────────────────────────────────────────────────┐
│  AD BANNER — full width, 90px height                 │
├─────────────────────────────────┬────────────────────┤
│  HEADER                         │                    │
│  [Calculadora 3D]  [EN | PT]    │  AD SIDEBAR        │
│  Powered by Duo3DLab            │  300px wide        │
├─────────────────────────────────┤  sticky            │
│  FORM                           │                    │
│  ▸ Section 1: Print Parameters  │                    │
│  ▸ Section 2: Machine & Ops     │                    │
│  ▸ Section 3: Post-Processing   │                    │
│  ▸ Section 4: Accessories &     │                    │
│               Packing           │                    │
│  ▸ Section 5: Pricing           │                    │
│                                 │                    │
│  [  CALCULATE  ]                │                    │
│                                 │                    │
│  RESULTS (shown after calc)     │                    │
│  ▸ Cost Breakdown               │                    │
│  ▸ Suggested Price & Links      │                    │
│  ▸ Production Capacity          │                    │
│  ▸ Revenue Projection           │                    │
│  ▸ Break-Even Analysis          │                    │
│                                 │                    │
│  [  EXPORT PDF  ]               │                    │
├─────────────────────────────────┴────────────────────┤
│  FOOTER: Powered by Duo3DLab                         │
└──────────────────────────────────────────────────────┘
```

**Responsive behaviour:** On screens < 900px the sidebar collapses below the main content. The top banner becomes a responsive unit (max 100% width).

---

## 4. Input Fields

### Section 1 — Print Parameters

| Field | Type | Unit | Validation | Notes |
|-------|------|------|------------|-------|
| Piece weight | number | g | > 0 | Weight of one printed piece |
| Print time — hours | number | h | ≥ 0, integer | |
| Print time — minutes | number | min | 0–59, integer | At least one of h/min must be > 0 |
| Filament price | number | $/kg | > 0 | |

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
| Post-processing time per unit | number | h | ≥ 0 |

### Section 4 — Accessories & Packing

| Field | Type | Unit | Validation | Notes |
|-------|------|------|------------|-------|
| Accessories cost per unit | number | $ | ≥ 0 | Single total |
| Piece height | number | cm | > 0 | Used for box auto-selection |
| Piece width | number | cm | > 0 | |
| Piece length | number | cm | > 0 | |

**Predefined box sizes (read-only reference, auto-selected):**

| Size | Dimensions (cm) | Cost |
|------|----------------|------|
| XS | 10 × 10 × 10 | $1 |
| S | 20 × 15 × 10 | $2 |
| M | 30 × 20 × 15 | $3 |
| L | 40 × 30 × 20 | $4 |
| XL | 50 × 40 × 30 | $5 |

**Box selection logic:** Sort piece dimensions (H, W, L) descending → compare against each box's sorted dimensions ascending. Select the smallest box where all three piece dimensions fit. If the piece exceeds the XL box, show a warning: *"Piece exceeds the largest predefined box. Enter packing cost manually."* and add a manual override field.

### Section 5 — Pricing

| Field | Type | Unit | Validation | Notes |
|-------|------|------|------------|-------|
| Desired profit margin | number | % | 0–99 | Used to compute suggested price |
| Your selling price | number | $ | > 0 | Used for revenue & break-even |

---

## 5. Calculations

### 5.1 Derived values

```
print_time_h     = print_hours + print_minutes / 60
filament_cost    = (weight_g / 1000) × filament_price_per_kg
electricity_cost = print_time_h × electricity_cost_per_hour
depreciation_per_hour  = printer_price / (3 × 12 × working_days × working_hours)
depreciation_per_piece = depreciation_per_hour × print_time_h
printing_labor   = print_time_h × labor_cost_per_hour
post_proc_cost   = post_processing_hours × labor_cost_per_hour
failure_factor   = 1 / (1 − failure_rate / 100)
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

```
suggested_price = total_cost / (1 − profit_margin / 100)
```

Edge case: if `profit_margin = 100`, show error *"Margin cannot be 100%."*

### 5.5 Production capacity

```
pieces_per_day   = floor(working_hours × 60 / (print_time_h × 60))
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

| Slot | ID | Position | Standard size |
|------|----|----------|---------------|
| Top banner | `ad-top` | Full-width, above header | 728 × 90 (leaderboard) |
| Sidebar | `ad-sidebar` | Right column, sticky | 300 × 250 or 300 × 600 |

Both slots are empty `<div>` elements with comments marking where ad network scripts should be inserted. They are styled with a visible placeholder (dashed border + label) when empty, so the layout is visible during development.

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
| `lbl_margin` | Profit margin (%) | Margem de lucro (%) |
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
- If piece dimensions exceed XL box: display packing warning and show manual cost override field

---

## 12. Out of Scope (v1.0)

- Multiple printers
- Saving/loading sessions
- User accounts
- Backend or database
- Email sending
- Real-time internet price lookup
- Multi-currency conversion
