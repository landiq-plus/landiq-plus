# 14. User Configuration Architecture

Beyond the admin-level module toggles, end-users need fine-grained control over their experience. These configurations will be stored in Supabase and synced to localStorage as fallback.

## PowerPoint Export Library & Settings

The PowerPoint export system allows users to:

1. **Select slide content from a template library**
2. **Customise slide layouts & data inclusions**
3. **Save configurations as reusable templates**
4. **Add custom logos and branding**
5. **Configure scoring logic and thresholds**
6. **Share templates with team members**

### Interface Definitions

```typescript
// types/pptx.ts
export interface SlideTemplate {
  id: string;
  name: string;
  category:
    | "Cover"
    | "Map"
    | "Analysis"
    | "Data"
    | "Comparison"
    | "Property Overview"
    | "Zoning Analysis"
    | "Flood Analysis"
    | "Development Potential"
    | "Site Constraints"
    | "Valuation"
    | "Executive Summary";
  thumbnail: string;
  requiredDataKeys: string[]; // Keys must exist to render
  optional: boolean;
  mapFrameConfig?: {
    layers: {
      id: string;
      visible: boolean;
      opacity: number;
      zIndex: number;
    }[];
    labelOptions?: {
      enabled: boolean;
      fontSettings: {
        font: string;
        size: number;
        color: string;
      };
      position: string;
    };
    legendOptions?: {
      enabled: boolean;
      position: string;
    };
    extentOptions?: {
      padding: number;
      fixedScale: boolean;
      scale?: number;
    };
  };
  scoringLogic?: {
    type: "numeric" | "chart" | "colorcoded";
    thresholds?: Record<string, number>;
    chartType?: "bar" | "radar";
  };
}

export interface PPTXConfig {
  id: string;
  name: string;
  slides: {
    templateId: string;
    order: number;
    settings: Record<string, any>; // Template-specific settings
  }[];
  defaultBranding: {
    logo: string;
    primaryColor: string;
    secondaryColor: string;
  };
  shared: boolean;
  sharedWith?: string[]; // User IDs
}
```

### User Interface

`modules/pptx/components/PPTXConfigEditor.tsx` provides a drag-and-drop interface where users:

- Browse the slide template library (thumbnails)
- Drag templates into their presentation sequence
- Configure individual slide settings
- Preview the fully-composed presentation
- Save/share configuration for reuse

These configurations pair with the main `services/pptx/exporter.ts` which applies user selections to the actual export process.

## Land Use Modal Configuration

Users can configure parameters for each land use type to customise evaluation criteria.

### Interface Definition

```typescript
// modules/landuse/types.ts
export interface LandUseConfig {
  id: string;
  name: string;
  landUseTypes: {
    id: string;
    name: string;
    parameters: {
      minimumArea: number; // square meters
      minimumWidth: number; // meters
      maximumSlope: number; // degrees
      distanceToPublicTransport: number; // meters
      distanceToOpenSpace: number; // meters
      bushfireCategory?: string[];
      floodCategory?: string[];
      otherConstraints: Record<string, any>;
    };
    evaluationCriteria: {
      id: string;
      name: string;
      weight: number;
      formula: string; // JS expression
    }[];
  }[];
}
```

## Site Reviewer Configuration

Allows users to define custom assessment fields and evaluation methodologies.

### Interface Definition

```typescript
// modules/sitereviewer/types.ts
export interface SiteReviewConfig {
  id: string;
  name: string;
  assessmentFields: {
    id: string;
    name: string;
    type: "text" | "number" | "select" | "boolean" | "score";
    required: boolean;
    options?: string[]; // For select type
    scoreRange?: [number, number]; // For score type
  }[];
  evaluationMethodology: {
    id: string;
    name: string;
    fields: string[]; // IDs from assessmentFields
    formula: string; // JS expression for calculation
    weight: number;
  }[];
  displayOptions: {
    dashboardLayout: "grid" | "list" | "kanban";
    reportSections: string[]; // Section IDs to include
    visualisations: {
      id: string;
      type: "chart" | "table" | "map";
      dataSource: string; // Field ID or formula
      settings: Record<string, any>;
    }[];
  };
}
```

## Feasibility Calculator Configuration

Provides tools for configuring development metrics, financial assumptions, and dwelling parameters.

### Interface Definition

```typescript
// modules/feasibility/types.ts
export interface FeasibilityConfig {
  id: string;
  name: string;
  developmentMetrics: {
    buildingFootprint: number; // percentage
    floorHeights: Record<string, number>; // meters by level type
    efficiencyRatios: Record<string, number>; // by use type
  };
  constructionCosts: {
    id: string;
    name: string;
    lga?: string;
    developmentType: string;
    baseRate: number; // per sqm
    additionalCosts: Record<string, number>;
  }[];
  financialAssumptions: {
    id: string;
    name: string;
    interestRate: number;
    fees: Record<string, number | string>; // Fixed or percentage
    profitMargins: Record<string, number>; // by development type
  }[];
  dwellingTemplates: {
    id: string;
    type: string; // studio, 1bed, 2bed, etc
    area: number; // sqm
    price: number;
    fixtures: Record<string, number>; // cost breakdown
  }[];
  affordableHousing: {
    enabled: boolean;
    percentage: number;
    discountRate: number;
  };
  lmrHousingOptions?: {
    enabled: boolean;
    densityBonus: number;
    heightBonus: number;
    affordablePercentage: number;
  };
}
```

