# 4. Complete File/Folder Layout

This section expands the earlier outline into an **explicit file tree** (≈ 150 files) so that any contributor can scaffold the repo exactly as intended. Where a pattern repeats across feature‐slices the comment **(repeat per slice)** is used.

> Top-level paths are relative to repository root. Each bullet line shows **file → short role description**.

```text
src/
├─ app/                         # Next-14 "App Router" entry-point
│  ├─ layout.tsx                # Root <html> shell (imports <Navbar />)
│  ├─ globals.css               # Tailwind base + custom variables
│  │
│  ├─ (public)/                 # Public route group (no auth)
│  │   ├─ layout.tsx            # Wraps children with GiraffeProvider
│  │   └─ page.tsx              # Landing page – renders <LandingCards />
│  │
│  ├─ api/                      # Edge/server routes
│  │   ├─ pptx/route.ts         # POST → generate PPTX (uses PPTXGenJS)
│  │   └─ proxy/                # Cloudflare proxy endpoints
│  │       ├─ arcgis/route.ts   # Proxy to ArcGIS REST servers
│  │       └─ map/route.ts      # Proxy to other map services
│  │
│  └─ [module]/page.tsx         # Dynamic catch-all that lazy-loads a feature slice
│
├─ components/
│  ├─ common/                   # Truly generic UI atoms
│  │   ├─ Button.tsx
│  │   ├─ Card.tsx
│  │   ├─ DropdownMenu.tsx
│  │   ├─ Icon.tsx              # Lucide wrapper + mapping helper
│  │   ├─ Loader.tsx            # Spinner / skeleton
│  │   └─ Modal.tsx
│  │
│  └─ navigation/
│      ├─ Navbar.tsx            # Fixed top-right bar
│      ├─ ModuleDropdown.tsx    # Dropdown fed by MODULES config
│      └─ LandingCards.tsx      # Grid of clickable module cards
│
├─ modules/                     # Feature-slices (all lazy-loaded)
│  ├─ localArea/                # <LocalArea /> slice (repeat per slice)
│  │   ├─ index.tsx             # Public re-export; default React component
│  │   ├─ hooks/useLocalArea.ts
│  │   ├─ components/
│  │   │   ├─ LocalAreaForm.tsx
│  │   │   ├─ MapPanel.tsx
│  │   │   └─ StatsPanel.tsx
│  │   ├─ controller.ts         # Calls ArcGIS endpoints via services/api
│  │   ├─ schema.ts             # Zod validation for inbound/outbound data
│  │   ├─ README.md             # Module-specific documentation
│  │   └─ __tests__/LocalArea.test.tsx
│  │
│  ├─ development/              # (repeat structure)
│  ├─ landUse/
│  ├─ imagery/
│  ├─ feasibility/              # Former "pptxApp" feasibility calc
│  ├─ valuation/
│  ├─ shortlist/
│  ├─ openSpace/
│  ├─ merge/
│  └─ pptx/                     # Shared slide templates & helpers
│      ├─ PPTXDashboard.tsx
│      └─ utils.ts
│
├─ contexts/
│  ├─ GiraffeContext.tsx        # Wraps giraffe-sdk client + auth
│  ├─ MapContext.tsx            # Connects to Giraffe map instance
│  ├─ ModuleContext.tsx         # Manages available/permitted modules
│  └─ ConfigContext.tsx         # User preferences & configurations
│
├─ hooks/                       # Generic (non-domain) hooks
│  ├─ useDebounce.ts
│  ├─ useOutsideClick.ts
│  ├─ useLocalStorage.ts
│  ├─ useUserConfig.ts          # Supabase user config access
│  └─ useIsMobile.ts
│
├─ services/
│  ├─ api/                      # Network adapters (fetch + Zod parse)
│  │   ├─ arcgis.ts             # REST + token support
│  │   ├─ giraffe.ts            # SDK wrapper for Giraffe platform
│  │   ├─ cloudflare.ts         # Proxy client for external services
│  │   └─ supabase.ts           # Configuration storage client
│  │
│  ├─ giraffe/
│  │   ├─ auth.ts               # Giraffe user authentication helpers
│  │   ├─ map.ts                # Map integration with Giraffe
│  │   └─ shortlist.ts          # Shortlist management
│  │
│  └─ pptx/
│      ├─ exporter.ts           # Thin wrapper around PPTXGenJS
│      └─ templates/default.json
│
├─ config/
│  ├─ modules.ts                # MODULES array (runtime toggles)
│  ├─ endpoints.ts              # Centralised URLs (ArcGIS, etc.)
│  ├─ themes.ts                 # Colour tokens & Tailwind extension
│  └─ env.d.ts                  # Type-safe process.env accessor
│
├─ types/                       # Source-of-truth TS + Zod schemas
│  ├─ arcgis.ts
│  ├─ giraffe.ts
│  ├─ pptx.ts
│  ├─ imagery.ts
│  ├─ user.ts
│  └─ index.ts                  # Barrel export
│
├─ utils/
│  ├─ auth/
│  │   ├─ token.ts                # Parse/refresh JWT, expose claims
│  │   └─ keycloak.ts             # Optional: silent-refresh helpers
│  │
│  ├─ cache/
│  │   └─ sessionCache.ts         # Typed sessionStorage wrapper (TTL, prefix)
│  │
│  ├─ geometry/
│  │   ├─ buffer.ts
│  │   ├─ centroid.ts
│  │   ├─ transform.ts            # proj4 CRS conversions
│  │   └─ analysis.ts             # area, distance, simplify (turf.js)
│  │
│  ├─ map/
│  │   ├─ utils/
│  │   │   ├─ layerDetails.ts     # Inspect & list visible Giraffe layers
│  │   │   ├─ projection.ts       # Mapbox CRS helpers
│  │   │   └─ geojson.ts          # Type guards & bbox helpers
│  │   │
│  │   ├─ services/
│  │   │   ├─ screenshot.ts       # Generic Mapbox canvas export
│  │   │   ├─ wmsService.ts       # Cached WMS fetch with cancel/retry
│  │   │   └─ screenshots/        # ≈ 18 themed capture functions
│  │   │       ├─ zoningScreenshot.ts
│  │   │       ├─ contourScreenshot.ts
│  │   │       └─ (repeat per layer)
│  │   │
│  │   ├─ config/
│  │   │   ├─ screenshotTypes.ts  # Enum of capture categories
│  │   │   └─ historicalLayers.ts # Time-aware WMS layer list
│  │   │
│  │   └─ captureHistoricalImagery.ts  # Thin wrapper bundling above
│  │
│  ├─ pptx/
│  │   ├─ createSlide.ts
│  │   └─ charts.ts
│  │
│  ├─ services/
│  │   └─ proxyService.ts         # Cloudflare Worker POST wrapper
│  │
│  ├─ api/
│  │   └─ queue.ts                # Promise queue with rate limiting
│  │
│  ├─ stats/
│  │   └─ reportStats.ts          # Track PPTX generation metrics
│  │
│  ├─ format/
│  │   └─ number.ts               # Numeric formatting helpers
│  │
│  ├─ validation/
│  │   └─ zodHelpers.ts
│  │
│  └─ calculations/
│      └─ feasibility.ts
│
├─ public/
│  ├─ images/logo.svg
│  └─ icons/*
│
├─ styles/
│  ├─ tailwind.css
│  └─ variables.css
│
├─ tests/
│  ├─ setupTests.ts             # Jest & RTL config
│  └─ __mocks__/fileMock.ts
│
├─ .eslintrc.cjs
├─ next.config.mjs
├─ tsconfig.json
├─ postcss.config.cjs
└─ README.md
```

