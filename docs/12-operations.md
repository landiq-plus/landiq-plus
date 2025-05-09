# 12. Operational Concerns

Building a production-ready application requires consideration of operational aspects beyond just the code.

## Environment Variables

- **Env vars** stored in `.env.local` (gitignored)
  - ArcGIS tokens
  - Supabase keys
  - API endpoints for different environments
  - Feature flags

```
# .env.local example
NEXT_PUBLIC_ARCGIS_API_KEY=xxxxx
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=xxxxx
ENABLE_FEATURE_X=true
```

Access via the type-safe environment accessor:

```typescript
// config/env.d.ts
export interface Env {
  NEXT_PUBLIC_ARCGIS_API_KEY: string;
  SUPABASE_URL: string;
  SUPABASE_ANON_KEY: string;
  ENABLE_FEATURE_X: boolean;
}

export const env: Env = {
  NEXT_PUBLIC_ARCGIS_API_KEY: process.env.NEXT_PUBLIC_ARCGIS_API_KEY!,
  SUPABASE_URL: process.env.SUPABASE_URL!,
  SUPABASE_ANON_KEY: process.env.SUPABASE_ANON_KEY!,
  ENABLE_FEATURE_X: process.env.ENABLE_FEATURE_X === "true",
};
```

## Credential Management via Giraffe

For services integrated with the Giraffe platform, credentials can be retrieved dynamically at runtime:

- **Tokens from Giraffe layers** avoid hardcoded credentials
  - Access layer configurations via `giraffeState.get('projectLayers')`
  - Extract tokens from vector tile URLs or layer configurations
  - Use these tokens for authenticated service requests

```typescript
// Example: Extracting ArcGIS token from Giraffe layer
async function getArcGISToken(layerId) {
  const projectLayers = await giraffeState.get("projectLayers");
  const targetLayer = projectLayers?.find((layer) => layer.layer === layerId);

  if (targetLayer?.layer_full?.vector_source?.tiles?.[0]) {
    const vectorTileUrl = targetLayer.layer_full.vector_source.tiles[0];
    const decodedUrl = decodeURIComponent(
      vectorTileUrl.split("/featureServer/{z}/{x}/{y}/")?.[1] || ""
    );
    return decodedUrl.split("token=")?.[1]?.split("&")?.[0];
  }

  return null;
}
```

This approach:

- Avoids storing sensitive credentials in environment variables
- Delegates authentication management to the Giraffe platform
- Ensures tokens are always current and valid
- Simplifies credential rotation and management

## Monitoring

- **Sentry** for front-end error tracking

  - Capture runtime errors
  - Track performance issues
  - Alert on unexpected behavior

- **Vercel Analytics** for usage patterns
  - Page views and navigation flows
  - Performance metrics
  - User engagement

## Feature Flags

Two approaches available:

1. **Simple boolean in `modules.ts`**

   - Quick on/off toggle for entire modules
   - Configured at build time

2. **Remote JSON fetch for enterprise tier**
   - Dynamic feature flags
   - User or organization-specific features
   - A/B testing capability

```typescript
// services/featureFlags.ts
export async function getFeatureFlags(userId?: string) {
  const response = await fetch("/api/feature-flags?userId=" + (userId || ""));
  return response.json();
}
```

## Accessibility

Accessibility is a first-class concern:

- **Keyboard navigation** throughout the application
- **ARIA labels** on dropdown & cards
- **Color contrast** meeting WCAG standards
- **Screen reader** support for interactive elements
