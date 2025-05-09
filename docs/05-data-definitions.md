# 5. Data Definitions

A core principle of the application is to have a single source of truth for all data types, schemas, and validations. This ensures consistency across the codebase and makes maintenance easier.

## Type Definition Files

1. **`types/landiq.ts`** – Type definitions for working with the LandiQ application, including:

   - Lots and Properties data structures
   - Interfaces for data returned from the Giraffe SDK
   - Helper types for SDK method parameters

2. **`types/arcgis.ts`** – GeoJSON & REST payload types.

3. **`types/pptx.ts`** – Shapes of PPTX slide templates.

4. **`types/index.ts`** – Barrel file exporting everything.

## Validation Approach

We use **Zod** for runtime validation and **TypeScript** for compile-time type checking:

```ts
// types/arcgis.ts
export const FeatureCollectionSchema = z.object({
  type: z.literal("FeatureCollection"),
  features: z.array(FeatureSchema),
});
export type FeatureCollection = z.infer<typeof FeatureCollectionSchema>;
```

## Benefits

- **Runtime safety**: All external data is validated before use
- **Self-documenting**: Types describe the shape of data
- **IDE support**: Autocomplete and error checking
- **Single source**: Update a schema in one place, affects everywhere it's used

## Common Data Sources

Each of these data types should be defined once and imported wherever needed:

| Data Type           | Description                      | Current Location                                    | Centralised Location             |
| ------------------- | -------------------------------- | --------------------------------------------------- | -------------------------------- |
| Zone Colors         | Colour codes for planning zones  | `src/components/pptxApp/zoneColors.js` (duplicated) | `src/data/colors/zoneColors.ts`  |
| Land Use Categories | Classification of land use types | Various modules                                     | `src/data/landUse/categories.ts` |
| Development Types   | Standard development typologies  | Feasibility/Development modules                     | `src/data/development/types.ts`  |
| Planning Controls   | Planning instrument controls     | Multiple files                                      | `src/data/planning/controls.ts`  |

### Zone Colours

The zone colors data should be centralised in `src/data/colors/zoneColors.ts`:

```typescript
/**
 * Standard zone colours for consistent display across modules
 * Source: NSW land_zoning_rgb_hex_codes.csv
 */
export const zoneColors: Record<string, string | null> = {
  "2(a)": "#FFA6A3", // Residential (Low Density)
  A: "#FC776E", // Residential Zone - Medium Density Residential
  AGB: "#FAE8C5", // Agribusiness
  B: "#63F0F5", // Business Zone - Local Centre
  B1: "#C9FFF9", // Neighbourhood Centre
  B2: "#62F0F5", // Local Centre
  B3: "#00C2ED", // Commercial Core
  B4: "#959DC2", // Mixed Use
  B5: "#7DA0AB", // Business Development
  B6: "#95BFCC", // Enterprise Corridor
  B7: "#BAD6DE", // Business Park
  C: "#BAD6DE", // Business Zone - Business Park
  C1: "#E69900", // National Parks and Nature Reserves
  C2: "#F0AE3C", // Environmental Conservation
  C3: "#F7C568", // Environmental Management
  C4: "#FFDA96", // Environmental Living
  CA: null, // Complex Area (No Fill)
  D: "#959DC2", // Business Zone - Mixed Use
  DM: "#FFFFFF", // Deferred Matter
  DR: "#FFFF70", // Drainage
  E: "#00C2ED", // Environment
  E1: "#62F0F5", // Local Centre
  E2: "#B4C6E7", // Commercial Centre
  E3: "#8EA9DB", // Productivity Support
  E4: "#9999FF", // General Industrial
  E5: "#9966FF", // Heavy Industrial
  EM: "#95BFCC", // Employment
  ENP: "#FFD640", // Environment Protection
  ENT: "#76C0D6", // Enterprise
  ENZ: "#73B273", // Environment and Recreation
  EP: "#FCF9B6", // Employment
  F: "#FFFFA1", // Special Purposes Zone - Community
  G: "#FFFF70", // Special Purposes Zone - Infrastructure
  H: "#55FF00", // Recreation Zone - Public Recreation
  I: "#D3FFBF", // Recreation Zone - Private Recreation
  IN1: "#DDB8F5", // General Industrial
  IN2: "#F3DBFF", // Light Industrial
  IN3: "#C595E8", // Heavy Industrial
  MAP: "#E6FFFF", // Marine Park
  MU: "#959DC2", // Mixed Use
  MU1: "#959DC2", // Mixed Use
  P: "#B3CCFC", // Parkland
  PAE: "#F4EC49", // Port and Employment
  PEP: "#74B374", // Permanent Park Preserve
  PRC: "#549980", // Public Recreation
  R: "#B3FCB3", // Residential
  R1: "#FFCFFF", // General Residential
  R2: "#FFA6A3", // Low Density Residential
  R3: "#FF776E", // Medium Density Residential
  R4: "#FF483B", // High Density Residential
  R5: "#FFD9D9", // Large Lot Residential
  RAC: "#E6CB97", // Rural Activity Zone
  RAZ: "#E6CB97", // Rural Activity Zone
  RE1: "#55FF00", // Public Recreation
  RE2: "#D3FFBE", // Private Recreation
  REC: "#AEF2B3", // Recreation
  REZ: "#DEB8F5", // Regional Enterprise Zone
  RO: "#55FF00", // Regional Open Space
  RP: "#D3FFBE", // Regional Park
  RU1: "#EDD8AD", // Primary Production
  RU2: "#E6CA97", // Rural Landscape
  RU3: "#DEC083", // Forestry
  RU4: "#D6BC6F", // Primary Production Small Lots
  RU5: "#D6A19C", // Village
  RU6: "#C79E4C", // Transition
  RUR: "#EFE4BE", // Rural
  RW: "#D3B8F5", // Road and Road Widening
  SET: "#FFD2DC", // Settlement
  SP1: "#FFFFA1", // Special Activities
  SP2: "#FFFF70", // Infrastructure
  SP3: "#FFFF00", // Tourist
  SP4: "#FFFF00", // Enterprise
  SP5: "#E6E600", // Metropolitan Centre
  SPU: "#FFFF00", // Special Uses
  T: "#FCD2EF", // Tourism
  U: "#CAFCED", // Unzoned
  UD: "#FF7F63", // Urban Development
  UL: "#FFFFFF", // Unzoned Land
  UR: "#FF776E", // Urban
  W: "#FCC4B8", // Waterway
  W1: "#D9FFF2", // Natural Waterways
  W2: "#99FFDD", // Recreational Waterways
  W3: "#33FFBB", // Working Waterways
  W4: "#00E6A9", // Working Waterfront
  WFU: "#1182C2", // Waterfront Use
};
```