## Map & Layer Configuration

Users can configure their preferred:

- Base maps
- Default layers & visibility
- Symbology for parcels, zones, etc.
- Saved map extents/bookmarks

### Interface Definition

```typescript
// modules/map/types.ts
export interface UserMapConfig {
  basemap: "satellite" | "streets" | "topo" | "light" | "dark";
  defaultLayers: {
    id: string;
    visible: boolean;
    opacity: number;
    symbology?: {
      fillColor: string;
      fillOpacity: number;
      outlineColor: string;
      outlineWidth: number;
      outlineStyle: "solid" | "dashed" | "dotted";
    };
  }[];
  defaultExtent?: {
    center: [number, number]; // longitude, latitude
    zoom: number;
  };
  savedLocations: {
    name: string;
    extent: {
      center: [number, number];
      zoom: number;
    };
  }[];
}
```

## Master Configuration Hub

A centralised configuration dashboard for module preferences will be accessible from the top-right dropdown menu throughout the application. This hub provides:

1. **Cross-module configuration management:** Access to all module-specific configurations in one place
2. **Configuration profiles:** Save and load complete sets of module configurations
3. **Organisation-wide templates:** Access to organisation-approved configuration templates
4. **Import/Export tools:** Export configurations for backup or sharing with colleagues
5. **Default settings:** Set preferred defaults for new projects

### Interface Definition

```typescript
// components/configuration/types.ts
export interface MasterConfigHub {
  activeProfile: string;
  profiles: {
    id: string;
    name: string;
    description: string;
    moduleConfigs: {
      module: string;
      configId: string;
    }[];
  }[];
  organisationDefaults?: {
    branding: {
      logo: string;
      primaryColor: string;
      secondaryColor: string;
    };
    moduleDefaults: Record<string, string>; // module: configId
  };
}
```

### Implementation

The master configuration hub will be implemented as:

```typescript
// components/configuration/MasterConfigHub.tsx
import React from "react";
import { Tabs, Tab, TabList, TabPanel } from "components/ui/Tabs";
import ModuleConfigList from "./ModuleConfigList";
import ProfileManager from "./ProfileManager";
import OrganisationDefaults from "./OrganisationDefaults";
import { useMasterConfig } from "hooks/useMasterConfig";

export const MasterConfigHub: React.FC = () => {
  const { profiles, activeProfile, saveProfile, setActiveProfile } =
    useMasterConfig();

  return (
    <div className="master-config-hub">
      <h1>Module Configuration Hub</h1>

      <Tabs>
        <TabList>
          <Tab>Module Configurations</Tab>
          <Tab>Profiles</Tab>
          <Tab>Organisation Defaults</Tab>
          <Tab>Import/Export</Tab>
        </TabList>

        <TabPanel>
          <ModuleConfigList />
        </TabPanel>

        <TabPanel>
          <ProfileManager
            profiles={profiles}
            activeProfile={activeProfile}
            onSave={saveProfile}
            onActivate={setActiveProfile}
          />
        </TabPanel>

        <TabPanel>
          <OrganisationDefaults />
        </TabPanel>

        <TabPanel>
          <div className="import-export">
            <button className="export-btn">Export Configurations</button>
            <button className="import-btn">Import Configurations</button>
          </div>
        </TabPanel>
      </Tabs>
    </div>
  );
};
```

## Implementation Details

Each module stores its custom user preferences under a consistent pattern in the database:

```typescript
// modules/[module]/preferences.ts
export const savePreferences = async (userId: string, preferences: any) => {
  try {
    await supabase.from("user_preferences").upsert({
      user_id: userId,
      module: "module-name",
      preferences: preferences,
    });

    // Also save to localStorage as fallback
    localStorage.setItem(
      "module-name:preferences",
      JSON.stringify(preferences)
    );

    return { success: true };
  } catch (error) {
    console.error("Error saving preferences:", error);
    return { success: false, error };
  }
};
```

This allows for a consistent API while letting each module define its own preference schema.

## Configuration UI

Each module will provide a dedicated configuration interface that:

1. Presents appropriate controls for each configuration type
2. Validates user input against schema requirements
3. Provides sensible defaults for new configurations
4. Allows saving, loading, and sharing configurations
5. Supports template libraries for common setups

These interfaces will be accessible from both the module itself and the master configuration hub accessible via the dropdown menu in the top right of the application.
