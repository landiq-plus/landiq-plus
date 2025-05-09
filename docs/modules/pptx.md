# PPTX Export Module (directory: `pptxApp/`)

## Purpose

Generate branded **PowerPoint presentations** (_.pptx_) comprising dynamically‐built slides that visualise the outputs of other modules (Land-Use, Development, Imagery, Feasibility, etc.). Slide content is driven by the user-selected _PPTX configuration_ stored in Supabase and conforms to the `SlideTemplate` / `PPTXConfig` interfaces outlined in _14-user-config.md_.

The module relies on **PptxGenJS** for document creation and `html2canvas` / `three-canvas-export` for rasterising map frames or 3-D massing previews.

---

## High-Level Architecture

```
ReportGenerator.jsx          – entry orchestrator
  ├─ loadConfig(configId)     (from Supabase)
  ├─ for template in config.slides orderBy .order
  │     ↳ import(`slides/${templateId}.js`).createSlide(pres, data, settings)
  └─ pres.writeFile({ fileName })  // triggers download
```

- **Slides** – each slide template exports `createSlide(pres, data, settings)` that mutates a `pptxgen` Presentation instance.
- **Data adapters** – helper functions inside each slide fetch or compute required data (e.g. zoning stats, map screenshot, chart datasets).
- **Global utilities** – `pptxApp/utils/` provides colour palettes, font registrations, image cache and unit conversion helpers.

---

## Key Files

| Path                            | Lines   | Purpose                                                                                                     |
| ------------------------------- | ------- | ----------------------------------------------------------------------------------------------------------- |
| `ReportGenerator.jsx`           | ~119 KB | Loads template config, gathers data from React contexts, builds presentation, updates a progress bar UI.    |
| `slides/coverSlide.js`          | 3.6 KB  | Title slide with hero image & branding.                                                                     |
| `slides/permissibilitySlide.js` | 51 KB   | Matrix chart of zoning vs permitted uses.                                                                   |
| `slides/contextSlide.js`        | 17 KB   | Map + suburb/LGA demographics.                                                                              |
| `slides/developmentSlide.js`    | 19 KB   | Massing parameters & DA stats (consumes DevelopmentModal context).                                          |
| `slides/feasibilitySlide.js`    | 18 KB   | OPTIONAL – injects residual‐value table; uses Feasibility context (calculations reside in separate module). |
| `SlidePreview.jsx`              | 6.1 KB  | Renders a canvas preview of the next slide while building.                                                  |
| `slides/scoringLogic.js`        | 130 KB  | Reusable scoring & colour-class helpers shared by multiple slide types.                                     |

> NOTE: although `feasibilitySlide.js` exists here, the **Feasibility calculations** themselves live in the _Feasibility module_ we documented separately; this file only renders its results.

---

## User Configuration Flow

1. **PPTXConfigEditor** (outside this repo scope) builds a JSON config object adhering to:
   ```ts
   interface PPTXConfig {
     id: string;
     name: string;
     slides: {
       templateId: string; // e.g. 'permissibilitySlide'
       order: number;
       settings: Record<string, any>; // template-specific
     }[];
     defaultBranding: {
       logo: string;
       primaryColor: string;
       secondaryColor: string;
     };
   }
   ```
2. Config is saved to Supabase & referenced by ReportGenerator via `configId` prop.
3. During export the generator injects extra runtime options (e.g. map style, DPI) from `user_preferences:pptx`.

---

## Slide Lifecycle (per template)

```js
export async function createSlide(pres, data, settings) {
  const slide = pres.addSlide();
  // 1. Gather required data
  const zoning = await getZoningStats(data.selectedFeature);
  // 2. Add shapes / images
  slide.addTable(zoning.table, { colW: … });
  // 3. Apply branding colours & fonts from settings.branding
  return slide;
}
```

Templates must be **self-contained**; heavy computations (e.g. suitability scoring) live in `/slides/scoringLogic.js` to avoid duplication.

---

## External Libraries

- `pptxgenjs` – core PPTX builder.
- `html2canvas` – map + DOM screenshot for image slides.
- `chart.js` – charts converted to PNG via `<canvas>.toDataURL()` and inserted into slides.
- `three` + `three-mesh-export` – exports massing model PNG for developmentSlide.

---

## Progress & Error Handling

- ReportGenerator tracks progress per slide, updating `<GenerationProgress>` component.
- Slides may throw; errors are caught and displayed inside modal log (`GenerationLog.jsx`) but export continues for remaining slides.

---

## Integration Points

- Consumes React contexts provided by **Land-Use**, **Development**, **Imagery**, **Feasibility**, **Site-Reviewer** and **Valuation** modules.
- Emits `fileName` once download finishes; parent component shows toast notification.

---

## File Size

Slides directory ≈ 450 KB total; individual slide files 15–50 KB. ReportGenerator ~120 KB.

---

Keep this doc updated when new slide templates are added or template API changes.
