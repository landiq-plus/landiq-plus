# Site-Reviewer Module (previously _OpenSpace_ utilities)

## Purpose

Provide a **flexible assessment dashboard** where users can design their own review templates (fields, scoring formulas, visualisations) and apply them to a selected site or development scenario. Originally focused on open-space sufficiency, it has been generalised so any assessment domain (e.g. ESG, service access, design excellence) can be configured.

---

## Directory Map

```
Utilities/OpenSpace/
├─ index.jsx                      – entry barrel
├─ OpenSpaceContext.jsx           – React context for assessment state
├─ Dashboard/                    │
│   ├─ KPIWidgets.jsx             │ small cards: POS %, WalkScore etc.
│   └─ RadarChart.jsx             │ multi-metric radar
├─ components/                   │
│   ├─ TableView.jsx              │ editable grid of scores by assessment field
│   ├─ StatusCards.jsx            │ colour-coded pass/fail cards
│   └─ OpenSpaceReview.jsx        │ orchestrates tabs
├─ FieldManagement.jsx            – CRUD for custom fields & categories
├─ hooks/                         – reusable hooks (selection, scoring)
└─ utils/                         – calc helpers (walk-catchment, NDVI)
```

---

## Public Props (`<OpenSpaceReview>` root)

| Prop              | Type                        | Notes                                      |
| ----------------- | --------------------------- | ------------------------------------------ |
| `selectedFeature` | `Feature`                   | Parcel being reviewed.                     |
| `developableArea` | `FeatureCollection \| null` | Optional footprint override.               |
| `onComplete`      | `(summary) => void`         | Emits final score card for PPTX/Reporting. |

---

## Feature Highlights

### 1 Custom Field Engine

- Each field has: `{ id, name, type, weight, formula }`.
- Supported types: `number`, `boolean`, `select`, `score`.
- Formulas are JavaScript strings evaluated with `eval()` inside a **safe sandbox** (only maths functions exposed) to compute scores.
- CRUD UI in `FieldManagement.jsx` with drag-reorder & category grouping.

### 2 Scoring & Aggregation

- Hook `useAssessmentScore(fields, rawInputs)` returns:
  - per-field `score` and `explanation` string.
  - aggregated `weightedScore`.
- Radar chart plots category averages.

### 3 Service-Gap Calculations (Open-Space legacy)

- Functions in `utils/serviceCalcs.js` measure distance to nearest park, playground, school, GP, hospital using Mapbox Turf `nearestPoint`. Flags insufficiency if > threshold.

### 4 GIS Visuals

- Mapbox layers show evaluated points (services) with colour by compliance.
- Table rows link to map via "zoom" icon.

---

## Context Object (`OpenSpaceContext`)

```ts
{
  fields: FieldDefinition[];
  rawInputs: Record<string, any>;   // collected via TableView
  scores: {
    [fieldId]: {
       value: number;
       score: number;   // 0-100
       status: 'pass'|'warn'|'fail';
    }
  };
  overallScore: number;            // weighted avg
  addField(), updateField(), …     // actions
}
```

Context allows deep-nested widgets (StatusCards, KPIWidgets) to stay in sync.

---

## Configuration Store

Conforms to **SiteReviewConfig** interface in _14-user-config.md_. Saved in Supabase table `user_preferences` with module=`siteReviewer`.

---

## Export Flow

`onComplete` packs:

```ts
{
  overall: 78,
  failingFields: ['open_space_ratio','gp_distance'],
  screenshot: <base64 png> // via html2canvas of Dashboard
}
```

This payload is consumed by `ReportGenerator` → slide template _Site Review_.

---

## Integration Points

- **Feasibility** – failing open-space or school distance will surface warning icons in FeasibilitySummary.
- **DevelopmentModal** – service layers re-used for merge view.

---

## File Size

- `FieldManagement.jsx` ~44 KB (forms & drag-drop logic).
- `TableView.jsx` ~33 KB.
- `OpenSpaceContext.jsx` ~26 KB.

---

Update this doc when new field types or scoring formulas are introduced.
