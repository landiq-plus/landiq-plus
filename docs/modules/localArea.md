# Local-Area Module (component: `LocalAreaModal/index.jsx`)

## Purpose

Offer a **suburb / LGA context snapshot** for a selected property, covering demographics, population growth, housing-target progress and service-gap indicators. The module pulls live data from ABS & NSW datasets and displays them alongside a thematic suburb map and bar-charts.

It answers questions such as:

- How fast is the suburb growing compared to its LGA?
- Does the LGA meet the NSW Housing Strategy 2029 targets?
- What age-cohorts dominate the area?
- Which key services (schools, hospitals, parks) are missing within catchment?

---

## Public Props (`LocalAreaModal`)

| Prop               | Type                | Description                                                                              |
| ------------------ | ------------------- | ---------------------------------------------------------------------------------------- |
| `isOpen`           | `boolean`           | Controls modal visibility.                                                               |
| `onClose`          | `() => void`        | Close handler.                                                                           |
| `selectedFeatures` | `Feature[]`         | Parcel features; first feature supplies suburb & LGA names from `properties.copiedFrom`. |
| `onDataLoaded`     | `(payload) => void` | Callback fired after all remote data resolve (used by PPTX exporter).                    |

---

## Data Sources & Endpoints

| Dataset                        | Endpoint / File                                                                 | Notes                                              |
| ------------------------------ | ------------------------------------------------------------------------------- | -------------------------------------------------- |
| **Suburb boundary**            | `https://maps.six.nsw.gov.au/…/NSW_Administrative_Boundaries/MapServer/0/query` | Where clause `UPPER(suburbname)=` selected suburb. |
| **ABS 2021 Suburb Population** | `…/FeatureServer/5/query` (ABS General Community Profile)                       | Returns population & age-distribution.             |
| **ABS 2021 LGA Population**    | `…/FeatureServer/3/query`                                                       | Queried with `LGA_NAME_2021`.                      |
| **NSW Housing Targets 2029**   | Local static `/housingTargets.csv` (parsed via PapaParse).                      |

All HTTP requests are proxied through `proxyRequest()` which posts to Cloudflare Worker (`proxy-server`) to bypass CORS.

---

## Internal State (truncated)

```ts
const [suburbData, setSuburbData]; // GeoJSON + attributes
const [populationData, setPopulationData];
const [lgaData, setLgaData];
const [housingTargets, setHousingTargets];

const [loading, setLoading]; // suburb boundary
const [populationLoading, setPopulationLoading];
const [lgaLoading, setLgaLoading];
const [housingTargetsLoading, setHousingTargetsLoading];

const [error, setError]; // suburb errors
const [populationError, setPopulationError];
```

Each dataset has its own loading & error flags so partial failures can be handled gracefully.

---

## Fetch Sequence

```
useEffect ▶ on open
  ├─ loadHousingTargets()                 (if not cached)
  ├─ fetchSuburbData()                    (boundary & attributes)
  ├─ fetchPopulationData(suburb)
  └─ fetchLgaData(LGA)
      ↳ once all succeed ➜ onDataLoaded({ suburbData, populationData, lgaData })
```

`housingTargets.csv` is parsed via PapaParse with `header:true` and numeric typing enabled.

### Housing-Target Lookup

`getHousingTargetsForLGA(lgaName)` attempts:

1. Exact upper-case match.
2. Contains match either direction.
3. Strips common prefixes (_City of_, _Municipality of_) & suffixes (_Council_) and retries.

The helper `calculateImpliedPopulation()` adds `Target2029 × Avg_househ` to 2021 population to chart potential growth.

---

## UI Structure

```
┌ Modal ───────────────────────────────────────────────────┐
│ Header: Local Area Context   (× close)                  │
├──────────────────────────────────────────────────────────┤
│ Left column:                                            │
│   • Suburb boundary map (<SuburbMap>)                   │
│   • KPI chips (population 2021, median age, growth %)   │
│ Right column (tabs):                                    │
│   ▸ Demographics – Age-group bar chart (Chart.js Bar)   │
│   ▸ Housing Targets – progress bar + implied pop        │
│   ▸ Service Gaps – bullet list from extractServiceGaps  │
└──────────────────────────────────────────────────────────┘
```

### Charts

- **Age distribution** – built with `react-chartjs-2`; dataset from `getAgeGroupChartData()` which groups ABS fields `Age_0_4_yr_P … Age_85ov_P`.
- **Housing-target progress** – simple percentage bar (inline div flex) comparing `Target2029` vs current DAs (future TODO).

---

## Helper Utilities

- `normalizeLGAName()` – maps verbose _City of Parramatta_ → _Parramatta_ to align ABS naming.
- `proxyRequest(url, options)` – generic POST wrapper used by all REST queries.
- `extractServiceGapsData()` – reads attributes (schools, hospitals counts) from suburb feature & population size to flag gaps.

---

## Integration Points

- **PPTX Generator** – `onDataLoaded` payload feeds _Local-Area Overview_ slides (map + demographic chart).
- **Feasibility Manager** – may use population and housing-target data (future roadmap).

---

## File Size

`LocalAreaModal/index.jsx` ≈ 850 lines plus `Map.jsx` (≈ 1 990 lines for Mapbox map with suburb highlight & marker cluster).

---

This documentation aligns with the current codebase; update when endpoints, chart datasets or KPI formulas change.
