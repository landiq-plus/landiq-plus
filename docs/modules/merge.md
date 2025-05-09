# Merge Module (component: `MergeSelector.jsx`)

## Purpose

Enable users to **merge multiple property polygons** into a single developable area for downstream calculations (Feasibility, Land-Use, Massing, etc.). It provides two main flows:

1. **Manual multi-select** – user clicks parcels on the map, then clicks _Merge_.
2. **Attribute merge** – user specifies an attribute (e.g. `plan_id`), and all features with the same value are merged in one action.

The module lives under `pptxApp/Utilities/Merge` and can be launched from the Utilities drop-down.

---

## File Inventory

| File                | Lines | Responsibility                                           |
| ------------------- | ----- | -------------------------------------------------------- |
| `MergeSelector.jsx` | ~670  | UI + Turf `union`, handles selection list & merge logic. |
| `index.jsx`         | 8     | Barrel export.                                           |

---

## Public Props (`<MergeSelector>`)

| Prop               | Type                      | Notes                                   |
| ------------------ | ------------------------- | --------------------------------------- |
| `selectedFeatures` | `Feature[]`               | Initial selection from Map.             |
| `onMergeComplete`  | `(mergedFeature) => void` | Receives merged polygon / multipolygon. |
| `onCancel`         | `() => void`              | Close handler without merging.          |

---

## Key State & Hooks

```ts
const [selection, setSelection]; // Feature[] currently chosen
const [merging, setMerging]; // boolean spinner flag
const [mergeStrategy, setMergeStrategy]; // 'manual' | 'attribute'
const [attributeField, setAttributeField]; // string (e.g. plan_id)
const [error, setError];
```

- `useEffect` recalculates preview union every time `selection` changes.
- Map highlight layer added via `map.addSource('merge_preview', …)` when modal mounts.

---

## Merge Algorithm

1. Choose feature set (`selection`).
2. If strategy = **attribute**: group by `properties[attributeField]` and flatten.
3. Run Turf `union` iteratively:
   ```js
   let merged = selection[0];
   for (let i = 1; i < selection.length; i++) {
     merged = turf.union(merged, selection[i]);
   }
   ```
4. Strip any `properties` to avoid multi-value conflicts; retain an array `source_ids`.
5. Pass merged feature to `onMergeComplete` and close modal.

During processing `setMerging(true)` shows `<Loader2>` overlay.

---

## UI Flow

```
┌ Modal ─────────────────────────────────────────┐
│ Header: Merge Parcels (× close)               │
├───────────────────────────────────────────────┤
│ 1. Pick strategy (radio)                      │
│    • Manual select                            │
│    • Attribute merge                          │
│      ⟶ dropdown of common field names        │
│ 2. Selection table (property id, area, zone) │
│    – checkbox to exclude individual features │
│ 3. Map preview (blue outline of union)        │
│ 4. Footer buttons:  [Cancel]   [Merge]        │
└───────────────────────────────────────────────┘
```

---

## Error Handling

- If union fails (e.g. invalid geometries) catch error → red banner _"Unable to merge selected polygons. Try dissolving invalid rings in GIS."_

---

## Integration Points

- **Feasibility Manager** – passes merged feature as `developableArea` prop.
- **Imagery & Land-Use Modals** – may use merged footprint for bounds.
- **Site Reviewer** – treat merged area as assessment boundary.

---

## File Size

`MergeSelector.jsx` ≈ 27 KB; modal remains lightweight.

This doc reflects current code – update if selection strategies or prop names change.