### Development Types

Standard development types should be centralised in `src/data/development/types.ts`:

```typescript
/**
 * Standard development typologies used across modules
 */
export interface DevelopmentType {
  id: string;
  name: string;
  description: string;
  defaultFSR?: number;
  defaultHeight?: number;
  category: "residential" | "commercial" | "industrial" | "mixed" | "other";
}

export const developmentTypes: DevelopmentType[] = [
  {
    id: "residential_low",
    name: "Low Density Residential",
    description: "Detached dwellings, typically single-family homes",
    defaultFSR: 0.5,
    category: "residential",
  },
  {
    id: "residential_medium",
    name: "Medium Density Residential",
    description: "Townhouses, terraces, and low-rise apartments",
    defaultFSR: 0.9,
    defaultHeight: 12,
    category: "residential",
  },
  {
    id: "residential_high",
    name: "High Density Residential",
    description: "Mid to high-rise apartment buildings",
    defaultFSR: 2.5,
    defaultHeight: 30,
    category: "residential",
  },
  {
    id: "commercial_retail",
    name: "Retail",
    description: "Shopping centres, supermarkets, and retail premises",
    defaultFSR: 1.0,
    category: "commercial",
  },
  {
    id: "commercial_office",
    name: "Commercial Office",
    description: "Office buildings and business premises",
    defaultFSR: 3.0,
    defaultHeight: 36,
    category: "commercial",
  },
  {
    id: "industrial_warehouse",
    name: "Warehouse/Logistics",
    description: "Storage, distribution, and logistics facilities",
    defaultFSR: 0.8,
    category: "industrial",
  },
  {
    id: "mixed_use",
    name: "Mixed Use",
    description: "Combined retail, commercial, and residential",
    defaultFSR: 2.0,
    defaultHeight: 24,
    category: "mixed",
  },
];
```

### Planning Controls

Planning control identifiers should be centralised in `src/data/planning/controls.ts`:

```typescript
/**
 * Standard planning control identifiers used across modules
 */
export const planningControls = {
  HEIGHT: "HOB",
  FSR: "FSR",
  MIN_LOT_SIZE: "MIN_LOT",
  HERITAGE: "HER",
  LAND_RESERVATION: "LAND_RES",
  ACID_SULFATE_SOILS: "ASS",
  FLOODING: "FLOOD",
  FORESHORE_BUILDING_LINE: "FBL",
  SCENIC_PROTECTION: "SCENIC",
};

export interface PlanningControl {
  id: string;
  name: string;
  description: string;
  unit?: string;
  arcGISLayerId?: string;
}

export const planningControlDetails: PlanningControl[] = [
  {
    id: planningControls.HEIGHT,
    name: "Height of Building",
    description: "Maximum permissible building height",
    unit: "m",
    arcGISLayerId: "height_of_buildings",
  },
  {
    id: planningControls.FSR,
    name: "Floor Space Ratio",
    description: "Maximum floor space ratio",
    arcGISLayerId: "floor_space_ratio",
  },
  {
    id: planningControls.MIN_LOT_SIZE,
    name: "Minimum Lot Size",
    description: "Minimum lot size for subdivision",
    unit: "m²",
    arcGISLayerId: "minimum_lot_size",
  },
  // Additional controls...
];
```

