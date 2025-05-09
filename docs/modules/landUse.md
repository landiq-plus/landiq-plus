# Land-Use Module (root component: `LandUseModal/index.jsx`)

## Purpose

Analyse **existing land-use mix** and **zoning permissibility** for a selected site or group of sites. It identifies gaps in open-space, community services, employment uses and provides interactive dashboards to support rezoning or master-planning decisions.

This module is often used before the Development & Feasibility steps to understand _what the planning scheme currently allows_ and _where conflicts or shortfalls lie_.

---

## Directory Map

```
LandUseModal/
├─ index.jsx                         (≈ 3 750 lines)  – modal orchestrator
├─ exportImage.js                    – exports PNG of current land-use map
├─ hooks/
│   ├─ usePermissibilityMatrix.js    – memoised matrix builder
│   └─ useServiceSufficiency.js      – calculates open-space & service KPIs
├─ utils/
│   ├─ zoningColours.js              – colour dictionary for legend
│   └─ scoring.js                    – suitability scoring helpers
├─ components/
│   ├─ SectionHeader.jsx             – reusable sub-headers
│   ├─ LandUseTable.jsx              – current vs permitted table view
│   ├─ SuitabilityMap.jsx            – Mapbox GLSource+Layer wrapper
│   └─ KPIBadges.jsx                 – circular KPI widgets
├─ ServiceSufficiencyContext.jsx     – React context for child charts
├─ OpenSpaceSufficiencyContext.jsx   – context for POS metrics
└─ InsufficientLandUsesDisplay.jsx   – quick list of missing uses
```

---

## Public Props (`LandUseModal`)

| Prop               | Type                        | Notes                                                           |
| ------------------ | --------------------------- | --------------------------------------------------------------- |
| `isOpen`           | `boolean`                   | Modal visibility.                                               |
| `onClose`          | `() => void`                | Close handler.                                                  |
| `selectedFeatures` | `Feature[]`                 | Parcels or polygons under review.                               |
| `developableArea`  | `FeatureCollection \| null` | Optional footprint override (highlights in maps).               |
| `siteTemplate`     | `string \| null`            | If provided, loads saved permissibility template from Supabase. |

---

## Data Inputs

1. **ArcGIS REST** layers (planning scheme):
   - `zones`, `overlays`, `permittedUses` service URLs configured in **env**.
2. **Local CSV look-ups** for _permissibility matrix_ (e.g. R1 → Residential, Retail permitted?).
3. **Supabase** tables (optional): user-saved land-use templates.

All remote requests are routed via the `proxy-server` Cloudflare Worker.

---

## Core Workflow

```
useEffect ▶ when modal opens
  ⟶ fetchZonePolygons(selectedFeatures)
  ⟶ buildPermissibilityMatrix(zoneCodes[], LOCAL_MATRIX)
  ⟶ analyseServiceSufficiency(developableArea, open-space layers)
  ⟶ setContexts: { openSpaceKPIs, serviceKPIs }
  ⟶ render map + table + KPI badges
```

### Permissibility Matrix (`usePermissibilityMatrix` hook)

- Input: array of zone codes (`R1`, `B6`, etc.)
- Output: object `{ zone: { landUseCategory: 'Permitted' | 'Prohibited' | 'STCA' } }`.
- Memoised with `useMemo` to avoid recalculation when only filters update.

### Service Sufficiency (`useServiceSufficiency` hook)

- Calculates:
  - **Open-space ratio** – POS area / population.
  - **Playground catchment** distance.
  - **Community facility GFA per 1 000 residents**.
- Uses thresholds from NSW DPIE (hard-coded constants file).
- Returns KPI flags: `Adequate | Marginal | Insufficient`.

### Context Providers

```
<ServiceSufficiencyContext.Provider value={{ kpis, rawLayers }}>
  <OpenSpaceSufficiencyContext.Provider value={{ openSpaceScore }}>
     …child charts…
  </…>
</…>
```

So nested child components (eg. `InsufficientLandUsesDisplay`) can subscribe without prop-drilling.

---

## UI Sections

| Section              | Component                                                           | Highlights                                                                   |
| -------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Header               | `SectionHeader`                                                     | Zone code chips, close button.                                               |
| Permissibility Table | `LandUseTable`                                                      | Sticky first-column, coloured cells (green / orange / red).                  |
| KPI Badges           | `KPIBadges`                                                         | Small circular gauges (Open space %, Jobs-Housing ratio, etc.).              |
| Suitability Map      | `SuitabilityMap`                                                    | Mapbox GL map with **zoning layer**, optional heat-map of suitability score. |
| Insufficient Uses    | `InsufficientLandUsesDisplay`                                       | Lists land uses missing or under-provided.                                   |
| Export               | Top-right `Download` icon ➜ `exportImage.js` captures map + badges. |

---

## Export Image (`exportImage.js`)

- Uses `html2canvas` to rasterise the `.modal-content` div.
- Adds attribution & timestamp footer.
- Passes PNG data-URI up via `onExport(image)` prop (used by PPTX generator).

---

## Hooks & Utility Highlights

- **`useOutsideClick`** – closes dropdowns when clicking outside.
- **`debounce`** – from `lodash` to throttle map move events when recomputing suitability overlay.
- **`scoring.js`** – generic formula parser (string of JS) passed from admin panel; enables user-defined suitability weightings.

---

## Integration Points

- **Feasibility Module** – uses permissibility outcomes to pre-select dwelling types.
- **Site Reviewer (Open-Space) Module** – shares Open-space KPI context.
- **PPTX Generator** – receives exported PNG for the _Land-Use Analysis_ slide template.

---

## File Size

`index.jsx` ~3 750 lines. Sub-components range 150-800 lines each.

---

This document mirrors the code as at the latest commit; remember to update when hooks, contexts or endpoint URLs change.
