# Feasibility Module (files: `FeasibilityManager.jsx`, `FeasibilitySettings.jsx`, `FeasibilityCalculation.jsx`, `FeasibilitySummary.jsx`)

## Purpose

Estimate the **residual land value** and other financial KPIs for both _Medium-Density_ and _High-Density_ development scenarios on a selected site. The module draws on:

- Construction-certificate analytics (median costs & dwelling sizes)
- Recent sales data (median prices)
- Live planning controls (FSR / HOB) and optional **Low-Mid-Rise (LMR)** bonuses
- User-specified financial assumptions (interest, profit & risk, marketing, etc.)

---

## Component Overview

| React Component                | Responsibility                                                                                                                                                  |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **FeasibilityManager.jsx**     | Top-level modal, orchestrates data fetch, initial calculations and hosts `<FeasibilitySettings>` inside a modal frame.                                          |
| **FeasibilitySettings.jsx**    | Two-column settings grid, social/affordable housing sliders, custom control toggle, LMR eligibility checker, tabs: _Settings / Calculation / Summary_.          |
| **FeasibilityCalculation.jsx** | Detailed line-item view for Medium- or High-density (and LMR variants) with breakeven charts, yield tables, Excel export button and optional 3-D massing hooks. |
| **FeasibilitySummary.jsx**     | Dashboard cards + animated Recharts comparison of residual land value & yield, cost- and revenue-sensitivity tornado charts, radar chart of key metrics.        |

Support files:

- `utils/lmrPermissibility.js` – returns FSR/HOB ranges for LMR types.
- `utils/map/utils/councilLgaMapping.js` – LGA ↔ council lookup used when fetching construction-certificate data.

---

## High-Level Workflow

```
User opens modal →
  fetchConstructionData()  ⟶ NSW ePlanning OnlineCC API
  • median cost per m² (medium & high)
  • median dwelling size (by bedroom count)
  fetchSalesData()         ⟶ Supabase function (pre-loaded in parent)
  fetchLMRData()           ⟶ utils/lmrPermissibility
  deriveMedianPrices()     (2-bed filter for high density)
  initialiseSettings()
  user edits settings ➜   onSettingChange()
  useEffect ─► calculateFeasibility() for:
        • current controls (medium & high)
        • LMR controls    (if eligible & toggled)
  results ➜ FeasibilitySummary & Calculation tabs
```

### Key Algorithm (`calculateFeasibility` in both Manager & Settings)

1.  Determine **site area** and optional **developable area** (via `@turf/area`).
2.  Pick control set:
    - Current LEP controls (FSR & HOB).
    - _or_ LMR bonus (best option per density).
    - or **Custom** controls typed by the user.
3.  Compute maximum GFA under FSR and under HOB; choose the lower (unless one is zero).
4.  Derive NSA using `gfaToNsaRatio`, then **development yield** using `dwellingSize`.
5.  Calculate **Gross Realisation** (yield × median price).
6.  Deduct GST + selling costs → _Net Realisation_.
7.  Deduct _Profit & Risk_ margin → _Net after P&R_.
8.  Add **Construction cost** (custom or median) + DA fees + professional fees + contributions.
9.  Add **Finance cost** (interest over half project period).
10. Add **Land Tax** (threshold rules coded inline).
11. Residual Land Value = Net after P&R − all costs.

The same function is reused for LMR scenarios by toggling `useLMR` and passing `lmrOption` (Dual Occupancy, RFB, etc.).

---

## Settings Inventory (per density)

| Setting id                  | Default                                              | Notes                                                  |
| --------------------------- | ---------------------------------------------------- | ------------------------------------------------------ |
| `siteEfficiencyRatio`       | 0.70 (medium) / 0.60 (high)                          | building footprint share of developable area           |
| `floorToFloorHeight`        | 3.1 m                                                | ceiling height for HOB→storey calc                     |
| `gbaToGfaRatio`             | 0.90 / 0.75                                          | efficiency from building footprint to GFA              |
| `gfaToNsaRatio`             | 0.82 (both)                                          | GFA → Net Saleable Area                                |
| `assumedConstructionCostM2` | rounded median from OnlineCC                         | editable override                                      |
| `assumedDwellingSize`       | median (medium) / 75 m² fixed (high)                 |                                                        |
| `dwellingPrice`             | median sale price                                    | editable, revert icon appears if overridden            |
| `minimumLotSize`            | 200 m²                                               | only used for Medium-density fallback when FSR=0 HOB=0 |
| Financial inputs            | commission, marketing, profit & risk, interest, etc. | percentages stored as decimals                         |

LMR extra toggles: `useLMR` (bool) + `selectedOptions.{medium|high}` containing `{ type, fsrRange, hobRange, potentialFSR, potentialHOB }`.

---

## External Data Calls

| Endpoint                                       | Reason                                           |
| ---------------------------------------------- | ------------------------------------------------ |
| `OnlineCC` (NSW ePlanning)                     | construction cost & dwelling-size medians by LGA |
| Supabase `public.sales` view (via parent prop) | recent dwelling sales                            |

All requests are funnelled through the **Cloudflare Worker** proxy to avoid CORS & hide API keys.

---

## Tabs & Modal Structure

```
┌─ Modal ──────────────────────────────────────────────────────────┐
│  Settings | Calculation | Summary                               │
│ ▸ Settings tab                                                  │
│   ├── Property Information (zone, FSR, HOB, custom controls)    │
│   ├── LMR eligibility & option selectors                        │
│   └── Two-column grid of inputs (medium vs high)                │
│ ▸ Calculation tab                                               │
│   ├── Sub-tabs: medium | high | (optional) high-LMR            │
│   └── Each shows FeasibilityCalculation table + breakeven chart │
│ ▸ Summary tab                                                   │
│     FeasibilitySummary with animated charts                     │
└──────────────────────────────────────────────────────────────────┘
```

---

## Error & Loading Handling

- Per-density **loadingStatus** flags for `constructionCosts` & `dwellingSizes` enable fine-grained spinners.
- Fetch errors populate `constructionData.error` and surface as red banners.

---

## Integration Points

- **DevelopmentModal** – supplies developable area & zoning controls.
- **PPTX Generator** – `ReportGenerator.jsx` queries the Feasibility context to fill slide templates _Development Summary_ and _Feasibility Tables_.
- **Valuation Module** – high-density residual value can flow into the Comparable Sales workflow (future roadmap).

---

## File Sizes

| File                         | Lines  |
| ---------------------------- | ------ |
| `FeasibilityManager.jsx`     | ~1 100 |
| `FeasibilitySettings.jsx`    | ~2 100 |
| `FeasibilityCalculation.jsx` | ~1 300 |
| `FeasibilitySummary.jsx`     | ~1 250 |

---

This document reflects code as of the current commit. Any future code changes should be mirrored here to keep the reference accurate.