### ArcGIS Layer IDs

ArcGIS layer identifiers should be centralised in `src/data/arcgis/layers.ts`:

```typescript
/**
 * Standard ArcGIS layer IDs used across modules
 */
export const arcgisLayers = {
  // Base layers
  CADASTRE: "nsw_cadastre",
  AERIAL_IMAGERY: "nsw_aerial_imagery",

  // Planning layers
  ZONING: "nsw_zoning",
  HEIGHT_OF_BUILDINGS: "height_of_buildings",
  FLOOR_SPACE_RATIO: "floor_space_ratio",
  HERITAGE: "heritage",

  // Environmental layers
  VEGETATION: "vegetation",
  BUSHFIRE: "bushfire_prone_land",
  FLOODING: "flood_planning",

  // Infrastructure layers
  ROADS: "roads",
  TRANSPORT: "public_transport",
  UTILITIES: "utilities",
};

export interface ArcGISLayer {
  id: string;
  name: string;
  url: string;
  metadataUrl?: string;
  type: "feature" | "map" | "image";
}

export const arcgisLayerDetails: Record<string, ArcGISLayer> = {
  [arcgisLayers.CADASTRE]: {
    id: arcgisLayers.CADASTRE,
    name: "NSW Digital Cadastre",
    url: "https://maps.six.nsw.gov.au/arcgis/rest/services/public/NSW_Cadastre/MapServer",
    type: "map",
  },
  [arcgisLayers.ZONING]: {
    id: arcgisLayers.ZONING,
    name: "NSW Planning Zones",
    url: "https://maps.six.nsw.gov.au/arcgis/rest/services/planning/NSW_Planning_Zones/MapServer",
    type: "map",
  },
  // Additional layer definitions...
};
```

### Map Configuration

Map configuration constants should be centralized in dedicated files:

```typescript
// src/data/map/historicalLayers.ts
/**
 * Configuration for historical imagery layers available from Metromap
 */
export const HISTORICAL_LAYERS = [
  {
    id: "aerial2022",
    year: 2022,
    label: "2022 NSW Aerial",
    provider: "Metromap",
    url: "https://maps.six.nsw.gov.au/arcgis/rest/services/public/NSW_Aerial_Imagery_2022/MapServer",
    tileSize: 256,
  },
  {
    id: "aerial2020",
    year: 2020,
    label: "2020 NSW Aerial",
    provider: "Metromap",
    url: "https://maps.six.nsw.gov.au/arcgis/rest/services/public/NSW_Aerial_Imagery_2020/MapServer",
    tileSize: 256,
  },
  // Additional years...
];

// src/data/map/screenshotTypes.ts
/**
 * Standard categories for map screenshots
 */
export enum SCREENSHOT_TYPES {
  ZONING = "zoning",
  CONTOUR = "contour",
  FSR = "fsr",
  HEIGHT = "height",
  HERITAGE = "heritage",
  ACID_SULFATE = "acid_sulfate",
  WATER = "water",
  SEWER = "sewer",
  POWER = "power",
  GEOSCAPE = "geoscape",
  ROADS = "roads",
  STRATEGIC = "strategic",
  PTAL = "ptal",
  FLOOD = "flood",
  BUSHFIRE = "bushfire",
  TEC = "tec",
  BIODIVERSITY = "biodiversity",
  CONTAMINATION = "contamination",
  HISTORICAL = "historical",
  CONTEXT = "context",
}
```

These configuration files provide consistent data to the map utilities in `utils/map/` and ensure that all components reference the same values.

## Centralisation Guidelines

When working with shared data:

1. **Identify duplicated data** - Look for the same constants, objects, or arrays used in multiple places
2. **Create a dedicated location** - Use the `src/data/` directory with appropriate subdirectories
3. **Create type definitions** - Use TypeScript interfaces to define the shape of the data
4. **Document the source** - Add comments indicating where the data comes from
5. **Refactor usages** - Update all files to import from the centralised location

## Benefits of Data Centralisation

1. **Single source of truth** - Changes only need to be made in one place
2. **Type safety** - TypeScript interfaces ensure consistent usage
3. **Documentation** - Central location for data descriptions and sources
4. **Reusability** - Easy imports across modules
5. **Testing** - Simplified testing of shared data structures
