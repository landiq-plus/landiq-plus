# Development Module (React component: `DevelopmentModal/index.jsx`)

## Purpose

Displays **Development Applications (DAs)** relevant to the currently-selected parcel (or group of parcels) and lets the user:

- inspect application details (status, cost, lodgement date, description, dwelling yield)
- apply filters & sorting
- view chart summaries
- export the results to a Mapbox GeoJSON layer for further GIS or reporting use

The component is mounted as a modal; it does **not** calculate feasibility or massing – it focuses purely on DA exploration.

---

## Public Props

| Prop               | Type         | Role                                                                                                 |
| ------------------ | ------------ | ---------------------------------------------------------------------------------------------------- |
| `isOpen`           | `boolean`    | Controls modal visibility and drives the initial data fetch.                                         |
| `onClose`          | `() => void` | Parent callback to hide the modal.                                                                   |
| `selectedFeatures` | `Feature[]`  | The first item provides geometry & copied‐property values used to derive the LGA and polygon filter. |

---

## Data-fetch Workflow (code-trace)

1. **`fetchDevelopmentData()`** runs inside `useEffect` when the modal is first opened.
2. The primary feature's `site_suitability__LGA` → translated to a **council name** via `lgaMapping`.
3. `fetchAllDAs(councilName)` issues **paginated POSTs** (via the Cloudflare Worker proxy) to:
   ```text
   https://api.apps1.nsw.gov.au/eplanning/data/v0/OnlineDA
   ```
   with JSON headers:
   ```json
   {
     "ApplicationType": ["Development Application"],
     "LodgementDateFrom": "2020-01-01",
     "CouncilName": ["<council>"]
   }
   ```
4. Each page's `Application[]` is pushed into an array until `TotalPages` is exhausted.
5. **`deduplicateDAs(applications)`** groups records by `FullAddress + CostOfDevelopment` and keeps the latest `LodgementDate`.
6. If the parcel geometry is a polygon, DAs are filtered with **`turf.booleanPointInPolygon`** so only applications _inside_ the site/developable-area remain.
7. The cleaned array is stored in React state: `developmentData`.

---

## React State (simplified)

```ts
const [developmentData, setDevelopmentData] = useState<Application[]>([]);
const [loading, setLoading] = useState(false);
const [error, setError] = useState<string | null>(null);
const [featureProperties, setFeatureProperties] = useState<Record<
  string,
  any
> | null>(null);
const [activeTab, setActiveTab] = useState<"all" | "charts" | "summary">("all");

const [filters, setFilters] = useState({
  developmentType: null as string | null,
  status: null as string | null,
  developmentCategory: null as string | null,
  value: { operator: null, value1: null, value2: null },
  dwellings: { operator: null, value1: null, value2: null },
  lodgedDate: { operator: null, date1: null, date2: null },
});
```

Derived helpers:

- **`activeFilterCount`** (`useMemo`) – badge number in the UI.
- **`getFilteredApplications()`** – sequentially applies every filter.
- **`toggleSort(field)`** – toggles ascending/descending and sets `sortField`.

---

## Summary Statistics

`calculateSummaryStats(applications)` produces an object with:

- `totalApplications`
- `byStatus` (Lodged, Under Assessment, Determined, …)
- `byType` (Residential flat building, Dual occupancy, etc.)
- `byResidentialType` (subset using a predefined `RESIDENTIAL_TYPES` set)
- `totalValue` (Σ `CostOfDevelopment`)
- `totalDwellings` (Σ `UnitsProposed` when available)

These stats feed the _Summary_ tab cards and chart data-sets.

---

## Visual Components

| Area                                                  | Libraries / files                                                       |
| ----------------------------------------------------- | ----------------------------------------------------------------------- |
| Charts (status bar, category pie, lodgement timeline) | `recharts` (`BarChart`, `PieChart`, `ResponsiveContainer`)              |
| Icons                                                 | `lucide-react` (e.g. `Construction`, `Filter`, `FileSpreadsheet`)       |
| Animation                                             | `framer-motion` (tab transitions)                                       |
| GIS layer helpers                                     | `mapLayerUtils.js` → `createDevelopmentLayer`, `removeDevelopmentLayer` |
| Tooltip                                               | `InfoTooltip.jsx` for explanatory hover hints                           |

### Tabs

- **ALL** – table view with sortable columns & inline filter pills.
- **CHARTS** – bar/pie/timeline charts based on `summaryData`.
- **SUMMARY** – KPI cards (total apps, total cost, etc.)

### Filter Side-Panel

A slide-out panel (triggered by the `Filter` icon) allows the user to set complex filter objects for value, dwellings and date ranges.

---

## GIS Layer Export

`handleCreateGeoJSONLayer()` converts the currently-filtered DA list to a **FeatureCollection** and adds a Mapbox layer via `createDevelopmentLayer`. `removeDevelopmentLayer` cleans up when the modal closes.

---

## Error & Loading Handling

- `loading` toggles a `<Loader2>` spinner overlay.
- Any caught error sets `error` → a red banner is rendered at the top of the modal.

---

## Known Integration

The only outward side-effect coded inside this module is the optional GeoJSON layer creation; other modules (Imagery, Feasibility, PPTX slides) may subscribe to that layer but no direct function calls are made from `DevelopmentModal`.

---

## File Footprint

- `index.jsx` – ≈ 1 730 lines
- `mapLayerUtils.js` – style helpers & Mapbox additions
- `tooltipContent.js` – builds HTML for status/description hover cards
- `developmentTypes.js` – categorisation dictionary used in filters & charts

This document will be updated automatically if the component API or data-flow changes.
