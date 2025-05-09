# 6. API & Endpoint Registry

To eliminate hardcoded URLs and maintain a single source of truth for API endpoints, we centralize all external service connections in a dedicated configuration file.

## Central Endpoint Configuration

```ts
// config/endpoints.ts
export const ARCGIS_BASE =
  "https://services.arcgisonline.com/ArcGIS/rest/services";

export const ENDPOINTS = {
  parcels: `${ARCGIS_BASE}/Parcels/MapServer`,
  imagery: "https://my-metromap-proxy/api/imagery",
  giraffe: "https://api.giraffeanalytics.com/v2",
  pptxGen: "/api/pptx", // Next.js route (edge function)
};
```

## Usage Pattern

All network layers import `ENDPOINTS` – **no magic strings** scattered through code:

```ts
// services/api/arcgis.ts
import { ENDPOINTS } from "@/config/endpoints";

export async function fetchParcelData(id: string) {
  const response = await fetch(`${ENDPOINTS.parcels}/query?...`);
  // ...
}
```

## Benefits

- **Maintainability**: Change an endpoint URL in one place
- **Environment awareness**: Can easily swap endpoints based on environment (dev/prod)
- **Discovery**: New developers can quickly see what APIs the application uses
- **Testability**: Easier to mock endpoints for testing