## Notes on Feature-Slice Modules

Each feature slice (`modules/<name>/`) is lazy-loaded via dynamic imports for optimal performance and is self-contained with its own:

| File                 | Purpose                                          |
| -------------------- | ------------------------------------------------ |
| `index.tsx`          | Entry component dynamically imported by router   |
| `components/*`       | View-layer sub-components (presentational)       |
| `hooks/use<Name>.ts` | React hooks encapsulating state & business logic |
| `controller.ts`      | Imperative actions (API, map commands)           |
| `schema.ts`          | Zod schema for data validation                   |
| `__tests__/*`        | Jest + RTL unit tests                            |
| `README.md`          | Module-specific documentation                    |

## Giraffe SDK Integration

The application is embedded within the Giraffe platform and leverages its SDK for:

1. User authentication and authorisation
2. Map interaction and visualisation
3. Shortlist and site selection
4. Project data access

The `GiraffeContext` provides access to the Giraffe SDK throughout the application, while module-specific integration is handled in each module's controller.

## Configuration Storage

User preferences and module configurations are stored in Supabase with:

- User email as the primary identifier (from Giraffe SDK)
- Module-specific settings (e.g., PowerPoint templates)
- Default views and UI preferences
- Export configurations

This data is accessed via the `useUserConfig` hook and managed through the `ConfigContext`.

## File Size Management

This breakdown keeps each file well under 300 lines, staying beneath the **file-size-rule** of 1,000 lines and grouping cohesive logic into folders for maintainability.
