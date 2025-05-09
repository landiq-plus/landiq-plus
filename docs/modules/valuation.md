# Valuation Module (directory: `pptxApp/Utilities/Valuation/`)

## Purpose

Estimate **site market value** by analysing comparable sales and planning potential. The workflow mirrors how valuers prepare a _residual land valuation_:

1. Select the _subject property_ (parcel or developable area)
2. Retrieve a basket of **comparable sales** matching zoning / use / scale
3. Score those comparables on similarity (location, size, zoning, risk)
4. Derive a valuation via **median price per potential GFA** and **median price per m² land area**
5. Present a dashboard of metrics, confidence bands and comparable inspection map

All calculations run client-side; the module can operate fully offline once the comparable dataset is cached.

---

## Directory Map

```
Utilities/Valuation/
├─ ValuationSelector.jsx            – mounts provider + subject/comps/dashboard (≈ 70 lines)
├─ ValuationContext.jsx             – React context & state (≈ 80 lines)
├─ ValuationDashboard.jsx           – main UI & algorithm driver (≈ 2 700 lines)
├─ ComparableSearch.jsx             – UI for search filters (≈ 620 lines)
├─ SubjectPropertySelector.jsx      – dropdown / map-pick for subject (≈ 440 lines)
├─ utils/
│   ├─ valuationAlgorithms.js       – median & IQR calc, dual-method valuation (≈ 190 lines)
│   ├─ comparableScoring.js         – returns 1-3 score based on attribute distance
│   └─ arcgisQueries.js             – helper calls to NSW ESRI services (zone, FSR, HoB, hazards)
└─ components/ (charts, filters)
```

---

## Context Shape (`ValuationContext`)

```ts
interface ValuationState {
  subjectProperty: Feature | null;
  comparables: Feature[]; // raw results from search
  enrichedComparables: Feature[]; // ↑ with extra fields + score
  valuationResult: {
    gfaMethod: {
      estimatedValue: number;
      confidence: number;
      medianPricePerGFA: number;
    };
    areaMethod: {
      estimatedValue: number;
      confidence: number;
      medianPricePerArea: number;
    };
  } | null;
  isSearching: boolean;
  searchComparables(params): Promise<void>;
  flyToProperty(geometry): void;
}
```

All child components consume this via `useValuation()`.

---

## Core Workflow

```mermaid
flowchart TD
  A[SubjectPropertySelector] -->|setSubjectProperty| C(ValuationContext)
  B[ComparableSearch] -->|searchComparables()| C
  C --> D(ValuationDashboard useEffect)
  D -->|enrich comparables| E[caculateComparableScore()]
  E --> F[valuationAlgorithms.estimateValuation()]
  F -->|setValuationResult| C
```

1. **ComparableSearch** builds query params (radius, zoning, date range) and executes `searchComparables` (placeholder awaiting ESRI/PriceFinder integration).
2. **ValuationDashboard**:
   - enriches each comparable with planning/hazard attributes (ArcGIS REST) and distance metrics (Turf)
   - calls `calculateComparableScore` to rate 1 – 3 (colour chips in table)
   - passes the filtered set to `estimateValuation` which returns two valuation methods plus ±IQR confidence
3. User can _pin_ comparables and re-run the estimation on the selected subset only.

---

## Valuation Algorithms (`utils/valuationAlgorithms.js`)

- **Price-per-GFA method** – uses `estimated_gfa` on comps (or computes `area × FSR`) and subject property. Median price × subject GFA gives value.
- **Price-per-Area method** – fallback when GFA absent.
- **Confidence** –
  \( \text{IQR} = Q_3 − Q_1 \) → half-IQR × subject metric.
- Supports _manual selection_ flag; method string records whether analyst manually chose comparables.

---

## Dashboard Highlights

| Widget                  | Description                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------- |
| **Valuation Summary**   | Big £ figure, confidence ±, toggle GFA/Area method.                                                     |
| **Comparable Table**    | Sortable columns (price, date, zone, FSR, HoB, score). Click row centre-pans map via `flyToProperty`.   |
| **Filters Drawer**      | Address search, price/date sliders, zoning dropdowns.                                                   |
| **Scenario Inputs**     | Override zone / FSR / HoB to test alternate development scenarios (updates _subjectDetailsForScoring_). |
| **Heat-map** (optional) | Mapbox layer shades comps by $/m² or score.                                                             |

---

## External Data Sources

- **NSW e-Spatial Services** feature services queried via `arcgisQueries.js` (zone, FSR, HoB, flood, bushfire, biodiversity).
- **PriceFinder / ValNet** – _to-be-integrated_ REST for comparable sales. Current build relies on sample GeoJSON inserted at runtime during demos.

All requests are proxied through `proxy-server` to satisfy CORS.

---

## User-Configurable Elements

The forthcoming **ValuationConfig** (referenced in _14-user-config.md_ roadmap) will allow admins to:

- set default search radius & zoning filters
- customise scoring weights used in `comparableScoring.js`
- define currency / unit display preferences

User-specific preferences are persisted via Supabase table `user_preferences` under key `valuation.settings`.

---

## Integration Points

- **Feasibility Module** – valuationResult feeds residual land price validation.
- **PPTX Generator** – `valuationSlide.js` (template category _Valuation_) consumes `valuationResult` and top 5 comparables.
- **Site-Reviewer** – hazard flags (flood, bushfire) are re-used for ESG scoring.

---

## File Sizes

ValuationDashboard ~210 KB, Context 6 KB, Selector bundle 5 KB, utility files ~25 KB.

Keep this document up-to-date as soon as the PriceFinder integration or scoring weight interface lands.
