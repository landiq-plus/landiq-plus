# Imagery Module (component: `ImageryModal.jsx`)

## Purpose

Provide an **Historical Aerial Imagery** viewer centred on the currently-selected parcel(s). It automates searching multiple WMTS layers, extracts screenshots via mapbox-style `captureHistoricalImagery`, and presents a scrollable timeline so the user can visually compare site conditions across decades.

Typical use-cases: flood-plain change, vegetation removal, subdivision history, and marketing slides in the PPTX exporter.

---

## Public Props

| Prop                          | Type                               | Behaviour                                                                   |
| ----------------------------- | ---------------------------------- | --------------------------------------------------------------------------- |
| `isOpen`                      | `boolean`                          | When true the modal mounts and kicks off the image-search sequence.         |
| `onClose`                     | `() => void`                       | Closes modal and clears auto-scroll interval.                               |
| `selectedFeatures`            | `Feature[]`                        | One or more parcel geometries used for map extent & screenshot cropping.    |
| `developableArea`             | `Feature\|FeatureCollection\|null` | Optional feature to highlight buildable footprint.                          |
| `showDevelopableArea`         | `boolean`                          | Toggles polygon overlay on the preview images.                              |
| `useDevelopableAreaForBounds` | `boolean`                          | If true, screenshots are bounded by developableArea instead of full parcel. |

---

## Internal State

```ts
const [images, setImages]; // { year:number; layerId:string; image:string; }[]
const [loading, setLoading];
const [error, setError];
const [currentIndex, setCurrentIndex];
const [zoom, setZoom]; // 1.0 = fit; >1 = zoom in via CSS scale
// Progress tracking while fetching
const [checkedYears, setCheckedYears];
const [currentlyChecking, setCurrentlyChecking];
const [foundYears, setFoundYears];
const [progressPercent, setProgressPercent];
// Preload optimisation
const [preloadedImages, setPreloadedImages];
const [imageTransitioning, setImageTransitioning];
// Auto-scroll
const [autoScrollEnabled, setAutoScrollEnabled];
const [autoScrollSpeed, setAutoScrollSpeed]; // seconds
```

Two `ref`s:

- `timelineRef` – DOM ref for horizontal timeline scroll.
- `autoScrollIntervalRef` – holds the `setInterval` handle.

---

## Image Search Algorithm (`loadHistoricalImagery`)

1. **Reset state** and progress indicators.
2. Build `featureToUse`:
   - if multiple parcels → wrap as `FeatureCollection`.
3. Group `HISTORICAL_LAYERS` (constant array `{ year, id, styleUrl }`) **by year**.
4. Sort years **descending** so newest appears first.
5. Chunk process (chunks of 5 years) to avoid blocking the UI:
   ```js
   for (let i=0; i<years.length; i+=chunkSize) { … }
   ```
6. For each year in the chunk, call **`captureHistoricalImagery(layer, featureToUse, options)`** which returns `{ imageData, meta }`.
7. **Blank image filter** – helper `isBlankImage(imageData)` samples ~1 % of pixels; if >90 % are near-white the screenshot is discarded.
8. Valid screenshots push `{ year, layerId, image }` into `foundImages`.
9. Update progress: `currentlyChecking`, `checkedYears`, `progressPercent`.
10. When all chunks resolved, sort `foundImages` _descending_, store in `images`, set `loading=false`.

### Pre-loading Adjacent Images

After initial load, `useEffect` calls `preloadAdjacentImages(currentIndex)`; forward/backward images are inserted into `Image()` objects so CSS transitions feel instant.

---

## UI Anatomy

```
┌─ Modal ───────────────────────────────────────────────┐
│ Header: [History icon] Historical Imagery   (× close) │
├───────────────────────────────────────────────────────┤
│ Main viewport:                                        │
│   <AnimatePresence> fade between images               │
│   overlay zoom controls (+ / −), download icon        │
│   optional developable-area outline                   │
│   year badge bottom-left                              │
├───────────────────────────────────────────────────────┤
│ Timeline bar: clickable year ticks                   │
│   progress checkmarks / loading spinners             │
│   <ChevronLeft>  scroll  <ChevronRight>              │
│ Auto-scroll toggle (Play/Pause) + speed slider       │
└───────────────────────────────────────────────────────┘
```

### Key Handlers

| Function                   | Purpose                                                |
| -------------------------- | ------------------------------------------------------ |
| `handlePrevious/Next()`    | manual image navigation, resets `zoom` to 1            |
| `handleZoomIn/Out()`       | multiplies `zoom` by 1.25 / 0.8                        |
| `handleDownload()`         | triggers `download` of current image (with PNG MIME)   |
| `handleTimelineClick(evt)` | jump to clicked year, calculated via bounding rect     |
| `toggleAutoScroll()`       | toggles `autoScrollEnabled` and manages interval setup |

---

## External Dependencies

- **`captureHistoricalImagery`** – custom util that spins up a hidden Mapbox GL map, adds the historical raster layer and exports a canvas PNG.
- **`HISTORICAL_LAYERS`** – config array with WMTS URLs & years.
- `lucide-react` icons, `framer-motion` animations.

---

## Error & Loading Feedback

- While fetching, a centred `<Loader2>` spinner and a percentage ring (via `progressPercent`).
- If `isBlankImage` discards all layers, `error` becomes _"No historical imagery found for this location"_.

---

## Integration Points

- **PPTX Generator** – exported image is passed upstream via `onDownload` callback (currently TODO, see inline comment) so the _Imagery_ slide template can embed time-series.
- **Map View** – not directly modified; screenshots are taken off-screen to avoid affecting the live map.

---

## File Size

`ImageryModal.jsx` ≈ 850 lines.

Updates to layers or screenshot logic should be reflected here.
