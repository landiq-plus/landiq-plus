# 3. Runtime Configurability

The application employs a module-based configuration system that allows administrators to enable/disable features and customise how they appear in the navigation without code changes.

## Module Configuration Type

```ts
// config/modules.ts
export type ModuleKey =
  | "localArea"
  | "development"
  | "landUse"
  | "imagery"
  | "feasibility"
  | "valuation"
  | "shortlist"
  | "openSpace"
  | "merge"
  | "pptx";

export interface ModuleConfig {
  key: ModuleKey;
  enabled: boolean;
  group: "Analysis" | "Reporting" | "Utilities";
  label: string;
  icon: React.ElementType;
  route: string; // Next.js route
}

export const MODULES: ModuleConfig[] = [
  {
    key: "localArea",
    enabled: true,
    group: "Analysis",
    label: "Local Area",
    icon: MapPin,
    route: "/local-area",
  },
  {
    key: "development",
    enabled: true,
    group: "Analysis",
    label: "Development",
    icon: Building2,
    route: "/development",
  },
  // …
];
```

## Implementation Notes

- Landing cards and the dropdown iterate over this list
- Switching a module on/off is one line of JSON
- Groups automatically appear/disappear if all their modules are disabled
- Icons are imported from Lucide React library
