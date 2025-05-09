# 8. Re-usability & Performance Guidelines

The architecture prioritizes performance and code reuse through several key principles.

## Core Guidelines

1. **Pure functions in `utils/`**

   - Dependency-free, easily unit-tested
   - Single responsibility principle
   - Exported individually for tree-shaking

2. **React context for shared GIS map instance**

   - Avoid re-mounting expensive map components per module
   - Maintain map state across route changes
   - Example: `contexts/MapContext.tsx`

3. **Dynamic imports**

   - Use `next/dynamic` for heavy components (Mapbox GL, PPTXGenJS)
   - Pair with `<Suspense>` for loading states
   - Only load what's needed for the current view

   ```tsx
   // Example dynamic import
   const MapComponent = dynamic(() => import("@/components/Map"), {
     loading: () => <MapSkeleton />,
     ssr: false, // Don't render map on server
   });
   ```

4. **Memoize expensive operations**

   - Use `reselect` or `useMemo` for derived data
   - Cache results of expensive calculations
   - Avoid unnecessary re-renders

5. **ESM tree-shaking**
   - Keep each utility in its own file
   - Avoid side effects in module scope
   - Use named exports instead of default exports

## Implementation Examples

### Shared Map Context

```tsx
// contexts/MapContext.tsx
export const MapContext = createContext<MapContextType | null>(null);

export function MapProvider({ children }) {
  const [map, setMap] = useState(null);
  // Initialize map only once

  return (
    <MapContext.Provider value={{ map, setMap }}>
      {children}
    </MapContext.Provider>
  );
}

// In a feature slice:
const { map } = useContext(MapContext);
```

### Memoized Selectors

```tsx
// Using useMemo to calculate derived state
const sortedItems = useMemo(() => {
  return [...items].sort((a, b) => a.name.localeCompare(b.name));
}, [items]);
```
